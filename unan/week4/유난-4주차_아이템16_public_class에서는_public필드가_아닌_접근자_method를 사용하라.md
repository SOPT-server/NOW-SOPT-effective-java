#  아이템 16 public 클래스에서는 public 필드가 아닌 접근자 method를 사용하라.

``` Java
class Point {

public double x;

public double y;

}
```

캡슐화가 필요함.

```Java
class Point {

private double x;
private double y;

public Point(double x, double y) {
	this.x = x;
	this.y = y;
}

public double getX() { return x; }
public double getY() { return y; }

}
```

패키지 바깥에서 접근할 수 있는 클래스라면 접근자를 제공


package-private, private 중첩 클래스라면 데이터 필드를 노출해도 문제없음. 가끔 나을때도 있지만... 신중하게 .. (예시 java.awt.package  Point와 Dimension -> Dimension 클래스는 내부를 노출해서 심각한 성능문제가 있음.) 



public 클래스의 필드가 불변인 경우도 ? public 지양하자.

```Java
public final class Time {

private static final int HOURS_PER_DAY = 24;

private static final int MINUTES_PER_HOUR = 60;

public final int hour;
public final int minute;


public Time(int hour, int minute) {
...
}
```
