# equals는 일반 규약을 지켜 재정의하라

## 기존 equals(Object)의 작동 방식

Object 클래스의 equals 메서드는 모든 자바 클래스에서 가장 기본적인 메서드 중 하나로, 객체 간 동등성을 비교하는 데 사용된다. 기본적으로 Object의 equals는 두 객체의 참조가 같은지 확인한다. 즉, 두 객체가 메모리 상에서 같은 위치를 가리키는지를 비교하는데 보통의 경우 이런 기능은 유저가 기대하는 기능이 아니다. 그렇기 때문에 보통 논리적 동등성을 확인하는 메소드로 오버라이딩을 한다.

## equals를 재정의하지 않는 상황

하지만 equals를 꼭 재정의하지 않아도 되는데 그런 경우를 꼽아보자면 다음과 같다.

1. 각 인스턴스가 본질적으로 고유하다.
   - 값을 표현하는 게 아닌 동작하는 개체 등
2. 인스턴스의 '논리적 동치성'을 검사할 일이 없다
   - 설계자가 논리적 동치성을 검사하는 메소드가 필요하지 않다고 판단했을 경우
3. 상위 클래스에서 재정의한 equals가 하위 클래스에도 딱 들어맞는다.
   - 대부분의 Set 구현체는 AbstractSet이 구현한 equals를 상속받아 쓴다.
4. 클래스가 private이거나 package-private이고 equals 메소드를 호출할 일이 없다.

## equals 재정의

### equals의 재정의가 필요한 상황

객체 식별성(두 객체가 물리적으로 같은가?)이 아니라 논리적 동치성을 확인해야하는데 상위 클래스의 equals가 논리적 동치성을 비교하도록 재정의하지 않았을 때 -> 주로 값 클래스(값을 표현하는 클래스)가 여기에 해당됨

### equals의 재정의 규약

#### 반사성

객체는 자기 자신과 비교했을 때 항상 true를 반환해야 함

#### 대칭성

두 객체가 서로에 대해 equals의 결과가 동일해야 함 즉, x.equals(y)가 true라면 y.equals(x)도 true여야 함

#### 추이성

세 객체에 대해 x.equals(y)와 y.equals(z)가 true라면, x.equals(z)도 true여야 함

#### 일관성

두 객체의 상태가 변경되지 않는 한, equals 메서드는 호출될 때마다 동일한 결과를 반환해야 함

#### null-아님

어떤 객체에 대해서도 null과 비교한 equals 결과는 false여야 함

### equals 메서드 구현 방법

equals 메서드를 구현할 때는 주로 필드별로 값을 비교한다. instanceof 검사를 통해 비교 대상이 올바른 타입인지 확인한 후, 타입 캐스팅을 통해 각 필드가 동일한지를 검사한다.

```java
@Override
public boolean equals(Object obj) {
    if (this == obj) return true;
    if (!(obj instanceof Person)) return false;
    Person person = (Person) obj;
    return age == person.age &&
           Objects.equals(name, person.name);
}
```

> 여기에 추가하고 싶은 내용(상세 내용 및 성능 관련...)이 있는데 아직 못했습니다 ㅠ 업데이트 할게요

### AutoValue 프레임워크

쉽게 equals를 구현하도록 도와주는 프레임워크가 있다. AutoValue 프레임워크는 구글에서 제공하는 라이브러리로, equals뿐 아니라 이후에 나올 hashCode, toString 메서드를 자동으로 생성해준다. 이를 사용하면, 값 클래스를 더 쉽고 오류 없이 작성할 수 있으며, equals와 관련된 규약을 자동으로 지키게 된다.
