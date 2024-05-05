# equals를 재정의하려거든 hashCode도 재정의하라

equals를 재정의했다면 hashCode 메소드도 반드시 재정의가 필요하다. 그렇지 않으면 해당 클래스의 인스턴스를 Hash를 사용하는 컬렉션의 원소로 사용할 때 문제가 일어나게 된다.

## 기존 hashCode(Object)의 작동 방식

Object 클래스에서 제공하는 hashCode 메소드는 객체의 메모리 주소를 기반으로 해시코드를 생성한다. 이 기본 구현은 equals 메서드가 재정의되면 더 이상 적절하지 않게된다. 같은 내용의 객체가 다른 해시코드 값을 반환할 수 있기 때문이다. 그렇기 때문에 equals 메소드를 재정의했다면 hashCode도 재정의가 필요하다.

### hashCode의 일반 규약

1. 애플리케이션 실행 중 객체가 수정되지 않는 한, hashCode 메소드는 몇 번을 호출하더라도 동일한 값을 반환해야한다.
2. equals 메서드로 비교했을 때 true를 반환한다면, 두 객체의 hashCode 메소드도 같은 정수 값을 반환해야 한다.
3. equals로 비교했을 때 false를 반환한다 하더라도, 두 객체의 hashCode가 서로 다른 값을 반환할 필요는 없다. 단 다른 객체에 대해서는 다른 값을 반환해야 해시 테이블의 성능이 좋아진다.

## hashCode 작성 방법

### Objects의 hash

Java 7 이상에서는 Objects.hash(Object...) 메서드를 사용하여 해시코드를 쉽게 생성할 수 있다. 이 메서드는 내부적으로 배열을 생성하고, 입력된 인수들에 대해 hashCode를 호출하여 결과를 합친다. 하지만 그 과정에 있어서 성능이 좋지 않을 수 있다고 한다.

### 좋은 hashCode를 작성하는 요령

1. 초기 해시값 설정
   해시코드 계산을 시작하기 전에 초기 해시값(int 타입)을 설정한다.

2. 핵심 필드에 대한 해시코드 계산
   객체의 동등성을 결정하는 데 각 핵심 필드에 대해 해시코드를 계산한다.

각 필드의 해시코드를 계산한 후, 이전 결과에 다음과 같이 계산을 추가한다. 여기서 c는 현재 필드의 해시코드이고, 31은 홀수 소수인데 소수를 사용하는 이유는 해시 충돌 확률을 줄이기 위함이다.

```java
result = 31 * result + c;
```

3. 결과 반환
   모든 필드에 대한 계산이 끝나면 최종적으로 계산된 result 값을 반환합니다.

```java
@Override
public int hashCode() {
    int result;
    result = 31 * result + (myBoolean ? 1 : 0);
    result = 31 * result + myInt;
    result = 31 * result + (int)(myLong ^ (myLong >>> 32));
    result = 31 * result + Float.floatToIntBits(myFloat);
    result = 31 * result + Double.hashCode(myDouble);
    result = 31 * result + Objects.hashCode(myObject);
    result = 31 * result + Arrays.hashCode(myArray);
    return result;
}
```

### 성능

좋은 hashCode 구현은 해시 테이블의 성능을 최적화하는 데 중요하다. 효율적인 해시 함수는 해시 충돌을 최소화하면서도 계산이 빠르게 수행되어야 한다.

성능을 개선하기 위한 방법으로 지연 초기화와 같은 기법을 사용하여 해시코드를 한 번 계산한 후 저장해두고 필요할 때마다 반환하는 방식을 사용할 수도 있다.

꼭 기억해둬야 할 점은 모든 중요한 필드를 해시 계산에 포함시켜야한다는 것이다. 당장의 성능을 쫓기 위해 일부 필드를 제외하게 된다면 해시 성능은 향상될 수 있지만 해시 테이블의 효율성이 떨어지게 된다.
