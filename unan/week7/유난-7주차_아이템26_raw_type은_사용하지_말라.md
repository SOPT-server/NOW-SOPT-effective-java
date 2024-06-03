# 아이템 26. raw type은 사용하지 말라.

generic, generic interface : 클래스와 인터페이스 선언에 타입 매개변수(type parameter)까 쓰인 경우

제네릭 클래스, 인터페이스를 통틀어 generic type이라고 한다.

제네릭 타입은 일련의 parameterized type을 정의한다.

마지막으로 제네릭 타입을 하나 정의 하면 그에 딸린 Raw type도 함께 정의된다.

 Raw type : 제네릭 타입에서 타입 매개변수를 전혀 사용하지 않을 때,
 Raw Type은 타입 선언에서 제네릭 타입 정보가 전부 지워진 것처럼 동작함.


#### 잘못된 예시

```Java

// Stamp 인스턴스만 취급한다.
private final Collection stamps = ...;


```

stamp 대신 coin을 넣어도 아무 문제 없이 실행된다.


#### 잘못된 예시

```Java
for (Iterator i = stamps.iterator(); i.hasNext();) {
  Stamp stamp = (Stamp) i.next(); // ClassCastException을 던진다.
  stamp.cancel();
}
```

매개변수화된 컬렉션 타입 -> 타입 안정성 확보
```Java
private final Collection<Stamp> stamps = ...;
```

Raw Type을 쓰는 것을 언어 레벨에서 막지 않았지만, 절대로 쓰지말자.
Raw type을 사용하면 제네릭이 안겨주는 안전성과 표현력을 모두 잃게 된다.

그럼 Raw Type이 존재하는 이유는?
-> 제네릭이 없는 동안 작성된 코드들과의 호환성

List와 같은 Raw type은 안되지만, List<Object> 처럼 임의 객체를 허용하는 매개변수화 타입은 괜찮다.


### 원소의 타입을 몰라도 되는 Raw Type을 쓰고 싶은 경우 -> 비한정적 와일드카드 타입(unbounded wildcard type)

```java
static int numElementsInCommon(Set<?> s1, Set <?> s2) {
	...
}
```

차이점
	- raw type 컬렉션은 아무 원소나 넣을 수 있어서 불변식을 훼손하기 쉽다.
	- Collection<?> null 외에는 어떤 원소도 넣을 . 수없다.

### Raw type을 쓰자의 예외
1. Class 리터럴에는 raw type을 쓰자.
	- List.class, String[].class 허용. List<String>.class , List<?.class는 허용하지 않음.
2. instanceOf 연산자
	- 런타임에는 제네릭 타입 정보가 지워지기 때문에 instanceOf 연산자는 비한정적 와일드카드 타입 이외의 매개변수화 타입에는 적용할 . 수없다.

```java
	if (o intanceOf Set) { //. raw 타입
		Set<?> s = (Set<?>) o; // wildcard. 타입
		...
	}
```
o의 타입이 Set임을 확인하고, 형 변환해야 컴파일러 경고가 뜨지 않는다.

### 정리

Raw type -> 런타임 예외를 유발하니 사용하지 말자. (호환성 때문에 남아있음.)
Set<Object> -> 어떤 타입의 객체도 저장 가능
Set<?> 모종의 타입 객체만 저장 가능
Raw 타입인 Set은 제네렉 타입 시스템에 속하지 않음.

|   |   |   |   |
|---|---|---|---|
|매개변수화 타입|parameterized type|List<String>|아이템 26|
|실제 타입 매개변수|actual type parameter|String|아이템 26|
|제네릭 타입|generic type|List<E>|아이템 26,29|
|정규 타입 매개변수|formal type parameter||아이템 26|
|비한정적 와일드카드<br><br>타입|unbounded wildcard typ|List<？>|아이템 26|
|로 타입|raw type|List|아이템 26|
|한정적 타입 매개변수|bounded type parameter|<E extends Number>|아이템 29|
|재귀적 타입 한정|recursive type bound|<T extends ComparablecT>> 아이템 30||
|한정적 와일드카드<br><br>타입||bounded wildcard type List<? extends Number>|아이템 31|
|제네릭 메서드|generic method|static <> List<E><br><br>asList(E[] a)|아이템 30|
|타입 토큰|type token|String.class|아이템 3|
