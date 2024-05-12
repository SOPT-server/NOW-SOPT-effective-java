### 캡슐화의 이점
API 를 수정하지 않고 내부 표현을 바꿀 수 있다
불변식을 보장할 수 있다
외부에서 필드에 접근할 때 부수 작업을 수행할 수 있다
``` java
public class Point {
private double x;
private double y;

    public Point(double x, double y) {
        this.x = x;
        this.y = y;
    }

    public double getX() {
        return x;
    }

    public void setX(double x) {
        this.x = x;
    }

    public double getY() {
        return y;
    }

    public void setY(double y) {
        this.y = y;
    }
}
```
public 클래스의 필드가 불변(const, final)이라면 직접 노출할 때의 단점이 조금은 줄어들지만 좋은 생각은 아니다.
API를 수정하지 않고 내부 표현을 바꿀수 없다는 점과 필드에 접근할 때 부수 작업을 수행할 수 없다는 점은 여전하다.

>public 클래스는 절대 가변필드를 직접적으로 노출해서는 안된다. 하지만, package-private 클래스나 private 중첩 클래스에서는 종종 필드를 노출하는 편이 나을때도 있다.