## 아이템8. finalizer와 cleaner 사용을 피하라

자바는 다음의 두 가지 객체 소멸자를 제공한다. 그 중 finalizer는 예측할 수 없고, 상황에 따라 위험할 수있어 일반적으로 불필요하다. 이는 오동작, 낮은 성능, 이식성 문제의 원인이 되기도 하여 기본적으로는 사용하지 않아야 한다. 

이의 대안으로는 cleaner 가 있다. 하지만 이 또한 예측하기 어렵고, 느리고, 일반적으로 불필요하다. 

자바는 접근할 수 없게 된 객체를 회수하는 역할을 가비지 컬렉터가 담당하며 비메모리 자원을 회수하는 용도는 try-with-resources, try-finally을 사용하여 해결한다.

<br>

finalizer와 cleaner의 단점은 다음과 같다.

1) finalizer와 cleaner는 즉시 수행된다는 보장이 없으므로, 제때 실행되어야하는 작업은 절대 할 수 없다. 얼마나 신속히 수행할지는 가비지 컬렉터 알고리즘에 따라 달렸으며 이는 구현에 따라 달라진다. 

2) 수행 시점 뿐 아니라 수행 여부도 보장하지 않는다. 접근할 수 없는 객체에 딸린 종료 작업을 수행하지 못한채 중단될 수도 있다. 따라서 상태를 영구적으로 수정하는 작업에서는 절대 사용하면 안된다. 

3) finalizer 동작 중 발생한 예외는 무시되며, 처리할 작업이 남았더라도 그 순간에 종료된다. 잡지 못한 예외로 인해 객체는 마무리가 덜 된 상태로 남게 될 수 있다. 그나마 cleaner를 사용하는 라이브러리에서는 자신의 스레드를 통제하므로 이런 문제가 발생하지 않는다. 

4) 심각한 성능 문제 동반한다. 

finalizer가 가비지 컬렉터의 효울을 떨어뜨리고, 안전망을 설치하는 대가로 성능이 약 5배 정도 느려진다.

5) finalizer 공격에 노출되어 심각한 보안 문제를 일으킬 수 있다. 

finalizer는 생성자나 직렬화 과정에서 예외가 발생한다면 이 생성되다만 객체에서 악의적인 하위 클래스의 finalizer가 수행될 수 있게 된다. 이 finalizer는 정적 필드에 자신의 참조를 할당하여 가비지 컬렉터가 수집하지 못하게 막을 수 있다. 따라서 공격으로 부터 방어하기 위해서 아무 일도 하지 않는 finalizer를 만들고 final로 선언하자.

<br>

그렇다면 파일이나 스레드 등 종료해야할 자원을 담고 있는 객체의 클래스에서 이들을 대신해 줄 해결책은 무엇일까? 

AutoCloseable을 구현해주고, 클라이언트에서 인스턴스를 다 쓰고 나면 close 메서드를 호출해주는 방식이 있다. 여기서 close 메서드에서는 이 객체는 더 이상 유효하지 않음을 필드에 기록하고, 다른 메서드는 이 필드를 검사하여 객체가 닫힌 후 불렸다면 IllegalStateException을 던지면 된다.

<br>

그렇다면 cleaner와 finalizer의 적절한 쓰임새는 없을까? 

첫 번째는 자원의 소유자가 close 메서드를 호출하지 않는 것에 대비한 안전망 역할이다. 자바 라이브러리의 일부 클래스는 FileInputStream, FileOutputStream, ThreadPoolExecutor과 같은 안전망 역할을 하는 finalizer를 제공한다.

두 번째는 네이티브 피어(native peer)와 연결된 객체에서다. 여기서 네이티브 피어란 일반 자바 객체가 네이티브 메서드를 통해 기능을 위임한 네이티브 객체로, 자바 객체가 아니기 때문에 가비지 컬렉터는 존재를 알지 못하여 자바 피어를 회수 할 때 네이티브 객체까지 회수하지 못한다. 따라서 특정 상황에서 cleaner나 finalizer가 처리하기에 적당하다.  

<br>

cleaner를 안전망으로 활용하는 AutoCloseable 클래스는 다음과 같다.

```java
public class Room implements AutoCloseable{
    private static final Cleaner cleaner = Cleaner.create();
    
    //청소가 필요한 자원, 절대 Room을 참조해서는 안된다 !
    private static class State implements Runnable{
        int numJunkPiles; // 방 (Room) 안의 쓰레기 수

        public State(final int numJunkPiles) {
            this.numJunkPiles = numJunkPiles;
        }
        //close 메서드나 cleaner가 호출한다.
        @Override
        public void run() {
            System.out.println("방청소");
            numJunkPiles = 0;
        }
    }

        //방 상태, cleanable과 공유한다.
    private final State state;

        // cleanable 객체, 수거 대상이 되면 방을 청소한다.
    private final Cleaner.Cleanable cleanable;

    public Room(final int numJunkFiles) {
        this.state = new State(numJunkFiles);
        cleanable = cleaner.register(this, state); 
    }

    @Override
    public void close() throws Exception {
        cleanable.clean();
    }
}
```


> ✅ **핵심 정리**
cleaner는 안전망 역할이나 중요하지 않는 네이티브 자원 회수용으로만 사용하자. 물론 이런 경우라도 불확실성과 성능 저하에 주의해야한다.

