# Comparable을 어떻게 구현할지 고려하라.

Comparable에는 `compareTo` 라는 method가 존재한다.

```Java

Comparable<T> {  
    /**  
     * Compares this object with the specified object for order.  Returns a     * negative integer, zero, or a positive integer as this object is less     * than, equal to, or greater than the specified object.     *     * <p>The implementor must ensure {@link Integer#signum  
     * signum}{@code (x.compareTo(y)) == -signum(y.compareTo(x))} for  
     * all {@code x} and {@code y}.  (This implies that {@code  
     * x.compareTo(y)} must throw an exception if and only if {@code  
     * y.compareTo(x)} throws an exception.)  
     *     * <p>The implementor must also ensure that the relation is transitive:     * {@code (x.compareTo(y) > 0 && y.compareTo(z) > 0)} implies  
     * {@code x.compareTo(z) > 0}.  
     *     * <p>Finally, the implementor must ensure that {@code  
     * x.compareTo(y)==0} implies that {@code signum(x.compareTo(z))  
     * == signum(y.compareTo(z))}, for all {@code z}.  
     *     * @apiNote  
     * It is strongly recommended, but <i>not</i> strictly required that  
     * {@code (x.compareTo(y)==0) == (x.equals(y))}.  Generally speaking, any  
     * class that implements the {@code Comparable} interface and violates  
     * this condition should clearly indicate this fact.  The recommended     * language is "Note: this class has a natural ordering that is     * inconsistent with equals."     *     * @param   o the object to be compared.  
     * @return  a negative integer, zero, or a positive integer as this object  
     *          is less than, equal to, or greater than the specified object.     *     * @throws NullPointerException if the specified object is null  
     * @throws ClassCastException if the specified object's type prevents it  
     *         from being compared to this object.     */    public int compareTo(T o);  
}

```

알파벳, 숫자, 연대 같이 순서가 명확한 값 클래스를 작성한다면, 반드시 Comparable을 구현

규약 정리
- Symmetric
- Transitivity
- Reflexivity
- (필수X) compareTo 메서드로 수행한 동치성 테스트 결과가 equals랑 같아야함.


아이템 10 PhoneNumber compareTo 구현 예시

```Java
public int compareTo (PhoneNumber pn) {
	int result = Short.compare(areaCode, pn.areaCode);

	if (result = 0) {
		int result = Short.compare(prefix, pn.prefix); // 가장 중요한 필드
		if (result = 0) // 두 번째로 중요한 필드
			result= Short.compare(lineNum, pn. LineNum); // 세 번째로 중요한 필드
	}
	return result;
}
```

Java 8에서 method chaining 방식으로 비교자를 생성할 수 있음.

다음은 PhoneNumber compareTo static 생성 메소드
```Java
	private static final Comparator<PhoneNumber> COMPARATOR = comparingInt ( (PhoneNumber pn) - pn.areaCode)
	.﻿﻿thenComparingInt (pn →> pn.prefix)
	.﻿﻿thenComparingInt (pn →> pn. lineNum);

public int compareTo (PhoneNumber pn) {
	return COMPARATOR. compare(this, pn);
```

정적 compare method를 활용한 비교자
```Java
static Comparator<Object> hashCodeOrder = new Comparator<>() {
	public int compare(Object o1, Object o2) {
		return Integer.compare(o1.hashCode(), o2.hashCode());
	}
}
```

비교자 생성 메소드를 활용한 비교자
```Java
static Comparator<Object> hashCodeOrder = Comparator.comparingInt(
	o -> o.hashCode();
)
```

### 핵심정리
- 순서를 고려하는 값은 Comparable을 구현하자.
- compareTo 메서드에서 필드의 값을 비교할 때 <와 > 연산자는 쓰지말자. 박싱된 기본 타입 클래스가 제공하는 static compare 메소드나 Comparator 인터페이스가 제공하는 비교자 생성 메소들르 사용하자.
