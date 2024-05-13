
아이템 20. 추상 클래스보다는 인터페이스를 우선하라.

자바가 제공하는 다중 구현 메커니즘
1. 인터페이스
2. 추상 클래스

둘의 차이
1. 추상클래스는 정의한 타입을 구현하는 클래스가 반듯이 추상 클래스의 하위 클래스가 되어야 한다. 자바는 단일 상속만 지원하기 때문에, 추상 클래스 방식은 새로운 타입을 정의하는데 커다란 제약이 있다.
2. 인터페이스는 선언한 메소드를 모두 정의하고, 일반 규약만 잘 지키면 어떤 클래스를 상속하든 같은 취급이 된다. -> 기존 클래스에서도 손쉽게 새로운 인터페이스를 구현할 수 있다.

인터페이스는 Mixin 정의에 안성맞춤이다.
믹스인 : 클래스가 구현할 수 있는 타입, 믹스인을 구현한 클래세 원래의 주된 타입외에도 특정 선택적 행위를 제공한다고 선언하는 효과. -> 대상 타입의 주된 기능에 선택적 기능을 혹합

Comparable은 자신을 구현한 클래스의 인스턴스들끼리는 순서를 정할 수 있다고 선언한 믹스인 인터페이스

인터페이스로는 계층구조가 없는 타입 프레임워크를 만들 수 있다.

```Java

public interface Singer {
	AudioClip sing(Song s);
}

public interface Songwriter {
	Song compose(int chartPosition);
}


```

타입을 인터페이스로 정의하면 가수클래스가 Singer, Songwriter모두를 구현해도 문제가 되지 않고, Singer, Songwriter 모두를 확장하고 새롭게 인터페이스를 정의할 수 있다.
```Java
	public interface SingerSongwriter extends Singer, Songwriter {
		AudioClip strum();
		void actSensitive();
	}
```


combinational explosion 현상을 인터페이스가 해결해줄 수 있다.
- 클래스 계층구조에는 공통 기능을 정의해놓은 타입이 없으니, 매개변수 타입만 다른 메서드들을 수도 없이 많이 가진 거대한 클래스를 낳을 수 있다.

Wrapper class와 함께 사용하면 인터페이스는 기능을 향상 시키는 안전하고 강력한 수단이 된다.

또한 인터페이스의 메서드 중 구현 방법이 명백한 것은 -> default 메소드로 제공할 수 있다.

default method 제약
- equals, hashCode와 같은 Object의 method를 디폴트로 제공하면 안된다.
- 인스턴스 필드를 가질 수없고, public이 아닌 static 멤버도 가질 수 없다.
- 만들지 않은 인터페이스에는 디폴트 메서드를 추가할 수 없다.

Template Method 패턴
- 인터페이스와 추상 골격 구현(Skeletal Implementation)을 같이 제공한다.



골격 구현 예시
```Java

static List<Integer> intArrayAsList(int[] a) {
 Objects.requireNonNull(a);

  return new AbstractList<>() {
	  @Override public Integer get(int i) {
		  return a[i]; // 오토박싱
	  }


	 @Override public Integer set(int i, Integer val) {
		 int oldVal = a[i];
		 a[i] = val; // 오토 언박싱
		 return oldVal; // 오토박식
	 }
  }
}

```
int 배열을 받아 Integer 인스턴스의 리스트 형태로 보여주는 어댑터(Adapter). int 값과 Integer 인스턴스 사이의 변환 떄문에 성능은 좋지 않다.

골격 구현 클래스
- 추상 클래스처럼 구현을 도와줌
- 추상클래스로 타입을 정의할 때 오는 제약으로부터 자유로움.

구조상 골격 구현이 어렵다면, 인터페이스를 직접 구현해야한다.
- -> 인터페이스가 직접 제공하는 디폴트 메소드의 이점을 누릴 수 있다.

골격 구현 클래스를 우회적으로 이용하는 방법
- 인터페이스를 구현한 클래스에서 해당 골격 구현을 확장한 private 내부 클래스를 정의하고, 각 메서드 호출을 내부 클래스의 인스턴스에 전달 하는 것.

아이템 18에서 다룬 래퍼 클래스와 비슷한 이 방식을 `simulated multiple interitance` 라고 함.

골격 구현 작성
1. 인터페이스를 잘 살펴 다른 메서드들의 구현에 사용되는 기반 메서드 들을 선정.
2. 기반 메서드들을 사용해 직접 구현할 수 있는 메서드를 모두 디폴트 메서드로 제공하면 안 된다는 사실 기억

ex) Map.Entry

getKey, getValue -> 기반 메소드
```Java
package com.example.demo.dd;  
  
import java.util.Map;  
import java.util.Objects;  
  
public abstract class AbstractMapEntry<K, V> implements Map.Entry<K, V> {  
  
    // 변경 가능한 엔트리는 이 메서드를 반드시 재정의해야한다.  
    @Override  
    public V setValue(V value) {  
        throw new UnsupportedOperationException();  
    }  
    // Map.Entry.equals의 일반 규약을 구현한다.  
    @Override  
    public boolean equals(Object o) {  
        if (o == this) {  
            return true;  
        }        if (!(o instanceof Map.Entry)) {  
            return false;  
        }        Map.Entry<?, ?> e = (Map.Entry<?, ?>) o;  
        return Objects.equals(getKey(), e.getKey()) && Objects.equals(getValue(), e.getValue());  
    }  
    // Map.Entry.hashCode의 일반 규약을 구현한다.  
    @Override  
    public int hashCode() {  
        return Objects.hashCode(getKey()) ^ Objects.hashCode(getValue());  
    }  
    @Override  
    public String toString() {  
        return getKey() + "=" + getValue();  
    }  
}
```

| Map.Entry 인터페이스나 그 하위 인터페이스로는 이 골격 구현을 제공할 수 없다. 디폴트 메소드는 equals, hashCode, toString 같은 Object method를 재정의 할수 없기 때문이다.

단순 구현(Simple Implementation)은 골격구현의 작은 변종으로 Abstract.Map.SimpleEntry가 좋은 예시다. 


### 핵심정리
일반적으로 다중 구현용 타입은 인터페이스가 적당하다.
골격 구현은 가능한 인터페이스의 디폴트 메소드로 제공하고 그 인터페이스를 구현한 모든 곳에서 활용하는 것이 좋다.
가능한 이라고 한이유는 인터페이스에 걸려 있는 구현 제약 때문에 골격 구현을 추상 클래스로 제공하는 경우가 더욱 흔함.
