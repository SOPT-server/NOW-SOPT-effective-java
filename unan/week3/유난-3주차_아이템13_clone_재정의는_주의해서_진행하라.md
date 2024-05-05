# 아이템 13: clone 재정의는 주의해서 진행하라.

`Cloneable`은 복제해도 되는 클래스임을 명시하는 용도의 믹스인 인터페이스.
문제 : clone 메서드는 Object에 선언되있고, protected임.

```Java
@IntrinsicCandidate  
protected native Object clone() throws CloneNotSupportedException;
```

`Cloneable` 은 method하나 없는 인터페이슨데 무슨 역할?
- Object의 protected method인 clone()의 동작 방식을 결정함.
- `Cloneable`을 구현한 클래스의 인스턴스에서 clone을 호출 -> 그 객체의 필드들을 하나 하나 복사한 객체를 반환.
- 그렇지 않은 클래스에서 호출 -> `CloneNotSupportedException`

`Cloneable`을 구현한 클래스는 clone 메서드를 public으로 제공하고, 사용자는 당연히 복제가 제대로 이뤄지기를 기대함.

`clone` 의 일반 규약은 허술함.
강제성이 없다는 점을 빼면 생성자 연쇄와 비슷함.


불변 클래스는 굳이 clone 메소드를 제공하지 말자.(복사 지양)

PhoneNumber clone 구현 예제
```Java
@Override public PhoneNumber clone() {
	try {
		return (PhoneNumber) super.clone();
	} catch(CloneNotSupportedException e) {
		throw new AssertionError(); // 일어날 수 없는 일
	}
}
```

```Java
public class Stack {

private Object [] elements;

private int size = 0;

private static final int DEFAULT_INITIAL_CAPACITY = 16;

public Stack() {
	this.elements = new Object (DEFAULT_INITIAL_CAPACITY) ;
}

public void push(Object e) {
	ensureCapacity();
	elements (size++) = e;
}

public Object pop() {
		if (size = 0) throw new EmptyStackException();
		Object result = elements (-size];
		elements[size] = null; // 다 쓴 참조 해제
		return result;

}

// 원소를 위한 공간을 적어도 하나 이상 확보한다.

	private void ensureCapacity() {
		if (elements. length = size)
		elements = Arrays.copyOf(elements, 2 * size + 1);
	}
}								  								  ```
위 Stack의 경우 clone이 super.clone()을 호출하면, size는 올바른 값을 갖지만 elements 필드는 원본 Stack 인스턴스와 똑같은 배열을 참조함.
-> 원본, 복제본의 수정이 생겼을 때 NPE 발생 

Stack 클래스의 하나뿐인 생성자를 호출한다면 ? -> clone method가 생성자와 같은 효과를 낸다. 
i.e. clone은 원본 객체에 아무런 해를 끼치지 않는 동시에 복제된 객체의 불변식을 보장해야 한다.

- Stack clone() 구현 예제
```Java
@Override public Stack clone() {
	try {
		Stack result = (Stack) super.clone();
		result.elements = elements.clone();
		return result;
	} catch (CloneNotSupportedException e) {
		throw new AssertionError();
	｝
}
```
- elements 필드가 final이면 이런 방식이 동작하지 않음. -> `Cloneable`은 가변 객체를 참조하는 필드는 final로 선언하라는 일반 용법과 충돌


```Java
public class HashTable implements Cloneable {
		private Entry[] buckets = ...;

		private static class Entry {
		final Object key;
		Object value;
		Entry next;

		Entry(Object key, Object value, Entry next) {

		this. key = key;
		this. value = value;
		this.next = next;
		}
	}
	... // 나머지 코드는 생랽
}
```


HashTable clone 잘못된 구현 -> 가변 상태를 공유함.
```Java
@Override public HashTable clone() {
	try {
		HashTable result = (HashTable) super.clone();
		result.buckets = buckets.clone();
		return result;
} catch (CloneNotSupportedException e) i
		throw new AssertionError();
```
- 복제본은 자신만의 bucket 배열을 갖지만, 원본과 같은 연결리스트를 참조함. -> 복제시에 연결리스트도 복사해야함.

```Java
public class HashTable implements Cloneable {

	private Entry[] buckets = ...;

		private static class Entry {
			final Object key;
			Object value;
			Entry next;

			Entry(Object key, Object value, Entry next) {
				this.key = key;
				this.value = value;
				this.next = next;
			}
		// 이 엔트리가 가리키는 연결 리스트를 재귀적으로 복사
		Entry deepCopy () {
			return new Entry(key, value,
			next = null ? null : next.deepCopy());
		}
	
	@Override public HashTable clone() {
		try {
			HashTable result = (HashTable) super.clone();
			result. buckets = new Entry [buckets. length];
			for (int i = 0; i < buckets. length; i++)
				if (buckets [il != null)
					result.buckets [i] = buckets [il.deepCopy();
				return result;
} catch (CloneNotSupportedException e) {
		throw new AssertionError();
	}
}

...
// 나머지 코드는 생략

}
```

해당 코드의 단점 -> 재귀 호출 -> 리스트 원소수만큼 스택 프레임을 소비-> 스택 오버프로우 위험

```Java
Entry deepCopy() {
	Entry result = new Entry(key, value, next);
	for (Entry p = result; p.next != null; p = p.next)
		p.next = new Entry(p.next.key, p.next. value, p.next.next);
	return result;
}
```

복잡한 가변 객체를 복사할 때, 고수준 API를 사용하는 방법도 있지만, 성능 이슈가 있음.

### 주의할 점
- `Cloneable`을 구현한 Thread-Safety class를 작성할 때는 clone 메서드 역시 동기화 해야함.
- Object의 clone method는 CloneNotSupportedException을 던진다고 선언했지만, 재정의한 메서드 (public) clone method에서는 없애야 한다. (<- 사용 편의성)

### 복사 생성자, 복사 팩터리
`Cloneable`을 구현한 메소드는 clone을 반드시 재정의해줘야함. -> 복잡
인터페이스 기반 복사 생성자와 복사 팩터리의 -> 변환 생성자(conversion constructor), 변환 팩터리(conversion factory)


### 핵심 정리
Cloneable이 몰고 온 모든 문제를 되짚어봤을 때, 새로운 인터페이스를 만들 때는 절대 Cloneable을 확장해서는 안 되며, 새로운 클래스도 이를 구현해서는 안 된다. final 클래스라면 Cloneable을 구현해도 위험이 크지 않지만, 성능 최적화 관점에서 검토한 후 별다른 문제가 없을 때만 드물게 허용해야 한다(아이템 67). 기본 원칙은 '복제 기능은 생성자와 팩터리를 이용하는 게 최고'라는 것이다. 단 배열만은 clone 메서드 방식이 가장 깔끔한, 이 규칙의 합당한 예외라 할 수 있다.
