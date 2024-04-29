자바의 객체 소멸자

- finalizer
    - 예측할 수 없고, 상황에 따라 위험할 수 있어 일반적으로 불필요하다.
    - 오작동, 낮은 성능, 이식성 문제의 원인이 되기도 한다.
    - 기본적으로 쓰지 말아야 한다.
- cleaner
    - 자바 9에서의 finalizer의 대안이다.
    - finalizer보단 덜 위험하지만, 여전히 예측할 수 없고, 느리고, 일반적으로 불필요하다.

문제점

- 즉시 수행된다는 보장이 없다.
    - 얼마나 신속히 수행할지는 전적으로 가비지 컬렉터 알고리즘에 달렸으며 가비지 컬렉터 구현마다 천차만별이다.
    - 클래스에 finalizer를 달아두면, 인스턴스의 자원 회수가 제멋대로 지연될 수 있다.
    - 자신을 수행할 스레드를 제어할 수 있다는 면에선 cleaner가 낫긴 하나 여전히 백그라운드에서 수행되며 가비지 컬렉터의 통제하에 있으니 즉각 수행된다는 보장은 없다.
    - 수행 시점뿐만아니라 수행 여부조차 보장하지 못한다.
    
    → 프로그램 생애주기와 상관없는, 상태를 영구적으로 수정하는 작업에서는 절대 의존해서는 안된다.
    
- 심각한 성능문제가 있다.
    - finalizer가 가비지 컬렉터 의 효율을 떨어뜨리기 때문에 finalizer를 사용한 객체를 생성하고 파괴하니 50배나 느렸다. cleaner도 비슷하다.
    
    → 안전망 형태로만 사용하면 훨씬 빨라진다.
    
- 심각한 보안문제가 있다.
    - 생성자나 직렬화 과정에서 예외가 발생하면, 이 생성되다 만 객체에서 악의적인 하위 클래스의 finalizer가 수행될 수 있게 된다.
    - 정적필드에 자신의 참조를 할당하여 가비지 컬렉터가 수집하지 못하게 막을 수 있다.
    
    → final 클래스들을 하위 클래스를 만들 수 없으니 공격에서 안전하다. final이 아닌 클래스를 방어하려면 아무 일도 하지 않는 finalize 메서드를 만들고 final로 선언하도록 한다.
    
    → 파일이나 스레드 등 종료해야 할 자원을 담고있는 객체의 클래스에서는 AutoCloseable을 구현해주고 인스턴스를 다 쓰면 close 메서드를 호출해준다.
    
    → 각 인스턴스는 자신이 닫혔는지 추적하는 것이 좋기에, close 메서드에서 이 객체는 유효하지 않음을 필드에 기록하고 다른 메서드는 이 필드를 검사해서 객체가 닫힌 후 사용했다면 IllegalStateException을 던지도록 하자.
    

언제 사용할까?

- 안전망 역할
    - 자원의 소유자가 close 메서드를 호출하지 않은 것에 대비한 안전망 역할이다. (아예 회수하지 않는 것보단 낫기 때문이다.)
    - 자바 라이브러리의 일부 클래스는 안전망 역할의 finalizer를 제공한다. Fileinputstream, FileOutputStream, ThreadPooLExecuto대표적 이다.
- 네이티브 피어와 연결된 객체
    - 일반 자바 객체가 네이티브 메서드를 통해 기능을 위힘한 네이티브 객체를 말한다.
    - 네이티브 피어는 자바 객체가 아니니 가비지 컬렉터는 그 존재를 알지 못한다. 따라서 자바 피어를 회수 할 때 네이브 객체까지 회수하지 못한다.
    - 단, 성능 저하를 감당할 수 있고 네이티브 피어가 심각한 자원을 가지고 있지 않을 때만 해당된다
        
        → 아닐 경우 앞서 말한 close 메서드를 사용하자.
        

Cleaner 예제

- Cleaner 객체는 JAVA 9에 추가된 기능이므로, JAVA 9이상 버전을 사용해야 한다.

```java
public class Room implements AutoCloseable {
    private static final Cleaner cleaner = Cleaner.create();

    // (1) Runnable 인터페이스
    private static class State implements Runnable {
        @Override public void run() {
            System.out.println("Cleaning room");
            numJunkPiles = 0;
        }
    }

    // 방의 상태. cleanable과 공유한다.
    private final State state;

    // cleanable 객체. 수거 대상이 되면 방을 청소한다.
    private final Cleaner.Cleanable cleanable;

    public Room(int numJunkPiles) {
        state = new State(numJunkPiles);
        cleanable = cleaner.register(this, state);
        // (2) Cleaner에 자원회수로직 runnable 인터페이스 등록
    }

    @Override public void close() { // (3) AutoCloseable 인터페이스 구현메소드
        cleanable.clean();
    }
}
```

- Runnable 인터페이스 : Clenaer가 clean을 할때 run()이 호출된다. 절대 Room을 참조해서는 안 된다!
- Cleanable register(Object obj, Runnable action) : 자원회수로직 runnable 인터페이스를 등록한다.
- close : AutoCloseable 인터페이스의 구현 메소드로 try-with-resources 문으로 관리되는 객체일 때 close() 메서드가 자동으로 호출된다. (Cleaner를 믿지 않고,, try-with-resources 블록으로 감싸서 close 메소드가 수행되게 하자)
- 자원회수로직인 State 인스턴스는 절대로 Room 인스턴스를 참조해서는 안된다. 순환참조가 생길경우, GC가 Room인스턴스를 회수해갈 기회가 오지 않는다.
- 정적이 아닌 중첩 클래스는 자동으로 바깥 객체의 참조를 갖게 되기 때문에 State는 중첩 클래스이다.

핵심 정리

→ cleaner(자바 8까지는 finalizer)는 안전망 역할이나 중요하지 않은 네이티브 자원 회수용으로만 사용하자. 물론 이런 경우라도 불확실성과 성능 저하에 주의해야 한다.