애플리케이션에 따라 특정 인스턴스는 애플리케이션 내에서 여러개가 필요하지 않을 수 있다.

여러 개가 필요하지 않는 경우는 2가지로 나눌 수 있다.

- 여러개가 필요하지 않은 경우

  → 여러 인스턴스를 만들어도 상관은 없지만, 리소스를 적게 쓰고 객체를 만드는 과정을 생략하는 등 이점이 있다.

- 반드시 하나로 유지해야 하는 경우 → `싱글턴(Singleton)`

  ex. 게임 내의 설정(시스템 컴포넌트), 무상태(stateless) 객체


# 싱글턴(Singleton)

<aside>
💡 인스턴스를 오직 하나만 생성할 수 있는 클래스
ex. 무상태 객체, 시스템 컴포넌트

</aside>

타입을 인터페이스로 정의하고, 그 인터페이스를 구현해서 만든 싱글턴이 아니라면
싱글턴 인스턴스를 가짜(mock) 구현으로 대체할 수 없기 때문에
싱글턴 클래스인 경우 클라이언트에서 테스트하기 어렵다.

# 싱글턴을 보장하는 방법 2가지

## 방법 1. private 생성자 + public static final 필드

```java
public class Elvis {
    **public static final Elvis INSTANCE = new Elvis();

    private Elvis() {    }**

    public void leaveTheBuilding() {
        System.out.println("Whoa baby, I'm outta here!");
    }

    public void sing() {
        System.out.println("I'll have a blue~ Christmas without you~");
    }
}
```

<aside>
💡 `private 생성자` + `public static final 필드`

</aside>

→ public이나 protected 생성자가 없으므로 Elvis 클래스가 초기화 될 때 인스턴스는 단 하나뿐임을 보장, public static 필드가 final이므로 절대 다른 객체 참조 불가능!

```java
public class ElvisMain {
		public static void main(String[] args) {
				Elvis elvis = **Elvis.INSTANCE**;
				elvis.leaveTheBuilding();
		}
}
```

- 클라이언트에서는 `Elvis.INSTANCE`로 인스턴스를 얻고, 인스턴스 메서드 호출

<aside>
💡 1. 간결한 방법을 통해 싱글턴을 보장할 수 있다.
2. javadoc(API)에서도 싱글턴임이 명백히 드러난다. (static final → 필드로 명시적으로 알릴 수 있음)

</aside>

### 문제 1. 테스트의 어려움

<aside>
💡 클라이언트에서 사용하는 싱글턴 클래스를 테스트하기 어렵다.

</aside>

Elvis를 사용하는 클라이언트인 `Concert`

```java
public class Concert {
    private boolean lightsOn;
    private boolean mainStateOpen;
    private **Elvis** elvis;

    public Concert(**Elvis** elvis) {
        this.elvis = elvis;
    }

    public void perform() {
        mainStateOpen = true;
        lightsOn = true;
        **elvis.sing();**
    }

    ...
}
```

Concert 클래스를 테스트한다고 했을 때, perform() 메서드의 `elvis.sing()`이 계속해서 호출된다.

→ 위와 같은 간단한 예시가 아니라, 외부 API를 호출한다고 가정하면 테스트임에도 지속적인 리소스가 낭비되는 것임.

### 해결. 인터페이스화하여 mocking 하자!

```java
public interface IElvis {
 
    void leaveTheBuilding();
    void sing();
}
```

`IElivs`라는 인터페이스를 통해 추상화!

```java
public class Concert {
    private boolean lightsOn;
    private boolean mainStateOpen;
    private **IElvis** elvis;

    public Concert(**IElvis** elvis) {
        this.elvis = elvis;
    }

    public void perform() {
        mainStateOpen = true;
        lightsOn = true;
        **elvis.sing();**
    }

    ...
}
```

인터페이스 기반 프로그래밍이 가능해짐.

`Concert`가 `Elvis`라는 구현체가 아닌 `IElvis`라는 `인터페이스에 의존`하게 할 수 있게 된다!

인터페이스 기반 프로그래밍의 장점

- `의존성 역전(Dependency Inversion)`: 구현체에 의존하지 않고 인터페이스에 의존하는 느슨한 결합을 가짐
- `다형성` 활용 가능: Elvis가 아닌 다른 구현체더라도 다형적 참조를 통해 다형성 활용 가능
- `테스트 용이`: 의존 객체 대신 테스트에 적합한 객체를 제공함으로써 테스트의 격리성 유지

  → Concert를 테스트하는 것이 목적인데, Conert가 Elvis에 의존하고 있는 것이 문제. Elvis는 어떤 수많은 클래스에 의존하고 있는지/Elvis에 의존하기 위해 얼마나 많은 리소스를 태워야 하는지.. 결국 인터페이스를 도입함으로써 강한 의존성을 끊어내고 mocking 할 수 있게 된다!

- `OCP`: IElvis의 다른 구현체가 등장해도 Concert 클래스에는 변경이 없음

```java
class ConcertTest {
    @Test
    void perform() {
        Concert concert = new Concert(**new MockElvis()**);
        **concert.perform();**

        assertTrue(concert.isLightsOn());
        assertTrue(concert.isMainStateOpen());
    }
}
```

`concert.perform()`에서 구현체인 Elvis의 sing()이 아닌 `mocking한 IElvis의 sing()`을 통해 테스트 시 오버헤드를 줄일 수 있다!!

### 문제 2. 리플렉션(Reflection) API를 통해 private 생성자를 호출할 수 있다.

- 리플렉션?

  > allows us to inspect and/or modify runtime attributes of classes, interfaces, fields and methods.
  Additionally, we can instantiate new objects, invoke methods and get or set field values using reflection.
  >

  [Guide to Java Reflection | Baeldung](https://www.baeldung.com/java-reflection)


```java
public class EnumElvisReflection {
    public static void main(String[] args) {
        try {
            Constructor<Elvis> declaredConstructor 
		            = Elvis.class.getDeclaredConstructor();
		        declaredConstructor.setAccessible(true);
		        
		        // elvis1과 elvis2는 다른 인스턴스가 만들어짐
		        Elvis elvis1 = declaredConstructor.newInstance();
		        Elvis elvis2 = declaredConstructor.newInstance();
        } catch (InvocationTargetException | NoSuchMethodException | InstantiationException | IllegalAccessException e)
            e.printStackTrace();
        }
    }
}
```

- `getDeclaredConstructor()`를 통해 선언되어 있는 모든 생성자에 접근
    - `declaredConstructor.setAccessible(true)`: private 생성자에도 접근 가능하게 함
    - `getConstructor()`: public으로 선언되어 있는 생성자에만 접근

### 해결. 생성자를 수정하여 두번째 객체가 생성되려 할 때 예외를 던지자.

```java
public class Elvis {
		public static final Elvis INSTANCE = new Elvis();
		
		**private static boolean created;**
		
		private Elvis() {
		    if (created) {
		        **throw new UnsupportedOperationException("can't be created by constructor.");**
		    }
		    created = true;
		}
		
		...
}
```

Elvis 클래스에 private static boolean으로 flag 변수를 두고,

객체를 또 생성하려고 할 때 `UnsupportedOperationException` 예외를 던진다!

```java
public class EnumElvisReflection {
    public static void main(String[] args) {
        try {
            Constructor<Elvis> declaredConstructor 
		            = Elvis.class.getDeclaredConstructor();
		        declaredConstructor.setAccessible(true);
		        
		        // 예외를 통해 막았으므로 아래 2줄에서는 에러가 터짐!
		        Elvis elvis1 = declaredConstructor.newInstance();
		        Elvis elvis2 = declaredConstructor.newInstance();
		        
		        Elvis.INSTANCE.sing();  //싱글턴 인스턴스는 사용 가능!
        } catch (InvocationTargetException | NoSuchMethodException | InstantiationException | IllegalAccessException e)
            e.printStackTrace();
        }
    }
}
```

<aside>
💡 싱글턴을 깨뜨리는 리플렉션을 막긴 했지만, 더 이상 장점이 드러나지 않는다. 간결하지 않다.

</aside>

### 문제 3. 역직렬화(Deserialization) 시 새로운 인스턴스가 생길 수 있다.

```java
public interface IElvis {
    void leaveTheBuilding();
    void sing();
}
```

```java
// 직렬화를 위해서는 Serializable를 구현해야 함
public class Elvis implements **Serializable** {
		...
}
```

```java
public class EnumElvisSerialization {
    public static void main(String[] args) {
				// 직렬화
        try (ObjectOutput out 
				        = new ObjectOutputStream(new FileOutputStream("elvis.obj"))) {
            out.writeObject(Elvis.INSTANCE);
        } catch (IOException e) {
            e.printStackTrace();
        }

				// 역직렬화
        try (ObjectInput in 
				        = new ObjectInputStream(new FileInputStream("elvis.obj"))) {
            Elvis elvis = (Elvis) in.readObject();
            System.out.println(elvis == Elvis.INSTANCE);  // false
        } catch (IOException | ClassNotFoundException e) {
            e.printStackTrace();
        }
    }
}

```

<aside>
💡 직렬화 → 역직렬화 하는 과정에서, 
역직렬화 할 때 새로운 인스턴스가 생기면서 기존 싱글턴 인스턴스와 다르다!

</aside>

### 해결. 가짜 Elvis 인스턴스가 생성는 것을, readResolve()를 통해 싱글턴임을 보장하자

```java
// 직렬화를 위해서는 Serializable를 구현해야 함
public class Elvis implements **Serializable** {
		...
		
		private Object **readResolve**() {
		    return INSTANCE;
		}
}
```

> In Java Deserialization, **the *readResolve()* method is used to replace the object that is created during deserialization with a different object**. This can be useful in situations where we need to ensure that only a single instance of a particular class exists in our application or when we want to replace an object with a different instance that may already exist in memory.
>
>
> [Java Serialization: readObject() vs. readResolve() | Baeldung](https://www.baeldung.com/java-serialization-readobject-vs-readresolve)
>

## 방법 2. private 생성자 + 정적 팩터리

<aside>
💡 방법 1. 과 동일하게 리플렉션(Reflection), 테스트, 직렬화/역직렬화 할 때 가지는 문제를 동일하게 가진다.

</aside>

```java
public class Elvis {
    **public static final Elvis INSTANCE = new Elvis();

    private Elvis() {    }**
        
		// getInstance()를 통해 클라이언트에게 싱글턴 인스턴스 제공
    **public static Elvis getInstance() {
        return INSTANCE;
    }**

    ...
}
```

```java
public class ElvisMain {
		public static void main(String[] args) {
        Elvis elvis = Elvis.getInstance();
        elvis.leaveTheBuilding();

        System.out.println(Elvis.getInstance());  // 같은 인스턴스
        System.out.println(Elvis.getInstance());  // 같은 인스턴스
		}
}
```

### 장점 1. 클라이언트의 API 변경 없이 싱글턴을 해제할 수 있다.

클라이언트인 ElvisMain에서는 변화 없이, Elvis에서만 수정을 통해 싱글턴 인스턴스를 제공하지 않을 수 있다.

```java
public class Elvis {
    public static final Elvis INSTANCE = new Elvis();

    private Elvis() {    }
        
    public static Elvis getInstance() {
        **return new Elvis();  // 더 이상 싱글턴이 아님!**
    }

    ...
}
```

### 장점 2. 정적 팩터리를 제네릭 싱글턴 팩토리로 만들 수 있다.

```java
public class GenericElvis<T> {
    public static final GenericElvis<Object> INSTANCE
		     = new GenericElvis<>();
		     
    private GenericElvis() {    }
		     
		// T와 E는 scope가 다르다..!!
		**@SuppressWarnings("unchecked")
    public static <E> GenericElvis<E> getInstance() {
        return (GenericElvis<E>) INSTANCE;
    }**
        
    public void say(T t) {
        System.out.println(t);
    }

    ...
}
```

```java
public class GenericElvisMain {
		public static void main(String[] args) {
				// 타입은 맘대로 지정 가능, Generic이므로
				GenericElvis<**String**> elvis1 = GenericElvis.getInstance();
				GenericElvis<**Integer**> elvis2 = GenericElvis.getInstance();
				
				System.out.println(elvis1.equals(elvis2));  //true, 인스턴스는 동일!!
				System.out.println(elvis1 == elvis2);  //false, 타입은 다름!
				
				elvis1.say("hi");  // hi
				elvis2.say(100);  // 100
		}
}
```

elvis1와 elvis2는 같은 인스턴스라서 참조값은 같다.

타입은 달라서 == 비교는 안된다.

### 장점 3. 정적 팩터리의 메서드 참조를 공급자(supplier)로 사용할 수 있다.

<aside>
💡 공급자(Supplier)는 Java8부터 도입된 함수형 인터페이스의 공급자를 의미한다.

</aside>

```java
public interface Singer {
    void sing();
}
```

```java
public class Elvis implements Singer {
    private static final Elvis INSTANCE = new Elvis();

    private Elvis() {
    }

    public static Elvis getInstance() {
        return INSTANCE;
    }

		...

    @Override
    public void sing() {
        System.out.println("my way~~~");
    }
}
```

- Singer 인터페이스를 두고, Elvis가 구현하도록 변경,
- sing()은 Elvis에서 오버라이딩

```java
public class Concert {
    public void start(**Supplier<Singer> singerSupplier**) {
        Singer singer = singerSupplier.get();
        singer.sing();
    }
}
```

- Conert 클래스의 start 메서드는 `Supplier<Singer>`를 매개변수로 받음
- `Supplier`는 Singer 인터페이스의 구현체의 인스턴스를 생성하는 역할을 함



```java
public static void main(String[] args) {
    Concert concert = new Concert();
    concert.start(Elvis::getInstance);
}
```

- concert 인스턴스에서 start 메서드를 호출
- `Elvis::getInstance`는 Supplier<Singer> 형식을 만족함 (Elvis는 Singer의 구현체니까)

  → Elvis 클래스의 싱글턴 인스턴스를 반환함

- 반환한 Elvis의 싱글턴 인스턴스로 sing() 메서드를 호출

  → my way~~~ 가 출력됨!


## 방법 3. 열거 타입

<aside>
💡 생성자를 private으로 설정하고
public static 메서드를 만들거나, public static final 필드를 정의하거나
할 필요가 전혀 없다!

</aside>

<aside>
💡 리플렉션과 직렬화/역직렬화, 테스트에도 안전하다!

</aside>

```java
public enum Elvis {
    INSTANCE;

    public void leaveTheBuilding() {
        System.out.println("기다려 자기야, 지금 나갈께!");
    }
}
```

```java
public static void main(String[] args) {
    public static void main(String[] args) {
        Elvis elvis = Elvis.INSTANCE;
        elvis.leaveTheBuilding();
    }
}
```