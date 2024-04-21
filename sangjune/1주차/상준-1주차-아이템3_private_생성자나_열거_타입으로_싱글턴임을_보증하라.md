# private 생성자나 열거 타입으로 싱글톤임을 보증하라

## 싱글톤이란?

싱글톤(singleton)이란 인스턴스를 오직 하나만 생성할 수 있는 클래스를 말한다. 함수와 같은 무상태 객체나 설계상 유일해야 하는 시스템 컴포넌트가 그 예이다.

타입을 인터페이스로 정의한 후 인터페이ㅡ를 구현해서 만든 싱글톤이 아니라면, 클래스를 싱글톤으로 만들 경우 이를 사용하는 클라이언트를 테스트하기 어려운데 이는 싱글톤 인스턴스를 가짜(mock) 구현으로 대체할 수 없기 때문이다.

## 싱글톤을 만드는 방식

### 1. public static final 필드 방식

생성자를 private로 감춰두고 유일한 인스턴스에 접근할 수 있는 수단으로 public static final 필드를 하나 마련해둔다.

private 생성자는 public static final 필드에서 초기화 시 딱 한 번만 호출되며 public이나 protected 생성자가 없다면 초기화될 때 만들어진 인스턴스가 전체 시스템에서 하나뿐임이 보장된다.

단, 리플렉션 API인 AccessibleObject.setAccessible을 사용해 private 생성자를 호출할 수 있기 때문에 이러한 공격을 방어하려면 생성자를 수정하여 두 번째 객체가 생성될 때 예외를 던지게 할 필요가 있다.

public static final 필드 방식의 장점은 다음과 같다

- 해당 클래스가 싱글턴임이 API에서 명확히 드러남
- 간결함

**교재에 나온 예시**

```java
public class Elvis {
    public static final Elvis INSTANCE = new Elvis();
    private Elvis() {...}

    public void leaveTheBuilding() {...}
}
```

### 2. 정적 팩토리 방식

public static final 필드 방식과 마찬가지로 생성자를 private로 감춰두고 인스턴스에 접근할 수 있는 유일한 수단으로 public static 메소드를 하나 제공한다.

**교재에 나온 예시**

```java
public class Elvis {
    private static final Elvis INSTANCE = new Elvis();
    private Elvis() {...}
    public static Elvis getInstance() { return INSTANCE; }

    public void leaveTheBuilding() {...}
}
```

위와 같이 작성할 경우 Elvis.getInstance가 항상 같은 객체의 참조를 반환하기 때문에 두번째 인스턴스는 만들어지지 않는다. (앞서 얘기한 리플렉션 예외는 똑같이 적용된다)

정적 팩토리 방식의 장점은 다음과 같다

- API를 바꾸지 않고도 싱글톤이 아니게 변경할 수 있음
  - 예를 들어 스레드별로 다른 인스턴스를 넘겨줄 수 있음
- 정적 팩토리를 제네릭 싱글톤 팩토리로 만들 수 있음
- 정적 팩토리 메소드 참조를 공급자로 사용할 수 있음  
  위 장점들은 필요할 경우 이런식으로 구현할 수 있다는 뜻인데 이러한 장점들을 활용하지 않을 경우에는 1번 방식으로 public 필드 방식이 좋다고 한다.

### 1, 2번 방식에 대해

싱클톤 클래스를 직렬화하려면 단순히 Serializable을 구현한다고 선언하는 것만으로는 부족하다. 모슨 인스턴스 필드를 transient라고 선언 후 readResolve 메소드를 제공해야한다고 한다. 이렇게 하지 않을 경우 직렬화된 인스턴스를 역직렬화할 때 매번 새로운 인스턴스가 만들어진다고 한다.

### 3. 열거 타입 방식

마지막 방법은 원소가 하나인 열거 타입을 선언하는 것이다.  
public 필드 방식과 비슷하지만 더 간결하고 직렬화에 추가 노력을 들일 필요가 없다. 대부분의 상황에서 원소가 하나뿐인 열거 타입이 싱글톤을 만드는 가장 좋은 방법이다.  
다만 만드려는 싱글톤이 Enum 외의 클래스를 상속해야 한다면 사용할 수 없다.
