# 아이템 23. 태그 달린 클래스보다는 클래스 계층구조를 활용하라.

두 가지 이상의 의미를 표현할 수 있고, 현재 표현하는 의미를 Tag 값으로 알려주는 클래스

```Java

class Figure {
	enum Shape { RECTANGLE, CIRCLE} ;

	// 태그 필드
	final Shape shape;


	// 모양이 사각형일 때만 사용
	double length;
	double width;

	// 모양이 원일 때만 사용
	double radius;

	// 원용 생성자
	Figure(double radius) {
		shape = Shape.CIRCLE;
		this.radius = radius;
	}

	// 사각형용 생성자
	Figure(double length, width) {
		shape = Shape.RECTANGLE;
		this.length = length;
		this.width = width;
	}

	double area() {
		switch(shape) {
		case RECTANGLE:
			return length * width;
		case CIRCLE:
			return Math.PI * (radius * radius)
		default:
			throw new AssertionError(shape);
		}
	
	}
}
```

### 단점
1. enum, 태그 필드, switch 등 불필요한 코드가 ㅁ낳아짐.
2. 가독성이 나빠짐.
3. 수반되는 코드가 많아져 메모리 사용량도 증가함.
4. 필드들을 final로 선언할려면 해당 의미에 쓰이지 않는 필드들도 생성자에서 초기화해야함.


=> 태그 달린 클래스는 오류를 내기 쉽고, 비효율적


### 태그 달린 클래스를 계층구조로 변경하는 방법
1. 계층 구조의 루트가 될 추상 클래스를 정의한다.
2. 태그 값에 따라 동작이 달라지는 메서드들을 루트 클래스의 추상 메서드로 선언한다.
3. 태그 값에 상관없이 동작이 일정한 메서드 들을 루트 클래스에 일반 메서드로 추가한다.
4. 루트 클래스를 확장한 구체 클래스를 의미별로 하나씩 정의한다.


```java

abstract class Figure {
	abstract double area();
}

class Circle extends Figure {
	final double radius;
	
	Circle(double radius) {
		this.radius = radius;
	}

	@Override double area() {
		return Math.PI * (radius * radius);
	}
}

class Rectangle extends Figure {
	final double length;
	final double width;

	// 생성자
	...

	@Override double area() {
		return length * width;
	}
}
```

위와 같은 구현 -> 유연성 + 타입 검사 능력

만약 정사각형을 지원하도록 수정하려면?
```java

class Square extends Rectangle {
	Square(double side) {
		super(side, side);
	}
}
```
