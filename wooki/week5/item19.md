# 상속을 허용하는 클래스가 지켜야 할 제약

## 1. 상속용 클래스는 재정의할 수 있는 메서드들을 내부적으로 어떻게 이용하는지(자기사용) 문서로 남겨야 한다.

메서드 주석에 `@implSpec` 태그를 붙이면 `javadoc`이 해당 메서드의 내부 동작 방식의 설명을 생성해준다.

```java
/**
 * @implSpec 여기에 적는다.
 */
public void method() {

}
```

AbstractCollection.isEmpty()

![스크린샷 2024-05-26 17.50.07.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/fbc70dbe-7cd1-4fee-b7ab-3b08ed457a1b/d8453cdb-5cec-4565-9cf4-c2fa9308d788/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2024-05-26_17.50.07.png)

`@implSpec`에 적은 내용은 위에서 보는 것처럼 `implementation Requirements:`에서 확인할 수 있다.

javadoc을 잘 작성한다면 의미있을 것 같다.

## 2. 하위 클래스를 효율적으로 어렵지 않게 만들려면 클래스의 내부 동작 과정 중간에 끼어들 수 있는 hook을 잘 선별하여 protected 메서드 형태로 공개하자.

어떤 것을 protected로 노출해야 하는지는 실제 하위 클래스를 만들어 시험해보는 것이 최선이다.

protected 메서드 하나하나는 내부 구현이고, 그 수는 가능한 적어야 캡슐화 관점에서 좋다. 너무 적게 노출하면 상속으로 얻는 이점이 희미해질 수 있으니 트레이드 오프를 잘 따져야 한다.

구현체의 사용자는 관심이 없을지라도 성능상 문제가 될 수 있는 메서드는 protected로 열어주는 판단을 할 수도 있다.

예시) java.util.AbstractList.removeRange()

![스크린샷 2024-05-26 17.55.28.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/fbc70dbe-7cd1-4fee-b7ab-3b08ed457a1b/d57df98d-f421-4577-94c7-696575ca7a20/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2024-05-26_17.55.28.png)

removeRage()가 없다면 하위 클래스에서 clear()를 호출할 때 제곱에 비례하여 성능이 느려지거나, 부분 리스트의 메커니즘을 처음부터 다시 구현해야할 수도 있다.

## 3. 상속용 클래스의 생성자는 직간접적으로든 재정의 가능 메서드를 호출해서는 안된다.

상위 클래스의 생성자는 하위 클래스의 생성자보다 먼저 실행된다.

하위 클래스에서 재정의한 메서드가 하위 클래스의 생성자보다 먼저 호출된다.

재정의한 메서드가 하위 클래스의 생성자에서 초기화하는 값에 의존한다면 의도대로 동작하지 않는다.

예시) 슈퍼 클래스의 생성자가 재정의 가능 메서드를 호출하는 경우

```java
class Super {
		public Super() {
				overrideMe();  //생성자가 재정의 가능 메서드를 호출
		}
		
		public void overrideMe() { }
}
```

```java
final class Sub extends Super {
		//초기화 되지 않은 필드, 생성자에서 초기화 함
		private final Instant instant;

		Sub() {
				instant = Instant.now();
		}
		
		//재정의 가능 메서드, 상위 클래스의 생성자가 호출된다.
		@Override
		public void overrideMe() {
				System.out.println(instant);
		}
}
```

```java
class Main {
		public static void main(String[] args) {
				Sub sub = new Sub();
				sub.overrideMe();
		}
}
```

이 코드의 실행 결과는 아래와 같다.

```bash
null
2024-05-26T09:19:12.381262Z
```

서브 클래스의 생성자를 실행하려면 슈퍼 클래스의 생성자가 먼저 실행되는데,
이때 `overrideMe()`가 호출되어 아직 초기화되기 전인 `instant`가 null로 출력된다.

메서드 실행 순서: `Super.Super -> Sub.overrideMe -> Sub.Sub`, `Sub.overrideMe`

## 4. Cloneable과 Serializable 인터페이스 중 하나라도 구현한 클래스는 상속할 수 없게 하자.

`clone`과 `readObject` 모두 직간접적으로 재정의 가능 메서드를 호출해서 안 된다.

# 상속을 금지하는 방법

위에서 다양한 예시로 상속용으로 설계하지 않은 클래스는 상속을 금지하는 것이 가장 좋은 방법임을 알 수 있다.

## 1. 클래스를 final로 선언

final 키워드는 `상속 금지`와 `오버라이딩 금지` 역할 가능!

## 2. 모든 생성자를 private이나 package-private으로 선언, public 정적 팩터리 사용

- private 생성자

  → 해당 클래스 내부에서 상속 가능

- package-private 생성자

  → 해당 패키지 내부에서 상속 가능
