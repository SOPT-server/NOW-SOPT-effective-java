# Utility Class (유틸리티 클래스)

<aside>
💡 정적(static) 메서드와 정적(static) 필드만을 제공하는 클래스
인스턴스 메서드와 인스턴스 변수를 제공하지 X

</aside>

- `static`

  ### static 변수

    <aside>
    💡 한 클래스의 모든 인스턴스들이 `공통적인 값`을 유지해야하는 속성일 경우
    → 모든 인스턴스가 공통된 저장 공간을 `공유` (메모리 할당 1번, 메모리 낭비 X)

    </aside>

    ```java
    class Card {
        // 인스턴스 변수
        String kind;    // 무늬
        int number;     // 숫자
        
        // 클래스 변수
        static int width = 100;     // 폭
        static int height = 250;    // 넓이
    }
    ```

    - `생명 주기 (lifetime)`: 프로그램 시작 ~ 프로그램 종료

      → ClassLoader에 의해 Class가 load될 때 ~ ClassLoader에 의해 Class가 unload될 때


    ### static 메서드
    
    인스턴스와 관계 없는 메서드는 클래스 메서드로 정의!

- 기본 타입 값이나 배열 관련 메서드들을 모아놓을 때

  ex. java.lang.Math, java.util.Array

- 특정 인터페이스를 구현하는 객체를 생성해주는 정적 메서드(혹은 팩터리)를 모아놓을 때

  ex. java.util.Collections

- final 클래스와 관련한 메서드를 모아놓을 때

<aside>
💡 `비슷한 기능의 메서드와 상수를 모아서 캡슐화`하는 목적, 클래스 본래의 목적인 `데이터와 데이터 처리를 위한 로직의 캡슐화`를 위함이 X

</aside>

# 인스턴스화를 막는 방법

```java
public class UtilityClass {
    public UtilityClass () {     }  // 컴파일러가 자동으로 만들어줌
    
    public static String hello() {
        return "hello";
    }
}
```

<aside>
💡 인스턴스로 만들어 쓰려고 설계한 것이 아님에도 
생성자를 명시하지 않으면 자동으로 기본 생성자가 생김!

</aside>

## 방법 1. 추상 클래스로 만들기

```java
public abstract class UtilityClass {
    public static String hello() {
        return "hello";
    }
}
```

```java
public class **DefaultUtilityClass extends UtilityClass** {
    public static void main(String[] args) {
        **DefaultUtilityClass utilityClass = new DefaultUtilityClass();**
        utilityClass.hello();
    }
}
```

→ DefaultUtilityClass의 인스턴스를 생성할 때, 슈퍼 클래스인 UtilityClass의 생성자도 같이 호출함

<aside>
💡 추상(abstract) 클래스로 만들어도 해결할 수 없다.
→ 추상 클래스를 `상속해서 인스턴스화` 할 수 있기 때문

</aside>

Spring에서는 abstract로 처리하긴 한다…!

## 방법 2. 생성자를 private으로 설정하자

```java
public class UtilityClass {
    private UtilityClass () {     }  // 외부에서 호출하지 못하게 private으로 설정

    public static String hello() {
        return "hello";
    }
    
    public static void main(String[] args) {
		    UtilityClass utilityClass = new UtilityClass();
    }
}
```

→ 내부에서는 생성자 호출이 가능하다.

```java
public class UtilityClass {
	  // 외부에서 호출하지 못하게 private으로 설정
    private UtilityClass () {
		    throw new AssertionError();  // 내부에서도 생성되지 못하도록 Error를 던지게 한다
    }

    public static String hello() {
        return "hello";
    }
    
    public static void main(String[] args) {
		    UtilityClass utilityClass = new UtilityClass();  // Error 발생!
    }
}
```

→ 생성자를 만들면서까지 못 쓰도록 막는 것이 어색할 수 있으므로 문서화하자.

```java
public class UtilityClass {
    /**
     * 이 클래스는 인스턴스를 만들 수 없습니다.
     */
		private UtilityClass () {
				throw new AssertionError();  // 내부에서도 생성되지 못하도록 Error를 던지게 한다
    }
    
    ...
}
```