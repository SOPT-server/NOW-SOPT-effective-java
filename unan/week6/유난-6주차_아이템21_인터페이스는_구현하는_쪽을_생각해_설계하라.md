# 아이템 21. 인터페이스는 구현하는 쪽을 생각해 설계하라

자바8 -> default method로 구현체 수정 없이 구현 가능해짐.
하지만 자바 7까지는 인터페이스에 새로운 메서드가 추가될일은 없다고 설계함. -> 매끄럽게 연동되기 어려움.

```java

default boolean removeIf(Predicate<? super E> filter) {
	Objects.requireNonNull(filter);
	boolean result = false;
	for (Iterator<E> it = iterator(); it.hasNext(); ) {
		if (filter.test(it.next())) {
			it.remove();
			result = true;
		}
	}
	return result;
}
```

해당 method는 org.apache.commons.collections4.collection.SynchronizedCollection이다. 아파치 커먼즈 라이브러리의 이 클래스는 java.util의 Collections.synchronizedCollection 정적 팩터리 메서드가 반환하는 클래스가 비슷하다.


아파치 버전은 클라이언트가 제공한 객체로 락을 거는 능력을 추가로 제공한다.

default method는 컴파일에 성공해도 기존 구현체에 런타임 오류를 일으킬 수 있다.
- default method 도구가 생겼더라도 인터페이스를 설계할 때는 세심한 주의를 기울여야 한다.
- 새로운 인터페이스라면 반드시 테스트를 해야함.
- 인터페이스를 릴리스한 한 후라도 결함을 수정하는게 가능한 경우도 있지만, 그 가능성에 기대지 말자.
