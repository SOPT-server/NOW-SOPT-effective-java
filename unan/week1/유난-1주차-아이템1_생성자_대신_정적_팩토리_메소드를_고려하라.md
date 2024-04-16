## 정적 팩토리 메서드

- 생성자(constructor)와는 별개의 개념
- 클래스의 인스턴스를 반환하는 정적 메서드

```java
public static Boolean valueOf(boolean b) {
    return b ? Boolean.TRUE : Boolean.FALSE;
}

---

public class Foo {
   String name;

  public static Foo withName(String name) {
     return new Foo(name);
  }
}
```

### 장점

1. 이름을 가질 수 있다.
- 생성자는 반환될 객체의 특성을 제대로 설명하지 못한다.
- 하나의 시그니처로는 생성자를 하나만 만들 수 있다.  한 클래스에서 여러개의 생성자를 필요로 할 때, 생성자를 정적 팩토리 메소드로 바꾸고, 차이를 잘 드러내는 이름을 지어주자.

```java
public class Foo {

   public Foo(String name) {
        this.name = name;
    }

    public Foo(String address) {
        this.address = address;
    }
}
```

1. 호출될 때마다 인스턴스를 새로 생성하지는 않아도 된다.
2. 반환 타입의 하위 타입 객체를 반환할 수 있는 능력이 있다.
- client의 입장에서는 interface만 보고, 구현체의 존재에 대해 알 필요가 없다.
- ex) java.util.Collections
1. 입력 매개변수에 따라 매번 다른 클래스의 객체를 반환할 수 있다.
- 반환 타입의 하위 타입이기만 하면, 어떤 클래스의 객체를 반환하든 상관없다.

```java
public static Foo getFoo(boolean flag) {
       return flag ? new Foo() : new BarFoo();
    }

    static class BarFoo extends Foo {
    }
```

- ex) EnumSet class는 public static 메소드로 allOf(), of() 를 제공한다. EnumSet의 경우 인스턴스의 개수에 따라서 RegularInstance, JumboInstance를 각각 반환하는데 client는 이두 클래스의 존재를 몰라도 상관없다.

1. 정적 팩토리 메서드를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다.

```java
public static Foo getFoo(boolean Flag) {
  Foo foo = new Foo();

// 중간에 특정 약속되어있는 foo의 텍스트 파일에서 Foo의 구현체의 FQCN을 읽어온다.
// FQCN에 해당하는 인스턴스를 생성한다.
// foo 변수를 해당인스턴스를 가리키도록 수정한다.
  return foo;
}
```

- ex) jdbc (Java Database Connectivity)

### 단점

1. 상속을 하려면 public, protected 생성자가 필요하니 정적 팩토리 메소드만 제공하면 하위클래스를 만들수 없다. 상속보다 컴포지션을 사용하도록 유도하고 불변타입을 만드려면 이 제약을 지켜야 한다는 점에서 장점으로 볼 수도 있다. 
2. 정적 팩토리 메소드는 프로그래머가 찾기 어렵다. javadoc에서는 상단에 생성자를 두지만, 정적 팩토리 메소드는 가독성이 좋게 잘 써야 한다.

### reference
<a href = "https://www.youtube.com/watch?v=X7RXP6EI-5E"> 백기선[이펙티브 자바] <a />
