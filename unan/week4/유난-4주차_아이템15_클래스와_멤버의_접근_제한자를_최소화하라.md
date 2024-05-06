# 아이템 15. 클래스와 멤버의 접근 제한자를 최소화하라.


## 정보 은닉의 장점
1. 시스템 개발 속도를 높인다.
2. 시스템 관리 비용을 낮춘다.
3. 정보 은닉화 자체가 성능을 높이지는 않지만, 성능 최적화에 도움을 준다.
4. 큰 시스템을 제작하는 난이도를 낮춰준다.

자바의 정보은닉 장치
1. 클래스
2. 인터페이스
3. 접근 제어자

모든 클래스와 멤버의 접근성을 가능한 한 좁혀야한다. 

클래스, 인터페이스 -> package private, public
- 패키지 외부에서 쓸 일이 없으면 package-private


멤버(필드, 메서드, 중첩 클래스, 중첩 인터페이스)에 부여할 수 있는 접근 수준
1. private : 멤버를 선언한 Top level class에만 접근
2. package-private 멤버가 소속된 패키지안의 모든 클래스에서 접근할 수 있다. 접근제한자를 명시하지 않았을 때 적용되는 패키지 접근 수준
3. protected: package-private의 접근 범위를 포함하며, 이 멤벌르 선언한 클래스의 하위 클래스에서도 접근할 수 있다.
4. public : 모든 곳에서 접근할 수 있다.

###  public class
public 클래스에서는 멤버 접근수준을 package-private -> protected로 바꾸면 대상범위가 엄청나게 넓어진다. public class의 protected 멤버는 공개 API이므로 영원히 지원되어야 한다.
내부 동작 방식을API 문서에 적어 사용자에게 공개해야할 수도 있음.

public class의 instance field는 되도록 Public이 아니어야한다.
- 필드가 가변 객체를 참조하거나, final이 아닌 인스턴스 필드를 public으로 선언하면 그 필드에 담을 수 있는 값을 제한할 힘을 잃게 된다.
- 그 필드와 관련 된 모든 것은 불변식을 보장할 수 없게 된다는 뜻이다.

보안 허점이 있는 코드 예시
```Java

public static final Thing[] VALUES = {
...
};
```
길이가 0이 아닌 배열은 모두 변경 가능하니 주의. 따라서 클래스에서 public static final 배열 필드를 두거나 이 필드를 반환하는 접근자 메서드를 제공하면 안된다.


=> 리팩토링

앞 코드의 public 배열을 private으로 만들고, public 불변 리스트 추가
```Java

private static final Thing[] PRIVATE_VALUES = {
	...
};

public static final List<Thing> VALUES = Collections.unmodifidableList(Arrays.asList(PRIVATE_VALUES));

```

배열을 private으로 만들고, 복사본을 반환하는 method 추가(방어적 복사)
```Java
private static final Thing[] PRIVATE_VALUES = {
	...
};

public static final Thing[] values() {
	return PRIVATE_VALUES.clone();
}

```

자바 9에서는 모듈 시스템이 도입됨. -> 접근 수준 추가


### 핵심정리

public class는 상수용 publics static final 필드 외에는 어떠한 public 필드도 가져서는 안된다.
public static final 필드가 참조하는 객체가 불변인지 확인.
