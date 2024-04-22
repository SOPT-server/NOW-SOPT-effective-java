# Item 06. 불필요한 객체 생성을 피하라.

똑같은 기능의 객체를 매번 생성하기보다는 객체 하나를 재사용하는 편이 나을때가 많다. 재사용은 빠르고 세련됨.

```Java

static boolean isRomanNumeral (String s) {
	return s.matches ("^ (?=.)M*(C[MD] |D?C{0, 3})" + "(X[CL] |L?X{0,3}) (I[XV] |V?I{0,3}) $");
}
```

Pattern 인스턴스를 재활용하기 위해서 아래와 같이 재구성 할 수 있다.

```Java
public class RomanNumerals {

	private static final Pattern ROMAN = Pattern.compile("^(?=.)M*(C[MD]|D?    C{0,3})" + "(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");

	static boolean isRomanNumeral(St ring s) {
		return ROMAN.matcher(s).matchesO;
	}
}
```

단순히 객체 생성을 피하자 X -> 불필요한 객체 생성을 피해서, 성능을 향상시키자.
Wrapper 타입 보다는 가능하다면 primitive 타입을 사용하자.
