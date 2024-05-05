## 아이템11. equals를 재정의하려거든 hashCode도 재정의하라.

<br>

equals를 재정의한 클래스에서는 hashCode도 재정의해야한다. 그렇지 않으면 HashMap이나 HashSet 같은 컬렉션의 원소로 사용할 때 문제를 일으킬 것이다. 

다음은 Object 명세에서 발췌한 규약이다. 

- equals 비교에 사용되는 정보가 변경되지 않았다면, 애플리케이션이 실행되는 동안 그 객체의 hashCode 메소드는 몇 번을 호출해도 일관되게 항상 같은 값을 반환해야 한다. 단, 애플리케이션을 재실행한다면 값이 달라져도 상관없다.
- equals(Object)가 두 객체를 같다고 판단했다면 두 객체의 hashCode는 똑같은 값을 반환해야 한다.
- equals(Object)가 두 객체를 다르게 판단했더라도 hashCode가 다를 필요는 없다. 단, 다른 객체에 대해서는 다른 값을 반환해야 해시테이블 성능이 좋아진다.

hashCode 재정의를 잘못했을 때 문제가 되는 조항은 두 번째 조항이다. 즉 논리적으로 같은 객체는 같은 해시코드를 반환해야한다. 

```java
Map<PhoneNumber, String> m = new HashMap<>();
m.put(new PhoneNumber(707, 867, 5309), "제니");
m.get(new PhoneNumber(707, 867, 5309));
```

이렇게 get으로 가져오면 어떤 값이 나올까? 예상은 제니가 나와야할 것 같지만 실제로는 null을 반환한다. 

PhoneNumber클래스는 hashCode를 재정의하지 않았기때문에 논리적 동치 관계인 두 객체가 서로 다른 해시코드를 반환하여 두번째 규약을 지키지 못하게 된다. 이 문제는 적절한 hashCode 메서드만 작성해주면 해결된다.

```java
@Override public int hashCode() { return 42; }
```

위 코드는 동치관계인 모든 객체에 같은 값을 반환해주니 아까의 문제는 해결되지만, 모든 객체가 같은 해시값을 가지게 되어 해시테이블의 버킷 하나에 담겨 연결리스트처럼 동작한다. 그 결과 평균수행시간이 O(1)에서 O(n)으로 느려지는 문제가 발생한다. 

따라서 좋은 해시 함수는 서로 다른 인스턴스에 다른 해시코드를 반환한다. 이상적인 해시함수는 주어진 인스턴스들을 32비트 정수 범위에 균일하게 분배해야한다. 그렇다면 좋은 hashCode를 작성하는 요령에 대해서 알아보자.
<br>

### 좋은 hashCode를 작성하는 요령
1. 기본타입 필드라면, Type.haskCode(f)를 수행
2. 참조타입 필드면서 이 클래스의 equals 메서드가 이 필드의 equals 메서드가 이 필드의 hashCode를 재귀적으로 호출한다. 복잡해질 것 같으면 표준형을 만들어서 그것의 hashCode를 호출해도 괜찮다. 필드 값이 null이면 0을 사용한다.
3. 필드가 배열이라면 배열 핵심 원소 각각을 별도로 다룬다. 배열에 핵심 원소가 없다면 0으로 처리해도 괜찮다. 모든 원소가 핵심이라면 Array.hashCode를 사용하라.
4. 그다음 값을 31 * result + c; 로 갱신하고 반환하면 된다.

이렇게 구현했다면 이 메서드가 동치인 인스턴스에 대해 똑같은 해시코드를 반환할지 고려해야한다. 파생필드는 해시코드 계산에서 제외해도 되고, equals 비교에 사용되지 않은 필드는 반드시 ! 제외해야한다. 

```java
@Override public int hashCode() {
    int result = Short.hashCode(areaCode);
    result = 31 * result + Short.hashCode(prefix);
    result = 31 * result + Short.hashCode(lineNum);
    return result;
}
```

위 메서드는 PhoneNumber에 딱 맞게 구현한 해시코드로, 인스턴스의 핵심 필드 3개만을 사용하여 간단한 계산만 수행한다. 

Objects 클래스는 임의의 개수만큼 객체를 받아 해시코드를 계산해주는 정적 메서드인 hash를 제공한다. 하지만 입력인수를 담기 위한 배열이 만들어지고, 입력 중 기본 타입이 있다면 박싱과 언박싱도 거쳐야하므로 속도는 더 느리다. 따라서 성능에 민감하지 않은 상황에서 사용해야한다.

```java
@Override public int hashCode() {
    return Objects.hash(lineNum, prefix, areaCode);
}
```

만약 클래스가 불변이고 해시코드를 계산하는 비용이 크다면, 캐싱을 이용해야한다. 이 타입의 객체가 해시의 키로 사용될 것 같다면 인스턴스가 만들어질때 해시코드를 계산해둬야한다. 해시의 키로 사용되지 않는 경우에는 지연 초기화 방식을 사용할 수 있다. 이렇게 필드를 지연 초기화하려면 스레드 안전하게 만들도록 신경써서 만들어야한다. 

```java
private int hashCode; // 자동으로 0으로 초기화된다

@Override public int hashCodeO {
    int result = hashCode;
    if (result = 0) {
        result = Short.hashCode(a reaCode);
        result = 31 * result + Short.hashCode(prefix);
        result = 31 * result + Short.hashCode(lineNum);
        hashCode = result;
    }
    return result;
}
```

<br>

성능을 높인다고 해시코드를 계산할 때 핵심 필드를 생략해서는 안된다. 속도는 빨라지겠지만 해시 품질이 나빠져 해시 테이블의 성능을 심각하게 떨어뜨릴수도 있다. 특히 어떤 필드는 특정 영역에 몰린 인스턴스들의 해시코드를 넓은 범위로 고르게 퍼뜨려주는 효과가 있을텐데 이를 생략하면 해시테이블의 속도가 선형으로 느려질 것이다.

또한 **hashCode**가 반환하는 값의 생성 규칙을 **API** 사용자에게 자세히 공표하지 말아야 한다. 그래야 클라이언트가 이 값에 의지하지 않게 되고 해시 기능에 결함을 발견했거나 더 나은 방식을 알아낸 경우 수정할 수 있다.
<br>

> ✅ **핵심 정리**
equal를 재정의할때는 hashCode도 반드시 재정의해야한다. Object의 API 문서에 기술된 일반 규약을 따라야하며, 서로 다른 인스턴스라면 되도록 해시코드도 서로 다르게 구현해야한다.
