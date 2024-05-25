> 꼭 필요하지 않다면 equals를 재정의하지 말자. (다수의 경우에 Object의 equals가 내가 원하는 비교를 정확하게 수행한다.)
그래도 해야한다면 모든 필드에 대해, 다섯 가지 규약을 지켜서 재정의하자.
그치만 사람보다는 IDE나 구글의 AutoValue가 낫다.
>

# equals()를 재정의하지 않아도 되는 상황

## 1. 각 인스턴스가 본질적으로 고유할 때

값이 아닌 동작하는 개체를 표현하는 클래스

ex. Thread (Object의 equals 메서드가 이러한 클래스에 딱 맞게 구현됨)

## 2. 인스턴스의 논리적 동치성(logical equality; 동등성)을 검사할 일이 없을 때

반대로 논리적 동치성을 검사하는 경우 → 문자열

## 3. 상위 클래스에서 재정의한 equals를 하위 클래스에서 그대로 쓸 수 있을 때

ex. Set 구현체는 AbstractSet이 구현한 equals를 상속받아서 사용, List 구현체들은 AbstractList가 구현한 equals를 상속받아서 사용, Map 구현체들을 AbstractMap이 구현한 equals를 상속받아서 사용

## 4. 클래스가 private이나 package-private이고 equals 메서드를 호출할 일이 없을 때

위험을 철저하게 막고 싶다면 (equals가 실수로라도 호출되는 걸 막고 싶다면) 아래처럼 구현해두자.

```java
@Override
public boolean equals(Object o) {
		throw new AssertionError();  //호출 금지!
}
```

# equals()를 재정의 해야 하는 상황

아래 조건을 모두 만족할 때

- 객체 식별성(object identity; 두 객체가 물리적으로 같은지)이 아니라 `논리적 동치성을 확인`해야할 때
- `상위 클래스`의 equals가 논리적 동치성을 비교하도록 `재정의되지 않았을 때`

→ 주로 값 클래스(Integer나 String처럼 값을 표현하는 클래스) 등이 해당

프로그래머는 값 클래의 객체가 같은지가 아니라 `값이 같은지`가 필요하다. 즉, 동일성이 아닌 `동등성`을 원한다!

물론 인스턴스 통제 클래스라면 값이 같은 인스턴스가 둘 이상 만들어지지 않음을 보장하기 때문에 equals를 재정의하지 않아도 된다. 인스턴스 통제 클래스는 하나의 인스턴스만 존재하기 때문에 동등성 비교를 할 필요가 없다. Object의 equals는 ==로 되어 있지만 위 이유때문에 그냥 써도 됨.

# equals를 재정의하기 위한 규약

Object에 정의되어 있는 equals이다.

```java
public boolean equals(Object obj) {
		//메모리 주소 비교로 구현되어 있어서 동등성 비교하려면 오버라이딩 해야함
		return (this == obj);
}
```

Object 명세에 적힌 규약이다.

equals 메서드는 동치관계(equivalence relation)을 구현하며, 다음을 만족한다.

## 반사성(reflexivity)

<aside>
💡 null이 아닌 모든 참조 값 x에 대해, `x.equals(x)`는 `true`

</aside>

즉, 객체가 자기 자신과 같아야 한다는 의미이다. 일부러 어기는 경우가 아니면 깨기 힘들다.

## 대칭성(symmetry)

<aside>
💡 null이 아닌 모든 참조 값 x, y에 대해, `x.equals(y)`가 `true`면 → `y.equals(x)`도 true

</aside>

상속이나 참조를 통해 깰 수 있다.

## 추이성(transitivity)

<aside>
💡 null이 아닌 모든 참조  값 x, y, z에 대해 `x.equals(y)`가 `true`고 `y.equals(z)`도 `true`면 → `x.equals(z)`도 `true`

</aside>

상속이나 참조를 통해 깰 수 있다.

## 일관성(consistency)

<aside>
💡 null이 아닌 모든 참조 값 x, y에 대해 x.equals(y)를 반복해서 호출하면 항상 같은 결과가 나와야 함

</aside>

언제든 변할 수 있는 가변 필드로 깰 수 있다.

reflection이나 상속으로 깰 수도 있다. (깨려고 마음먹고 깨는게 아니면..)

## null이 아님

<aside>
💡 null이 아닌 모든 참조 값 x에 대해 `x.equals(null)`은 `false`

</aside>

`if (arg == null) return true;`로 깰 수 있다.

`대칭성`, `추이성`, `일관성`은 `기본 클래스를 확장`했을 때 깨지기 쉽다.

타입 비교를 엄격히 하여 `instanceof` 조건을 꼭 검사하고, 특별한 이유가 아니면 상위 클래스와 하위 클래스 간의 비교를 허용하지 말자!
