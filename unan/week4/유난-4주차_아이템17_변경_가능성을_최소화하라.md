# 아이템 17. 변경 가능성을 최소화하라

불변 클래스 : 인스턴스 내부 값을 수정할 수 없는 클래스
Java 불변 클래스 : String, Wrapping Class, BigInteger, BigDecimal

불변 클래스를 만드는 규칙

1. Setter를 제공하지 않는다.
2. 클래스를 확장할 수 없도록 한다.
3. 모든 필드를 final로 선언한다.
4. 모든 필드를 private으로 선언한다.
	- 직접 필드 수정하는 것을 막자.
5. 클래스에 가변 객체를 잠조하는 필드가 하나라도 있다면, 클라이언트에서 그 객체의 잠조를 얻을 수 없도록 해야한다. 해당 필드가 클라이언트가 제공한 객체 참조를 가리키게 해서는 안 됨.
	- 생성자, 접근자, readObject method 모두 방어적 복사 수행

```Java
public final class Complex {

private final double re;
private final double im;

public Complex(double re, double im) {
	this.re = re;
	this. im = im;
}

public double realPart() {
	return re;
}

public double imaginaryPart() { return im; }

public Complex plus (Complex c) {
	return new Complex(re + c.re, im + c. im);
}

public Complex minus (Complex c) {
	return new Complex(re - c.re, im - c. im);

public Complex times (Complex c) {
	return new Complexre * c.re - im * c. im, re * c.im + im * c.re);
}

public Complex dividedBy(Complex c) {
	double tmp = c.re * c.re + c. im * c. im;
	return new Complex((re * c.re + im * c. im) / tmp, (im * c.re - re * c. im) / tmp);
	}

@Override public boolean equals(Object o) {
	if (o = this)
		return true;
	if (!(o instanceof Complex))
		return false;
	Complex c = (Complex) 0;
	// == 대신 compare를 사용하는 이유는 63쪽을 확인하라.
	return Double.compare(c.re, re) == 0 && Double.compare(c. im, im) == 0;
}

@Override public int hashCode() {
	return 31 * Double. hashCode(re) + Double. hashCode (im);
}

@Override public String toString() {
	return "(" + re + " + " + im + "i)";
	}
}
```

피연산자에 함수를 적용해 결과를 반환하지만 새로운 인스턴스를 반환한다.(함수형)
-> 코드에서 불변이되는 영역의 비율이 높아지는 장점.
불변 객체는 단순하지만, 가변 객체는 복잡한 상태에 놓일 수 있다.

불변 객체는 기본적으로 Thread-Safety -> 동기화 필요 X

객체를 만들 때 다른 불변 객체들을 구성요소로 사용하면 이점이 많다.
1. 맵, Set의 원소로 쓰기 좋다.
2. 불변 객체는 . 그자체로 실패 원자성을 제공한다.

### 단점
 값이 다르면 반드시 독립된 객체로 만들어야 함. -> 비용 증가할 . 수있다.

해결방법
.multistep operation을 제공
	- ex) BigInterger modulo
 

불변 클래스 설계 방법
- 모든 생성자를 private, package-private으로 만들고, 정적 팩토리를 제공

```Java
public class Complex {
	private final double re;
	private final double im;


	private Complex(double re, double im) {
		this.re = re;
		this.im = im;
		
	}

	public static Complex valueOf(double re, double im) {
		return new Complex(re, im);
	}
}
```

## 정리
- 클래스는 꼭 필요가 아니라면 불변
- Setter 조심
- 성능 때문에 가변 클래스를 만들어야 한다면 쌍을 이루는 동반 클래스를 제공
- 모든 클래스를 불변으로 만들 수는 없으니 최소화 하는 방향
- 다른 합당한 이유가 없다면 모든 필드는 private final
- 생성자는 불변식 설정이 모두 완료된 초기화가 완벽히 끝난 상태의 객체를 생성

예시 java.util.concurrent.CountDownLatch 
