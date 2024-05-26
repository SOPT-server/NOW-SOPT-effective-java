> equals를 재정의했다면 hashCode도 재정의하자 꼭! (아니면 제대로 동작안한다…)
재정의한 hashCode는 Object의 API 문서에 기술된 일반 규약을 따라야 한다.
IDE나 구글의 AutoValue의 도움을 받자!
>

Object 명세에서 발췌한 규약

- equals 비교에 사용되는 정보가 변경되지 않았다면, 애플리케이션이 실행되는 동안 그 객체의 hashCode 메서드는 몇 번을 호출해도 일관되게 항상 같은 값을 반환해야 한다.
  단, 애플리케이션을 다시 실행한다면 이 값은 달라져도 상관없다.
- equals(Object)가 두 객체를 같다고 판단했다면, 두 객체의 hashCode는 똑같은 값을 반환해야 한다.
  → `논리적으로 같은(동등한) 객체는 같은 해시코드를 반환해야 한다.`
- equals(Object)가 두 객체를 다르다고 판단했더라도, 두 객체의 hashCode가 서로 다른 값을 반환할 필요는 없다.
  단, 다른 객체에 대해서는 다른 값을 반환해야 해시테이블의 성능이 좋아진다.

# hashCode 구현 사례

## 최악이지만 적법한 구현 - 사용 금지!

```java
@Override
public int hashCode() {
		return 42;
}
```

동치인 모든 객체에 똑같은 해시코드를 반환하니 적법하긴 하지만, 모든 객체에게 같은 값만 내어주므로 모든 객체가 해시 테이블의 버킷 하나에 담겨 마치 linked list처럼 동작한다.

이렇게 되면 평균 수행시간이 O(1)인 해시 테이블이 O(N)으로 느려져서, 객체가 많아지면 도저히 쓸 수 없다.

→ 해시 테이블은 객체를 해시값으로 매핑하여 빠르게 검색/삽입/삭제를 가능하게 하는 자료구조인데, 해시 값이 같은 경우 `hash collision`이 발생하고 같은 버킷에 저장하게 된다. 일반적으로 hash collision은 충돌 객체를 다른 버킷으로 재할당하는 `rehashing`을 통해 해결하지만, 위 코드에서는 모든 객체에 대해 `42`라는 동일한 해시값을 할당하여 결국 모든 객체가 같은 버킷에 충돌하게 된다.

## 전형적인 hashCode 메서드

1. int 변수인 `result`를 선언한 후 값을 c로 초기화한다.
    - 이 때, c는 해당 객체의 첫번째 핵심 필드를 단계 2.1 방식으로 계산한 해시코드이다.
    - 여기서 핵심 필드는 `equals` 비교에 사용되는 필드를 말한다. (item 10 참고)
2. 해당 객체의 나머지 핵심 필드인 f 각각에 대해 다음 작업을 수행한다.
    1. 해당 필드의 해시코드 c 를 계산한다.
        - 기본 타입 필드라면, `Type.hashCode(f)`를 수행한다. 여기서 *Type*은 해당 기본타입의 박싱 클래스다.
        - 참조 타입 필드면서, 이 클래스의 `equals` 메소드가 이 필드의 `equals`를 재귀적으로 호출하여 비교한다면, 이 필드의 `hashCode`를 재귀적으로 호출한다.
        - 필드가 배열이라면, 핵심 원소 각각을 별도 필드처럼 다룬다.모든 원소가 핵심 원소라면 `Arrays.hashCode`를 사용한다.
    2. 단계 2.1에서 계산한 해시코드 c로 `result`를 갱신한다.
        - `result = 31 * result + c;`
3. `result`를 반환한다.

```java
@Override
public int hashCode() {
		int result = Short.hashCode(areaCode);
		result = 31 * result + Short.hashCode(prefix);
		result = 31 * result + Short.hashCode(lineNum);
		return result;
}
```

PhoneNumber 인스턴스의 핵심 필드 3개만을 사용하여 비결정적인(undeterministic) 요소는 전혀 없이, 동치인 PhoneNumber 인스턴스들은 같은 해시코드를 가진다.

hash collision이 더욱 적은 방법을 써야 한다면 Guava의 `com.google.common.hash.Hashing`을 참고하자

## Objects 클래스에서 제공하는 hash를 이용한 hashCode 메서드

Objects 클래스에서 제공하는 정적 메서드 hash를 이용한 hashCode이다.

입력 인수를 담기 위한 배열이 만들어져야하고, 기본 타입이 있는 경우 박싱/언박싱도 거쳐야 하기 때문에 속도는 더 느리다. 성능에 민감하지 않은 경우만 사용하자.

```java
@Override
public int hashCode() {
		return Objects.hash(lineNum, prefix, areaCode);
}
```

불변 클래스이고 해시코드 계산 비용이 크다면 캐싱하여 사용하도록 하자.

## 해시코드를 lazy initialize 하는 hashCode 메서드

`지연 초기화 방식`으로 객체 생성 시 hashCode값을 계산하지 않고, hashCode 메서드가 호출될 때 계산하여 초기화한다.
→ 지연 초기화하려면 그 클래스의 스레드를 안전하게 만들어야 한다.

```java
private int hashCode;    //자동으로 0으로 초기화

@Overrride 
public int hashCode() {
		int result = hashCode;
		if (result == 0) {
				int result = Short.hashCode(areaCode);
				result = 31 * result + Short.hashCode(prefix);
				result = 31 * result + Short.hashCode(lineNum);
				hashCode = result;
		}
		return result;
}
```

# 주의 사항

## 성능을 높인답시고 해시코드 계산 시 핵심 필드를 생략하지 말자.

속도는 빨라지겠지만, 해시의 품질이 나빠져서 해시 테이블의 성능이 심각하게 안좋아질 수 있다.

## hashCode가 반환하는 값의 생성 규칙을 API 사용자에게 자세히 공표하지 말자.

클라이언트가 이 값에 의지하지 않게 되고, 추후에 계산 방식을 바꿀 수도 있기 때문이다.

# JPA에서 실패 사례

```java
@Test
void equalsTest() {
    EntityManager entityManager = entityManagerFactory.createEntityManager();
    EntityTransaction transaction = entityManager.getTransaction();
    transaction.begin();

    Item cake = Item.builder()
        .name("Cake")
        .price(2500)
        .stockQuantity(10).build();
    entityManager.persist(cake);
    transaction.commit();
    entityManager.clear();

    Item findCake = entityManager.find(Item.class, cake.getId());
    assertEquals(cake, findCake);
}
```

영속성 컨텍스트에서 1차 캐시를 이용하여 같은 ID의 엔티티를 항상 같은 객체로 가져올 수 있는데, DB 반영하고 flush하는 등의 경우에는 1차 캐시가 초기화되기 때문에 동일한 엔티티를 읽어오더라도 서로 다른 객체일 수 있다.
