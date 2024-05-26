# public 클래스에서는 public 필드가 아닌 접근자 메서드를 사용하라

## public 클래스의 공개된 필드

```java
class Point {
    public double x;
    public double y;
}
```

public 클래스에서 데이터 필드에 직접 접근할 수 있다면 캡슐화의 이점을 제공하지 못하는 것이다. 불변식 보장도 불가능하며 외부에서 필드에 접근할 때 부수 작업을 수행할 수도 없다. public 클래스가 필드를 public으로 공개하면 이를 사용하는 클라이언트가 생겨나므로 내부 표현 방식을 바꿀 수 없어진다.

## 접근자 메소드 사용

객체 지향 프로그래머는 다음과 같이 수정하도록 하자

```java
class Point {
    private double x;
    private double y;

    public Point(double x, double y) {
        this.x = x;
        this.y = y;
    }

    public double getX() {return x;}
    public double getY() {return y;}

    public void setX() {this.x = x;}
    public void setY() {this.y = y;}
}
```

위와 같이 변경함을 통해 클래스 내부 표현 방식을 언제든 바꿀 수 있어진다. (클라이언트가 접근자 메소드를 통해서만 접근하는 것이 보장되므로)

public 클래스의 가변 필드는 직접 노출하지 말자. 불변 필드라면 크게 상관없을 수 있지만 몇몇 제약이 생길 수는 있다.

## 예외

간혹 위 규칙이 안 지켜져도 있을 때가 있는데 package-private 클래스 혹은 private 중첩 클래스가 그 예이다.

위 방식들 같은 경우는 접근자를 사용하는 것보다 직접 노출하는 것이 나을 수 있다.
