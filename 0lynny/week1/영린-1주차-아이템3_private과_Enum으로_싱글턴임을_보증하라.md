## 아이템3. private 생성자나 열거타입으로 싱글턴임을 보증하라

<br>

먼저, 싱글턴이란 무엇일까? 

싱글턴(singleton)이란 인스턴스를 오직 하나만 생성할 수 있는 클래스를 말한다.

<br>

싱글턴을 만드는 방식은 아래와 같다.

1. **public static 멤버가 final 필드인 방식**

```java
public class Elvis {
    public static final Elvis INSTANCE = new Elvis();
    private Elvis() { ... }
    
    public void leaveTheBuilding() { ... }
}
```

private 생성자는 public static final 필드인 Elvis.INSTANCE를 한 번 초기화할 때만 호출된다. public이나 protected 생성자가 없으므로 Elvis 클래스가 초기화될 때 만들어진 인스턴스가 하나라는 것이 보장된다.

예외상황이 있는데 권한이 있는 클라이언트는 리플렉션API인 AccessibleObject.setAccessible을 사용해 private 생성자를 호출할 수 있다. 하지만 생성자를 수정하여 두 번째 객체가 생성되려고 할 때 예외를 던지면 이 문제를 해결할 수 있다.

`장점` 

- 해당 클래스가 싱글턴임이 명백히 드러난다.
- 간결하다.

1. **정적 팩토리 메소드를 public static 멤버로 제공하는 방식**

```java
public class Elvis {
    private static final Elvis INSTANCE = new Elvis();
    private Elvis() { ... }
    public static Elvis getInstance() { return INSTANCE; }

    public void leaveTheBuilding() { ... }
}
```

Elvis.getInstance에서는 항상 같은 객체의 참조를 반환하므로 인스턴스가 하나라는 것이 보장된다. 하지만 위의 리플렉션을 통한 예외는 똑같이 적용된다. 

`장점`

- API를 바꾸지 않고도 싱글턴이 아니도록 변경할 수 있다.
- 제네릭 싱글턴 팩터리로 만들 수 있다.
- 정적 팩터리의 메서드 참조를 공급자로 사용할 수 있다.

둘 중의 하나의 방식으로 만든 싱글턴 클래스를 직렬화하려면 단순히 Seriazible을 구현한다고 선언하는 것으로는 부족하다. 모든 인스턴스 필드를 일시적(transient)이라고 선언하고 readResolve 메서드를 제공해야한다. 

```java
// 싱글턴임을 보장해주는 메서드
private Object readResolve() { 
    return INSTANCE; 
}
```

이렇게 해주지 않으면 역질렬화를 하는 과정에서 새로운 인스턴스가 생성되어 싱글턴이 깨지게 되므로 위의 코드를 통해 해당 싱글톤 객체를 반환하도록 해주어야한다.

**3. 열거타입을 선언하는 방식**

```java
public enum Elvis {
    INSTANCE;

    public void leaveTheBuilding() { ... }
}
```

public 필드 방식과 비슷하지만, 더 간결하고 추가노력 없이 직렬화할 수 있고, 리플렉션 공격에서도 인스턴스가 더 생기는 일을 막아준다.  대부분의 상황에서는 가장 좋은 방법이지만 Enum이외의 클래스를 상속해야한다면 사용이 불가능하다. 

<br>
  
>    ✅ ** 핵심 정리 **
    싱글턴을 만드는 방식은 public 필드 방식, 정적 팩터리 메서드 방식, 열거타입 방식이 있으며, 각각의 방식은 리플렉션 예외를 처리해주어야하며, 직렬화를 위해서 readResolve()와 같은 메서드를 추가해야할 수도 있다.  
