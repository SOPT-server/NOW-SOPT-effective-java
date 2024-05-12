### compareTo 메서드란?
Comparable 인터페이스의 유일무이한 메서드
equals와 비슷하지만 단순 동치성 비교에 더해 순서까지 비교할 수 있고 제네릭하다.
Comparable을 구현했다는 건 객체 간의 순서가 있다는 것이다.
순서를 고려해야 하는 값 클래스를 작성했다면 Comparable 인터페이스를 구현하여, 그 인스턴스들을 쉽게 정렬하고, 검색하고, 비교 기능을 제공하는 컬렉션들과 어우러지도록 해야 한다.

### compareTo 메서드의 일반 규약
이 객체와 주어진 객체의 순서를 비교한다. 이 객체가 주어진 객체보다 작으면 음의 정수를, 같으면 0을, 크면 양의 정수로 반환한다. 이 객체와 비교할 수 없는 타입의 객체가 주어지면 ClassCastException을 던진다.
>gn(x.compartTo(y)) == -sgn.(y.compareTo(x))
x.compareTo(y) > 0 && y.compareTo(z) > 0)이면 x.compareTo(z) > 0
x.compareTo(y) == 0이면 sgn(x.compareTo(z)) == sgn(y.compareTo(z))
x.compareTo(y) == 0) == (x.equals(y)) (필수 아님, ex.BigDecimal)
> 
Comparable을 구현하고 이 권고를 지키지 않는 모든 클래스는 문서에 명시를 잘 해두자.
equals과 같이 기존 클래스를 확장한 구체 클래스에서 새로운 값 컴포넌트를 추가했다면 compareTo 규약을 지킬 방법이 없다.
➡️ 컴포지션을 사용하자.

- compareTo를 구현하는 방법
    - Comparable을 구현하지 않은 필드나 표준이 아닌 순서로 비교해야 한다면, 비교자(comarator)를 만들어서 compare() 로 비교할 수 있다.
비교자는 직접 만들거나 java가 제공하는 것을 쓰면 된다.

    - 기본 타입 비교할 때
java7 전에는 정수 기본 타입을 비교할 때 >, <을 사용했다. java7 이후에는 박싱된 기본 타입 클래스들에 새로 추가된 정적 메서드인 compare를 이용하자.
(ex. Short.compare(lineNum, pn.lineNum) )


java8 비교자 생성 메서드로 비교자 만들기
java8에서는 비교자 생성 메서드를 제공한다. 비교자 생성 메서드로 비교자를 만들어 보자.

``` java
private static final Comparator<PhoneNumber> COMPARATOR =
Comparator.comparingInt((PhoneNumber pn) -> pn.areaCode) // (1)
.thenComparingInt(pn -> pn.getPrefix()) // (2)
.thenComparingInt(pn -> pn.lineNum); // (3)
```
(1) Comparator의 static 메서드 comparingInt는 키 추출함수를 매개로 받아 비교자 Comparator<PhoneNumber> 객체 반환
(2), (3) 키 추출자 함수를 받아 비교자 Comparator<PhoneNumber> 객체 반환
</br>

- 기본 타입</br>
Comparator는 long과 double용으로는 comparingInt와 thenComparingInt의 변형 메서드를 갖고 있다.
short처럼 더 작은 정수 타입에는 int용 버전을 사용하면 된다. 마찬가지로 float는 double용을 이용해 수행한다.

- 객체 참조 타입 </br>
객체 참조용 비교자 생성 메서드도 있다. comparing이라는 정적 메서드 2개가 다중정의되어 있다.
또한, thenComparing이란 인스턴스 메서드가 3개 다중정의되어 있다.


### 핵심정리
> 순서를 고려해야 하는 값 클래스를 작성한다면 꼭 Comparable 인터페이스를 구현하여 , 그 인스턴스들을 쉽게 정렬하고, 검색하고, 비교 기능을 제공하는 컬렉션과 어우러지도록 해야 한다 . 
> compareTo 메서드에서 필드의 값을 비교할 때 <와 > 연산자는 쓰 지 말아야 한다 .
> 그 대신 박싱된 기본 타입 클래스가 제공하는 정적 compare 메서드나 Comparator 인터페이스가 제공하는 비교자 생성 메서드를 사용하자.

