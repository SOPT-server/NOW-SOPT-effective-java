## 아이템10. equals는 일반 구약을 지켜 재정의하라

equals 메소드는 굉장히 중요하다! 대부분의 경우에는 우리가 원하는 비교를 수행해준다. 정 그래도 재정의를 해야겠다면 조심해야 한다. 재정의를 할 때는 아래의 체크리스트를 확인해보자.

1. 각 인스턴스가 본질적으로 고유함: 다른 객체는 절대 같을 수가 없다 (ex. Thread)
2. 논리적 동치성을 검사할 일이 없음: String은 equals로 내용이 같으면(논리적으로) true를 반환하는데 이런식의 비교가 필요 없을 경우
3. 상위 클래스에서 재정의한 equals가 하위 클래스에도 맞음: Collection 같은 경우가 이에 해당함
4. 클래스가 private이거나 package-private이고 equals 메소드를 호출할 일이 없음: 아예 금지를 시키고 싶으면 AssertionError()를 띄우면 된다.

이를 모두 통과했다면 어느정도 재정의를 해도 되는 것이다. 그럼 다음의 조건들을 만족해서 equals 메소드를 구현해야 한다.

<br>

### 규약

- 반사성(reflexivity): null이 아닌 모든 참조 값 x에 대해, x.equals(x)는 true
- 대칭성(symmetry): null이 아닌 모든 참조 값 x, y에 대해, x.equals(y)가 true면 y.equals(x)도 true
- 추이성(transitivity): null이 아닌 모든 참조 값 x, y, z에 대해, x.equals(y)가 true고 y.equals(z)가 true면 x.equals(z)도 true
- 일관성(consistency): null이 아닌 모든 참조 값 x, y에 대해, x.equals(y)를 반복해서 호출하면 항상 true거나 false여야 함
- null-아님: null이 아닌 모든 참조 값 x에 대해, x.equals(null)은 false

이를 지키게 되면 동치관계를 구현하게 되는 것이다.

<br>

### 반사성

왠만하면 만족하는 속성이다. 말도 안되는 예시를 한번 보자.

```java
public final class NonReflect {
    @Override
    public boolean equals(Object o) {
        if (o == this) return false;
        return true;
    }
}
```

이렇게 코드를 짜면 인스턴스를 넣어줘도 없다고 뜨는 기이한 현상이 발생할 것이다.

### 대칭성

대칭성은 서로 다른 두 객체가 동치 여부에 똑같이 답을 해야 한다는 것이다. 대칭성은 자칫 잘못하면 규칙에서 벗어날 수 있을 것 같다. 다음의 예시를 한번 보자.

```java
public final class CaseInsensitiveString {
    private final String s;

    public CaseInsensitiveString(String s) {
        this.s = Objects.requireNonNull(s);
    }

    @Override public boolean equals(Object o) {
        if (o instanceof CaseInsensitiveString)
            return s.equalsIgnoreCase(((CaseInsensitiveString) o).s);
        if (o instanceof String)
                return s.equalsIgnoreCase((String) o);
        return false;
    }
    ...생략...
}
```

여기서는 CaseInsensitiveString과 String의 비교에서 문제가 발생한다.

CaseInsensitiveString인 cis가 있다고 하면 cis.equals(string)은 문제가 없지만,

string.equals(cis)에서 문제가 발생한다. 문자열과 문자열 클래스가 아닌 두 인스턴스를 비교하게 되면 toString을 가져와서 비교할텐데 이 과정에선 equlasIgnoreCase()(대소문자를 구별 안함)를 호출하지 않으므로 문제가 발생한다!

이런식으로 기본 클래스와의 직접적인 비교는 생각치 않은 버그가 발생할 수 있으니 유념해야 한다.

### 추이성

1 == 2이고, 2 == 3이면 1 == 3 이어야 한다는 뜻이다. 이 요건도 자칫하면 어기기 쉬우니 유념해야 한다. 다음의 예시를 보자.

```java
public class Point {
    private final int x;
    private final int y;

    public Point(int x, int y) {
        this.x = x;
        this.y = y;
    }

    @Override public boolean equals(Object o) {
        if (!(o instanceof Point))
            return false;
        Point p = (Point) o;
        return p.x == x && p.y == y;
    }
}
```

이 코드에 색상을 더해서 ColorPoint를 만들어보자.

```java
public class ColorPoint extends Point{
    private final Color color;

    public ColorPoint(int x, int y, Color color) {
        super(x, y);
        this.color = color;
    }
}
```

equals를 상속하지 않았는데 그러면 Point의 equals를 그대로 상속되어 색상을 제외하고 비교하게 된다.

이는 받아들일 수 없는 상황이다. 그래서 위치와 색상이 같을 때만 true를 반환하는 equals를 생각해보자.

```java
@Override public boolean equals(Object o) {
    if (!(o instanceof ColorPoint))
        return false;
    return super.equals(o) && ((ColorPoint) o).color == color;
}
```

Point와 ColorPoint와 비교는 서로 결과가 다를 것이다. Point는 색상을 무시하고 ColorPoint는 매개변수의 클래스가 다르다고 매번 false를 반환할 것이다.

```java
@Override public boolean equals(Object o) {
    if (!(o instanceof ColorPoint))
        return false;

    if (!(o instanceof ColorPoint))
        return o.equals(this);

    return super.equals(o) && ((ColorPoint) o).color == color;
}
```

ColorPoint.equals가 Point와의 비교할 때 색상을 무시하고 비교한다면 대칭성은 지켜주지만 추이성을 깨버린다.

```java
ColorPoint p1 = new ColorPoint(1, 2, Color.RED);
Point p2 = new Point(1, 2);
ColorPoint p3 = new ColorPoint(1, 2, Color.BLUE);
```

p1.equals(p2), p2.equals(p3)는 true를 반환하는데 p1.equals(p3)는 false를 반환한다. 이는 추이성을 깨버린 것이다. 또한 무한 재귀를 일으킬 위험도 상존한다.

### 리스코프 치환 원칙(Liskov substitution priciple)

```java
@Override public boolean equals(Object o) {
    if (o == null || o.getClass() != getClass())
        return false;
    Point p = (Point) o;
    return p.x == x && p.y == y;
}
```

ColorPoint는 여전히 Point로써 활용되지 못하게 되는 코드이다.

**리스코프 치환 원칙**은 `어떤 타입에 있어서 중요한 속성이라면 하위 타입에서도 중요하다는 원칙`이다.

우리의 상황에 빗대어 표현하면 Point의 속성인 위치는 ColorPoint에서도 중요하다는 것이다.

위의 문제들을 우회할 방법이 하나 있는데 그것은 ColorPoint 내에 Point를 private 필드로 선언하여 일반 Point를 반환하는 뷰 메소드를 public으로 추가하는 것이다.

### 일관성

두 객체가 같다면 불변이라면 계속 쭉~~ 같거나 달라야 한다는 특성이다.

다만 **equals 구현에 신뢰할 수 없는 자원이 들어가서는 안된다.**

java.net.URL에서는 URL과 호스트 IP를 활용하는데 고정IP가 아니라 유동IP라면 문제가 생긴다. 신뢰할 수 없는 자원은 메모리에 존재하는 객체만을 사용하는 결정적(deterministic) 계산만 수행해야 한다는 것이고 외부의 개입이 있는 자원간의 비교는 지양하는 것이다!

### null-아님

이름처럼 모든 객체가 null과 같지 않아야 한다는 것이다. 그런데 이를 지키기 위해서 임의적으로 null과의 비교로 return false;를 할 필요는 없다. instanceof로 묵시적 null 검사를 할 수 있기 때문이다.

<br>

### 순서 정리

- == 연산자를 사용해 입력이 자기 자신인지 확인해보자: 성능 최적화용으로 매우 좋다.
- instanceof 연산자로 입력이 올바른 타입인지 확인하자: 자신의 클래스일수도 있고 자신의 인터페이스가 될 수도 있다. Set, List, Map.Entry 등이 이에 해당한다.
- 입력을 올바른 타입으로 형변환한다: 2번으로 ClassCastException이 발생하지 않는다.
- 입력 객체와 자기 자신의 대응되는 ‘핵심’필드들이 모두 일치하는지 하나씩 검사한다: 2단계에서 인터페이스를 사용했다면 필드 값을 가져올 때도 인터페이스 메소드를 사용해야 함을 주의하자.

<br>

### 유의할점

- float과 double을 제외한 기본 필드는 ==로 비교하고 참조타입은 equals로 비교한다. float과 double타입은 compare 메소드로 비교한다. Float.NaN, -0.0f와 같은 outlier가 있기 때문이다. equals를 사용할 수도 있는데 연산 과정에서 오토박싱이 일어날 수 있어서 성능상 좋지 않을 것이다.
- null을 정상적으로 받아야 할 경우라면 Objects.equals(Object, Object)로 비교해서 NullPointException 발생을 예방하자.
- 앞의 CaseInsensitiveString처럼 비교하기 복잡한 필드를 가진 클래스도 있다. 필드의 표준형을 저장하여 그것끼리 비교하면 경제적이다.
- 필드가 많다면 비용이 싼 필드부터 비교하자. 비싼 필드를 먼저 비교하게 되면 시간이 늘어진다.
- 대칭성, 추이성, 일관성을 만족하는지 확인하고 `단위 테스트`를 작성해서 돌려보자.
- equals를 정의할때는 hashCode도 반드시 재정의해야 한다.
- 필드의 동치성만 확인해도 충분하고 alias name과 같은 것은 비교하지 않는 것이 좋다.
- equals의 입력 값은 반드시 Object여야 한다. 다른 클래스로 받게 되면 Override가 아니라 다중정의를 한 것이 된다. @Override Annotation을 이용하면 이러한 실수를 방지할 수 있다.

> 핵심 정리
> 
> 
> 꼭 필요한 경우가 아니면 equals를 재정의하지 말자. 대부분은 잘 비교해주고 재정의할거면 위의 다섯 가지 규칙을 잘 지켜서 정의하자.
>
