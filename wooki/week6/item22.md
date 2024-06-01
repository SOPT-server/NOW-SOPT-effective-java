> 인터페이스는 `타입을 정의`하는 용도로만 사용하자.
상수 공개용 수단으로 사용하지 말자.
>

인터페이스의 본래 역할은 `구현체의 인스턴스를 참조할 수 있는 타입` 역할이다.

즉, 클래스가 어떤 인터페이스를 구현한다는 의미는 자신의 인스턴스로 무엇을 할 수 있는지를 클라이언트에게 얘기해주는 것이다.

상수 인터페이스 안티 패턴) - 잘못된 인터페이스 사용 예시

```java
public interface PhysicalConstants {
    // 아보가드로 수 (1/몰)
    static final double AVOGADROS_NUMBER   = 6.022_140_857e23;

    // 볼츠만 상수 (J/K)
    static final double BOLTZMANN_CONSTANT = 1.380_648_52e-23;

    // 전자 질량 (kg)
    static final double ELECTRON_MASS      = 9.109_383_56e-31;
}
```

클래스 내부에서 사용하는 상수는 외부 인터페이스가 아닌 내부 구현이다. 상수는 내부 구현이다!

즉, 이 내부 구현을 클래스의 API로 노출하는 행위를 하는 것이다.

네임 스페이스 없이 쓰고자 이렇게 사용하는 경우가 있지만, 인터페이스의 본래 의도대로 사용하지 않은 부적절한 행동이다.

상수를 공개할 목적이라면 `상수 유틸리티 클래스`로 사용하자.

네임 스페이스는 static import로 제거할 수도 있겠다!

```java
// 상수 유틸리티 클래스
public final class PhysicalConstants {
  private PhysicalConstants() { }  // 인스턴스화 방지

  // 아보가드로 수 (1/몰)
  public static final double AVOGADROS_NUMBER = 6.022_140_857e23;

  // 볼츠만 상수 (J/K)
  public static final double BOLTZMANN_CONST  = 1.380_648_52e-23;

  // 전자 질량 (kg)
  public static final double ELECTRON_MASS    = 9.109_383_56e-31;
}
```
