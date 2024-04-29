# 예시 1. 똑같은 기능을 하는 객체

<aside>
💡 똑같은 기능의 객체는 매번 생성하기보다는 하나의 객체를 재사용하자.

</aside>

```java
// 하지 말아야 할 예시
String str1 = new String("hello");
```

실행될 때마다 인스턴스를 새로 만드는 것은 매우 불필요하다.

```java
// 권장되는 방법
String str2 = "hello";
```

Java에서는 메모리 효율성과 성능 최적화를 위해 힙 영역에 문자열 풀(String Pool)을 사용한다. 해시 알고리즘을 사용하기 때문에 매우 빠른 속도로 인스턴스를 찾을 수 있다. 또한 같은 문자열을 사용하는 모든 코드는 같은 객체를 사용함이 보장된다.

# 예시 2. 정규표현식, Pattern

<aside>
💡 생성 비용이 아주 비싼 객체는 캐싱해서 재사용하자

</aside>

```java
private static final Pattern ROMAN = Pattern.compile(
    "^(?=.)M*(C[MD]|D?C{0,3})(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");

static boolean isRomanNumeral(String s) {
    return ROMAN.matcher(s).matches();
}
```

`isRomanNumeral`이 빈번히 호출되는 상황에서 성능을 개선할 수 있다.

```java
// AS - IS
boolean isRomanNumeral(String s) {
    return s.matches("^(?=.)M*(C[MD]|D?C{0,3})(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");
    
// TO - BE
static boolean isRomanNumeralSlow(String s) {
    return s.matches("^(?=.)M*(C[MD]|D?C{0,3})(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");
```

`String.matches`는 정규표현식으로 문자열 형태를 확인하기 가장 쉽지만, 성능이 중요한 상황에서 반복해 사용하기에는 적합하지 않다. 메서드 내부에서 만드는 정규표현식용 Pattern 인스턴스는 한 번 쓰고 버려져서 곧바로 GC 대상이기 때문이다.

→ 불변인 Pattern 인스턴스를 `static`으로 생성하여 `캐싱`하고, 호출될 때마다 재사용한다.

# 예시 3. 오토박싱(auto boxing)

```java
// AS - IS
public class Sum {
    private static long sum() {
        Long sum = 0L;
        for (long i = 0; i <= Integer.MAX_VALUE; i++)
            sum += i;
        return sum;
    }
}    
```

Long 타입(Wrapper 타입) 변수 sum에 기본 타입 변수 i가 더해질 때마다 오토박싱이 발생한다.

```java
// TO - BE
public class Sum {
    private static long sum() {
        long sum = 0L;
        for (long i = 0; i <= Integer.MAX_VALUE; i++)
            sum += i;
        return sum;
    }
}    
```

Wrapper 타입이 필요하지 않다면 굳이 사용하여 오토박싱이 일어나지 않도록 하자.

<aside>
💡 박싱된 기본 타입보다는 기본 타입을 사용하자.
의도치 않은 오토박싱이 숨어들지 않도록 주의하자.

</aside>