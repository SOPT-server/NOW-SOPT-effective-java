> 상속은 강력하지만 캡슐화를 해친다.
상위 클래스와 하위 클래스가 순수한 is-a 관계일 때만 사용하자.
상위 클래스가 확장을 고려해 설계되지 않을 수 있어도 문제다.
컴포지션과 전달을 사용하자.
래퍼 클래스는 하위 클래스보다 견고하고 강력하다.
>

코드 재사용 관점에서 상속은 강력하지만, 잘못 사용하면 오류를 내기 쉬운 소프트웨어를 만들게 된다.

같은 프로그래머가 통제하는 패키지 안에서 상위 클래스와 하위 클래스를 사용한다면 안전하겠지만, 다른 패키지의 구체 클래스를 상속하는 일은 위험하다.

# 상속의 단점

## 메서드 호출과 달리 상속은 캡슐화를 깨뜨린다.

상위 클래스의 구현에 따라 하위 클래스의 동작에 이상이 생길 수 있다. 상위 클래스의 내부 구현은 릴리즈마다 달라질 수 있고, 그에 따라 하위 클래스가 오동작할 수 있기 때문이다.

상위 클래스 설계자가 확장을 충분히 고려하지 않거나, 문서화가 잘 되어있지 않다면
하위 클래스는 상위 클래스의 변화에 따라 수정되어야만 한다.

HashSet 예시는 아래에서 참고하자.

HashSet의 addAll과 add 메서드의 예시처럼 `자기사용(self-use)`하고 있기 때문에 원하는 결과를 얻을 수 없다. 사용하고자 하는 클래스가 third-party 라이브러리이거나 구현 내용이 많은 경우 문제는 더 심각해질 수 있다.

또한 HashSet에 어떤 요소가 추가된다면, addCount를 계산하고 싶기에 요소가 추가될 때마다 오버라이딩해야 한다. 그렇지만 해당 기능이 추가되었는지 확인은 어렵다.

하위 클래스에서 재정의한 메서드가 상위 클래스에서 다시 정의될 수도 있다. 문제 발생 여지가 다분하다.

### 다른 패키지의 구체 클래스를 상속하여 발생한 문제

Java에서 제공하는 `HashSet`을 상속하여 구현한 `InstrumentedHashSet`

```java
@NoArgsConstructor
@Getter
public class InstrumentedHashSet<E> extends HashSet<E> {
    private int addCount = 0;    // 추가된 원소의 수

    public InstrumentedHashSet(int initCap, float loadFactor) {
        super(initCap, loadFactor);
    }

		//addAll이나 add 실행 시, addCount를 증가시키도록 오버라이딩
    @Override 
    public boolean add(E e) {
        addCount++;
        return super.add(e);
    }

    @Override 
    public boolean addAll(Collection<? extends E> c) {
        addCount += c.size();
        return super.addAll(c);
    }

    public static void main(String[] args) {
        InstrumentedHashSet<String> s = new InstrumentedHashSet<>();
        s.addAll(List.of("틱", "탁탁", "펑"));
        System.out.println(s.getAddCount());    //3이 아닌 6이 나온다.
    }
}
```

상위 클래스인 `HashSet`의 `addAll` 메서드가 `add`메서드를 사용하여 구현되었기 때문에 기대하지 않은 값이 나온다.

# 해결책

## 기존 클래스 확장 대신 `컴포지션`을 사용하자.

### 컴포지션(composition; 구성)

새로운 클래스를 만들고 private 필드로 기존 클래스의 인스턴스를 참조하게 한다.

기존 클래스가 새로운 클래스의 구성요소로 쓰인다는 의미의 `구성(composition)`이다.

새 클래스의 인스턴스 메서드들은 기존 클래스의 대응하는 메서드를 호출하여 그 결과를 반환한다.

이러한 방식을 `전달(forwarding)`, 새 클래스의 메서드들을 `전달 메서드(forwarding method)`라고 한다.

이러한 결과 새로운 클래스는 기존 클래스의 내부 구현 방식을 벗어나고,
기존 클래스에 새로운 메서드가 추가되더라도 전혀 영향 받지 않는다.

### 다른 패키지의 구체 클래스를 상속하는 대신 컴포지션을 사용하여 해결

상속 대신 컴포지션을 사용한 래퍼 클래스

`래퍼 클래스`: 기본 데이터 형식을 객체로 표현하거나 기존 클래스에 새로운 기능을 추가하는 데 사용

- 전달 메서드로만 이루어진 재사용 가능한 `전달 클래스`

    ```java
    // 컴포지션 사용
    // 전달 메서드로만 이루어진 재사용 가능한 전달 클래스
    public class ForwardingSet<E> implements Set<E> {
        //새로운 ForwardingSet 클래스를 만들고,
        //private 필드로 기존 클래스(Set)의 인스턴스를 참조
        private final Set<E> s;  
        public ForwardingSet(Set<E> s) { this.s = s; }
    
    		//새로운 ForwardingSet 클래스의 인스턴스 메서드들은
    		//기존 클래스 Set에 대응하는 메서드를 호출하여 그 결과를 반환! -> 전달(forwarding)
    		//이러한 ForwardingSet 클래스의 메서드들 -> 전달 메서드(forwarding method)
        public void clear()               { s.clear();            }
        public boolean contains(Object o) { return s.contains(o); }
        public boolean isEmpty()          { return s.isEmpty();   }
        
        //... 중략
    }
    
    // 새로운 클래스인 ForwardingSet은 기존 클래스인 Set의 내부 구현 방식의 영향에서 벗어나고,
    // Set에 새로운 메서드가 추가되더라도 영향받지 않는다.
    ```

- `래퍼 클래스`

    ```java
    // ForwardingSet을 상속하여 사용, 집합 클래스
    // Set 인스턴스를 감싸고 있다는 의미에서, 래퍼(wrapper) 클래스
    public class MySet<E> extends ForwardingSet<E> {
        private int addCount = 0;
    
    		//Set 인스턴스를 인수로 받는 생성자를 하나 제공
        public MySet(Set<E> s) {
            super(s);
        }
    
        @Override public boolean add(E e) {
            addCount++;
            return super.add(e);
        }
        
        @Override public boolean addAll(Collection<? extends E> c) {
            addCount += c.size();
            return super.addAll(c);
        }
        public int getAddCount() {
            return addCount;
        }
    
        public static void main(String[] args) {
            MySet<String> s = new MySet<>(new HashSet<>());
            s.addAll(List.of("틱", "탁탁", "펑"));
            System.out.println(s.getAddCount()); // 3이 나온다.
        }
    }
    ```

  `InstrumentedSet`은 `HashSet`의 모든 기능을 정의한 `Set` 인터페이스를 활용하여 설계되어 견고하고 유연하다.

  임의의 Set에 계측 기능을 덧씌워서 새로운 Set으로 만드는 것이 이 클래스의 핵심이다!


상속 방식은 구체 클래스를 각각 따로 확장해야 하며, 지원하고 싶은 상위 클래스의 생성자에 대응하는 생성자를 별도로 정의해주어야 하지만,

컴포지션 방식은 한 번만 구현해두면 어떠한 Set 구현체라도 계측할 수 있으며, 기존 생성자들과도 함께 사용 가능하다.
