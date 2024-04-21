싱글턴(singleton)

- 인스턴스를 오직 하나만 생성할 수 있는 클래스
- 예로는 함수와 같은 무상태 객체나 설계상 유일해야하는 시스템 컴포넌트가 있다.
- 단점
    - 타입을 인터페이스로 정의한 다음 그 인터페이스를 구현해서 만든 싱클턴이 아니면 싱글턴 인스턴스를 가짜(mock) 구현으로 대체할 수 없으므로 테스트하기가 어려워질 수 있다.

싱글턴 만드는 방식

- public static final 필드 방식
    
    ```java
    public class Elvis {
    	public static final Elvis INSTANCE = new Elvis();
    	private Elvis() { ... }
    	
    	public void leaveTheBuilding() { ... }
    }
    ```
    
    - 생성자는 private으로 감춰두고, 유일한 인스턴스에 접근할 수 있는 수단으로 public static 멤버를 마련한다.
    - private 생성자는 public static final 필드를 초기화 할 때 한 번만 호출된다.
    - public이나 protected 생성자가 없으므로 Elvis 클래스가 초기화될 때 만들어진 인스턴스가 전체 시스템에서 하나뿐임이 보장된다.
    - 클라이언트는 리플렉션 API를 통해 private 생성자를 호출할 수 있다. 이를 막기위해 생성자를 수정해 두번째 객체 생성에 예외를 던지면 된다.
    
    → 클래스가 싱글턴임이 API에 명백하게 드러난다.
    
    → 간결하다.
    
- 정적 팩터리 방식
    
    ```java
    public class Elvis {
    	private static final Elvis INSTANCE = new ElvisO;
    	private Elvis（） { ... }
    	public static Elvis getlnstance（） { return INSTANCE; }
    	
    	public void leaveTheBuildingO { ... }
    }
    ```
    
    - Elvis.getlnstance는 항상 같은 객체의 참조를 반환하므로 제2의 Elvis 인스턴스란 결코 만들어지지 않는다.
    
    → API를 바꾸지 않고도 싱글턴이 아니게 변경할 수 있다.(호출하는 스레드별로 다른 인스턴스를 넘겨주게 할 수 있다.)
    
    → 정적 팩터리를 제네릭 싱글턴 팩터 리로 만들 수 있다.
    
    → 정적 팩터 리의 메서드 참조를 공급자（supplier）로 사용할 수 있다.
    
    ⇒ 이러한 장점이 굳이 필요하지 않다면 public 필드 방식이 좋다. 
    
- 열거 타입 방식
    - 위 두 방식으로 만든 싱글턴 클래스를 직렬화하려면 인스턴스 필드를 일시적이라고 선언하고 readResolve 메서드를 제공해야한다. 이렇게 하지 않으면 역직렬화할 때마다 새로운 인스턴스가 만들어진다.
    
    ```java
    // 싱글턴임울 보장해주는 readResolve 메서드
    private Object readResolve() {
    	// '진짜‘ Elvis를 반환하고,가짜 Elvis는 가비지 컬렉터에 맡긴다.
    	return INSTANCE;
    }
    ```
    
    ```java
    public enum Elvis {
    	INSTANCE;
    	public void leaveTheBuilding() { ... }
    }
    ```
    
    → 더 간결하고 추가 노력없이 직렬화 할 수 있다.
    
    → 아주 복잡한 직렬화 상황이나 리플렉션 공격에서도 제 2의 인스턴스가 생기는 일을 완벽히 막아준다.
    

핵심 정리

→ 대부분 상황에서는 원소가 하나뿐인 열거 타입이 싱글턴을 만드는 가장 좋은 방법이다. 단, 만들려는 싱글턴이 Enum 외의 클래스를 상속해야 한다면 이 방법은 사용할 수 없다.