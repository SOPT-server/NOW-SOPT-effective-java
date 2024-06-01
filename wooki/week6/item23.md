**`태그 달린 클래스`**: 두 가지 이상의 의미를 표현하며, 그 중 현재 표현하는 의미를 태그 값으로 알려주는 클래스

태그 달린 클래스 예시) Figure: 사각형 혹은 원을 나타내는 클래스

```java
class Figure {
    enum Shape { RECTANGLE, CIRCLE };

    // 태그 필드 - 현재 모양을 나타낸다.
    final Shape shape;

    // 다음 필드들은 모양이 사각형(RECTANGLE)일 때만 쓰인다.
    double length;
    double width;

    // 다음 필드는 모양이 원(CIRCLE)일 때만 쓰인다.
    double radius;

    // 원용 생성자
    Figure(double radius) {
        shape = Shape.CIRCLE;
        this.radius = radius;
    }

    // 사각형용 생성자
    Figure(double length, double width) {
       shape = Shape.RECTANGLE;
        this.length = length;
        this.width = width;
    }

    double area() {
        switch(shape) {
            case RECTANGLE:
                return length * width;
            case CIRCLE:
                return Math.PI * (radius * radius);
            default:
                throw new AssertionError(shape);
        }
    }
}
```

각각의 관점에 따라 쓸데 없는 코드가 한 곳에 모인다. - 열거 타입 선언, 태그 필드, switch문 등등

가독성이 떨어진다. - 여러 구현이 한 클래스에 혼합

메모리 관점에서도 인스턴스를 만들 때 불필요한 필드에 대한 메모리를 사용하게 된다.

인스턴스의 타입만으로는 어떤 shape인지 알 수 없다.

→ 단점만 많은 구조이니 `계층구조`를 사용하자!

계층구조 사용)

```java
abstract class Figure {
    abstract double area();
}
```

```java
class Circle extends Figure {
    final double radius;

    Circle(double radius) { this.radius = radius; }

    @Override double area() { return Math.PI * (radius * radius); }
}
```

```java
class Rectangle extends Figure {
    final double length;
    final double width;

    Rectangle(double length, double width) {
        this.length = length;
        this.width  = width;
    }
    
    @Override double area() { return length * width; }
}
```

쓸데 없는 코드가 모두 사라졌다

간결하고 명확하게 알아볼 수 있다

각 의미를 독립된 클래스에 담아 관련 없던 데이터 필드를 모두 제거했고, 살아남은 필드는 모두 final이다. → 각 클래스의 생성자가 모든 필드를 초기화하고 추상 메서드를 모두 구현했는지 컴파일러가 확인해준다.
