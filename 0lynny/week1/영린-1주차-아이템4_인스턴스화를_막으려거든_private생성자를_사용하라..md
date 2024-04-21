## 아이템4. 인스턴스화를 막으려거든 private 생성자를 사용하라

<br>

정적메서드와 정적필드만은 담은 클래스는 객체지향적 관점에서는 좋지 않은 방식이지만 나름의 쓰임새는 있을 것이다.

하지만 이러한 정적멤버만 담은 클래스는 인스턴스로 만드려는 목적이 아닌데 생성자를 명시해주지 않으면 컴파일러가 자동적으로 기본 생성자를 만들어준다. 

<br>

그렇다면 어떻게 인스턴스화를 막을 수 있을까?

private 생성자를 추가하여 클래스의 인스턴스화를 막을 수 있다. 명시적 생성자가 private이므로 클래스 바깥에서는 접근할 수 없다. 

 ex) java.util.Math 클래스

```java
public final class Math {

    private Math() {}
    
}
```

이렇게 private을 붙여주면 생성자가 존재하지만 호출을 할 수 없다. 

따라서 아래과 같이 적절안 주석을 달아줘도 좋다 !

```java
public class Utilityclass {
    
    // 기본 생성자가 만들어지는 것을 막는다（인스턴스화 방지용
    private UtilityClass() {
        throw new AssertionError();
    }
    // 나머지 코드는 생략
}
```

<br>

>    ✅ ** 핵심 정리 **
    생성자의 접근자를 private으로 설정하여 인스턴스화를 막자.
