### Objects
Java의 Objects -> 객체를 만들 수 있는 구체 클래스이지만 기본적으로 상속해서 사용.
따라서 equals, hashCode, toString, clone, finalize는 모두 재정의를 염두에 두고 설계된 것. -> 정의시 지켜야하는 일반 규약을 지켜서 재정의해야한다. 


재정의하지 않는 것이 좋은 상황

1. 각 인스턴스가 본질적으로 고유하다. 
	- 값을 표현하는 것이 아닌 동작하는 개체를 표현하는 class ex) Thread
2. 인스턴스의 논리적 동치성을 검사할 일이 없다.
	- 논리적 동치성 검사 예시) 정규 표현식
3. 상위 클래스에서 재정의한 equals가 하위클래스에도 딱 들어 맞는다.
	- 상위 클래스에서 재정의한 equals는 대부분 하위 클래스에도 맞는다.
		- AbstarctSet, Set/ AbstractMap, Map
4. 클래스가 private이거나 package-private이고 equals 메서드를 호출할 일이 없다.
	- 위험을 피하고 싶을 때
	
```Java
	@Override public boolean equals(Object o) {
		throw new AssertionError();
	}
```

equals 메소드는 동치관계(equivalence relation)를 구현해야함.
- Reflectivity
- Symmetry
- Transitivity
- Consistency
- Non-null

 ## equals 구현 방법

1. == 연산자로 입력이 자기 자신의 참조인지 확인.
2. instanceof 연산자로 입력이 올바른 타입인지 확인.
3. 입력을 올바른 타입으로 형변환한다.
4. 입력 객체와 자기 자신의 대응되는 핵심 필드들이 모두 일치하는지 하나씩 검사

float, double을 제외한 기본 타입 필드는 == 로  비교, 참조타입 필드는 각각의 equals로, float, double은 static method Float.compare(), Double.compare()로 비교
(Float.NaN, -0.0f, 특수한 부동소수 값을 다루기 위해서)
배열 필드는 원소 각각을 비교하되, 모든 원소가 핵심필드이면 Arrays.equals()

비교하기가 아주 복잡한 필드를 가진 클래스는 필드의 표준형(canocial form)을 저장해둔 후 표준형끼리 비교.

**구현 완료하면 단위테스트 작성하기!**

규약을 지켜 구현한 equals 예시
```Java
public final class PhoneNumber {

	private final short areaCode, prefix, LineNum;

}

public PhoneNumber(int areaCode, int prefix, int lineNum) {
	this.areaCode = rangeCheck(areaCode, 999, "지역코드");
	this.prefix= rangeCheck(prefix, 999, "프리픽스");
	this.lineNum =rangeCheck(LineNum, 9999, "가입자 번호");
}

private static short rangeCheck(int val, int max, String arg) {
	if (val < 0 || val > max)
		throw new IllegalArgumentException(arg + "; " + val);
		return (short) val;
}

@Override public boolean equals (Object o) {}
	if (o = this)
		return true;
if (! (0 instanceof PhoneNumber) )
	return false;
PhoneNumber pn = (PhoneNumber) o;
return pn.lineNum == LineNum && pn.prefix == prefix
		&& pn.areaCode = areaCode;
}
	••• // 나머지 코드는 생략
```

### 주의사항
- equals를 재정의할 때는 hashCode도 반드시 재정의하자
- 너무 복잡하게 해결하려 들지 말자. -> field의 동치성만 비교해도 충분
- Object외의 타입을 매개변수로 받는 equals method는 선언하지 말자.
