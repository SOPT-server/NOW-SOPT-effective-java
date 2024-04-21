> 정적 팩터리 메서드와 public 생성자는 각자의 쓰임새가 있다. 장단점을 이해하고 사용하자. 상대적으로 정적 팩터리 메서드를 사용할 때 유리한 경우가 많으니, 무작정 public 생성자를 제공하던 습관이 있다면 고치자!
>

`public 생성자` -> 클라이언트가 클래스의 인스턴스를 얻는 전통적 수단

<aside>
💡 클래스는 생성자와 별도로 `정적 팩터리 메서드(static factory method)`를 제공할 수 있음!

</aside>

`정적 팩터리 메서드`: 클래스의 인스턴스를 반환하는 단순한 정적 메서드

ex. java.lang.Boolean 클래스의 valueOf 메서드

: 기본 타입인 boolean 값을 받아서 → Boolean 객체 참조로 변환

```java
public static Boolean valueOf(boolean b) {
    return (b ? TRUE : FALSE);
}
```

# 정적 팩터리 메서드가 생성자보다 좋은 장점 5가지

## 1. 이름을 가질 수 있다.

<aside>
💡 생성자에 넘기는 매개변수와 생성자 자체만으로는 반환될 객체의 특성을 제대로 설명하지 못한다.

</aside>

<aside>
💡 동일한 시그니처의 생성자를 가질 수 없다.
→ 한 클래스에 시그니처가 같은 생성자가 여러 개 필요하다면 생성자 대신 정적 팩터리 메서드를 사용하자!

</aside>

### 예시. Order 클래스 - prime한 order와 urgent한 order

Order 클래스에서 prime한 order와 urgent한 order를 둘 다 표현해야 한다.

```java
public class Order {

    private boolean prime;
    
    private boolean urgent;
    
    private Product product;
}
```

- 방법 1

  생성자의 시그니처가 동일하기 때문에 `컴파일 에러` 발생

    ```java
    public Order(Product product, boolean prime) {
        this.product = product;
        this.prime = prime;
    }
    
    public Order(Product product, boolean urgent) {
        this.product = product;
        this.urgent = urgent;
    }
    ```

  → 매개변수의 개수나 타입이 달라야 하지만 다르지 않음

- 방법 2

  방법 1에서 우회하여, 파라미터의 순서를 바꿈

  → 컴파일 에러가 발생하진 않지만, `생성자는 이름을 표현할 수 없음`

    ```java
    public Order(Product product, boolean prime) {
        this.product = product;
        this.prime = prime;
    }
        
    public Order(boolean urgent, Product product) {
        this.product = product;
        this.urgent = urgent;
    }
    ```

- 방법 3

    <aside>
    💡 생성자가 아닌 정적 팩터리 메서드를 사용하여 `이름을 표현하자!`

    </aside>

    ```java
    public static Order **primeOrder**(Product product) {
        Order order = new Order();
        order.prime = true;
        order.product = product;
        return order;
    }
    
    public static Order **urgentOrder**(Product product) {
        Order order = new Order();
        order.urgent = true;
        order.product = product;
        return order;
    }
    ```


## 2. 호출될 때마다 인스턴스를 새로 생성하지 않아도 된다.

### 예시. Settings 클래스

```java
public class Settings {

    private boolean useAutoSteering;

    private boolean useABS;

    private Difficulty difficulty;
}
```

만약 생성자만 사용한다면 아래에서처럼 매번 새로운 인스턴스를 생성하게 된다.

→ 생성자만을 통해서는 인스턴스 생성을 통제할 수 없다.

```java
public static void main(String[] args) {

    System.out.println(new Settings());
    System.out.println(new Settings());
    System.out.println(new Settings());
}
```

경우에 따라서 어떤 인스턴스를 매번 만들지, 특정한 경우에만 새로 만들지 등등 인스턴스 생성을 통제해야 할 때가 있다.

<aside>
💡 외부에서 생성자 호출 통제하기 위해 생성자를 private으로,
미리 만들어준 인스턴스를 정적 팩터리 메서드를 통해서만 제공
→ `인스턴스의 생성을 control`!

</aside>

```java
public class Settings {

		...
		
		private Settings() {}

    private static final Settings SETTINGS = new Settings();

    public static Settings getInstance() {
        return SETTINGS;
    }
}
```

### 예시. java.lang.Boolean 클래스의 valueOf 메서드

```java
public static Boolean valueOf(boolean b) {
    return (b ? TRUE : FALSE);
}

public static final Boolean TRUE = new Boolean(true);
public static final Boolean FALSE = new Boolean(false);
```

→ valueOf 메서드는 미리 만들어둔 인스턴스를 사용할 뿐, 객체를 아예 생성하지 않는다.

이처럼 정적 팩터리 메서드를 사용하여 반복되는 요청에 같은 객체를 반환하는 식으로 언제 어느 인스턴스를 살아있게 할지 통제한다. → 이러한 클래스를 `인스턴스 통제(instance-controlled) 클래스`라고 한다.

### 인스턴스를 통제해야 하는 이유?

싱글턴 패턴 적용

인스턴스화 불가로 만들기

플라이웨이트의 근간

열거 타입에서 인스턴스가 하나임을 보장

## 3. 반환 타입의 하위 타입 객체를 반환할 수 있는 능력이 있다.

### 예시. HelloService(인터페이스) - EnglishHelloService(구현체)와 KoreanHelloService(구현체)

생성자를 사용할 경우, 해당하는 클래스의 인스턴스만 만들어준다.

<aside>
💡 반면 정적 팩터리 메서드의 리턴 타입을 인터페이스 혹은 부모 클래스로 지정하고, 정작 리턴은 구현체나 자식 클래스의 객체로 할 수 있다.

</aside>

<aside>
💡 즉, 반환할 객체의 클래스를 자유롭게 선택할 수 있다. → 유연성!

</aside>

```java
public class HelloServiceFactory {

		public static **HelloService** of(String lang) {    //HelloService는 인터페이스
				if (lang.equals("ko")) {
						return new **KoreanHelloService**();    //KoreanHelloService는 구현체
				} else {
						return new **EnglishHelloService**();    //EnglishHelloService는 구현체
				}
		}
}
```

```java
public interface HelloService {

		String hello();
}
```

```java
public class KoreanHelloService implements HelloService {

		@Override
		public String hello() {
				return "안녕하세요";
		}
}
```

```java
public class EnglishHelloService implements HelloService {

		@Override
		public String hello() {
				return "hi";
		}
}
```

## 4. 입력 매개변수에 따라 매번 다른 클래스의 객체를 반환할 수 있다.

### 예시. HelloServiceFactory

```java
public static void main(String[] args) {
 
		HelloService ko = HelloServiceFactory.of("ko");
}
```

<aside>
💡 `HelloServiceFactory`를 거쳐서 어떤 인스턴스를 가져와도 `HelloService`라는 인터페이스 타입이다.
→ 구현체가 어떤 타입인지를 클라이언트에게 숨길 수 있다. 구현체를 공개하지 않아도 객체를 반환할 수 있어서 API를 작게 유지할 수 있다.

</aside>

<aside>
💡 또한, 클라이언트에게 `인터페이스 기반 프레임워크`를 사용하도록 강제할 수 있다.
→ 얻은 객체를 구현체가 아닌 인터페이스만으로 다루게 된다.

</aside>

### 참고. 인터페이스에서 메서드와 멤버 변수의 가시성

- 메서드: public, abstract (생략 권장)
- 멤버 변수: public, static, final

### Java 8부터 인터페이스에서 정적 메서드를 가질 수 있게 되었다!

인터페이스에서 정적 메서드를 선언할 수 있기 때문에
정적 팩터리 메서드를 가지고 있는 동반 클래스(companion class)를 만들지 않아도 된다.

- AS IS

    ```java
    // Java 8 이전 인터페이스에서 정적 메서드 선언 X
    public interface HelloService {
    
    		String hello();
    }
    ```

    ```java
    // 동반 클래스 - 정적 팩터리 메서드를 가지고 있는 
    public class HelloServiceFactory {
    
    		public static **HelloService** of(String lang) {    //HelloService는 인터페이스
    				if (lang.equals("ko")) {
    						return new **KoreanHelloService**();    //KoreanHelloService는 구현체
    				} else {
    						return new **EnglishHelloService**();    //EnglishHelloService는 구현체
    				}
    		}
    }
    ```

- TO BE

    ```java
    // Java 8 이후 인터페이스에서 정적 메서드 선언 가능
    // -> 인터페이스에서 정적 메서드 가지고 있기
    public interface HelloService {
    
    		String hello();
    		
    		**public static HelloService of(String lang) {
    				if (lang.equals("ko")) {
    						return new KoreanHelloService();
    				} else {
    						return new EnglishHelloService();
    				}
    		}**
    }
    ```

    ```java
    // 이러한 동반 클래스가 많이 필요하지 않게 됨
    public class HelloServiceFactory {
    
    		public static void main(String[] args){
    				**HelloService ko = HelloService.of("ko");**
    				System.out.println(ko.hello());    // 안녕하세요
    		}
    }
    ```


## 5. 정적 팩터리 메서드를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다.

### 예시. ChineseHelloService(구현체)가 존재하지 않아도 되는..!

HelloService라는 인터페이스만 있고 구현체는 없다고 가정

```java
public interface HelloService {
		String hello();
}
```

```java
public class HelloServiceFactory {

		public static void main(String[] args){
				ServiceLoader<HelloService> loader = ServiceLoader.load(HelloService.class);
				Optional<HelloService> helloServiceOptional = loader.findFirst();
				helloServiceOptional.ifPresent(h -> {
						System.out.println(h.hello());    // Ni Hao
				});
		}
}
```

- `HelloService` 인터페이스만 있는데, 어떻게 `ChineseHelloService`가 동작할 수 있을까?

  → Service Provider Framework의 Java 기본 구현체인 java.util.ServiceLoader를 통해 가능!

    1. 다른 프로젝트에서 `ChineseHelloService` 구현
    2. 해당 프로젝트의 `META-INF/services`에 제공할 구현체의 경로를 적어줌

       ex. <packageName>.ChineseHelloService

    3. 해당 경로를 보고 Jar로 패키징할 때 `META-INF/services`에 있는 구현체 정보가 들어감
    4. 사용할 프로젝트에서 의존성을 추가해줌으로써 사용할 수 있게 됨

- 의존성을 추가했으니 아래처럼 ChineseHelloService를 바로 호출하면?

    ```java
    public class HelloServiceFactory {
    
    		public static void main(String[] args){
    				HelloService helloService = new ChineseHelloService();
    				System.out.println(helloService.hello());
    		}
    }
    ```

  → `ChineseHelloService(구현체)`에 의존하는 코드이기 때문에 바람직하지 않음.

  어떤 구현체가 오는지 관심 없게 인터페이스 기반으로 코딩하는 것이 주는 유리함이 크다!


# 정적 팩터리 메서드의 단점 2가지

## 1. 상속을 하려면 public이나 protected 생성자가 필요하니, 정적 팩터리 메서드만 제공하면 하위 클래스를 만들 수 없다.

`정적 팩터리 메서드`만을 사용한다 → 생성자를 `private`으로 설정해야 한다 → 상속을 허용하지 않는다

<aside>
💡 상속 대신 Composition을 사용하도록 유도함으로써 장점이 될 수도 있다!

</aside>

## 2. 정적 팩터리 메서드는 프로그래머가 찾기 어렵다.

javadoc 사용 시, 생성자는 별도의 컬럼이 있다.

생성자를 사용하지 않고 정적 팩터리 메서드만 사용하는 경우 API 설명이 명확히 드러나지 않으니 널리 알려진 규약에 따라 메서드 이름을 짓도록 하자!

<aside>
💡 *`/** * 이 클래스의 인스턴스는 #getInstance()를 통해 사용한다. * @see #getInstance() */`*
→ 이런 식으로 문서화를 하자…!!

</aside>

### 정적 팩터리 메서드 명명 방식

- `from`: 매개변수를 하나 받아서 해당 타입의 인스턴스 반환, 형변환 메서드
    - Date date = Date.from(instant);
- `of`: 매개변수를 여러개 받아서 적절한 타입의 인스턴스 반환, 집계 메서드
    - Set<Rank> faceCards = EnumSet.of(JACK, QUEEN, KING);
- `valueOf`: from과 of의 더 자세한 버전
    - BigInteger prime = Biginteger.valueOf(Integer.MAX_VALUE);
- `instance, getInstance`: (매개변수를 받는다면) 매개변수로 명시한 인스턴스 반환, 같은 인스턴스임을 보장하지는 않음
    - StackWalker Luke = StackWalker.getlnstance(options);
- `create, newInstance`: `instance, getInstance`와 같지만, 매번 새로운 인스턴스를 생성하여 반환함을 보장함
    - Object newArray = Array.newInstance(classObject, arrayLen);
- `getType`: `getInstance`와 같지만, 생성할 클래스가 아닌 다른 클래스에 팩터리 메서드를 정의할 때
    - Filestore fs = Files.getFileStore(path)
- `newType`: `newInstnace`와 같지만, 생성할 클래스가 아닌 다른 클래스에 팩터리 메서드를 정의할 때
    - BufferedReader br = Files.newBufferedReader(path);
- `type`: `getType`과 `newType`의 간결한 버전
    - List<Complaint> litany = Collections.list(legacyLitany);