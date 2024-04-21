![](https://velog.velcdn.com/images/bo-ram-bo-ram/post/99944e65-a07a-4214-a1a1-df2f0ff5d832/image.png)

### 배경 : 싱글턴

싱글턴은 인스턴스를 오직 하나만 생성할 수 있는 클래스다. 이런 싱글턴 클래스는 호출 시 매번 인스턴스를 생성하지 않고 미리 생성해둔 인스턴스를 반환하기 때문에 DB의 Connection Pool에서도 사용하여 관리하는 것이 효율적이다

책에서는 이런 싱글톤을 만드는 방법으로 3가지를 소개하고 있다. 각각에 대해 알아보자.
<br>
- 싱글톤을 만드는 방법
    1. **public static final 필드 방식의 싱글턴**

        ```java
        public class Elvis {
        	**public static final Elvis INSTANCE = new Elvis();**
        	private Elvis() { ... }
        	public void leaveTheBuilding() { ... }
        }
        ```

        - 장점
            1. 해당 클래스가 싱글턴임이 API에 드러남
            2. 간결함

            <br>
    2. **정적 팩터리 방식의 싱글턴**

        ```java
        public class Elvis {
        	private static final Elvis INSTANCE = new ElvisO;
        	private Elvis（） { ... }
        	**public static Elvis getlnstance（） { return INSTANCE; }**
        	public void leaveTheBuildingO { ... }
        	
        	// 싱글턴임울 보장해주는 readResolve 메서드
        	private Object readResolve() {
        		// '진짜‘ Elvis를 반환하고, 가짜 Elvis는 가비지 컬렉터에 맡긴다.
        		return INSTANCE;
        	}
        }
        ```

        - 장점
            1. api를 바꾸지 않고도 싱글턴이 아니게 변경 가능
            2. 정적 팩터리를 제네릭 싱글턴 패턴으로 변경 가능
            3. 정적 팩터리의 메서드 참조를 공급자로 사용 가능
               <br>
    3. **열거 타입 방식의 싱글턴 << 제일 추천**

        ```jsx
        public enum Elvis {
        	INSTANCE;
        	public void leaveTheBuilding() { ... }
        }
        ```

        - 장점 : 간결함 및 추가 노력 없이 직렬화 가능
        - 단점 : 만들려는 싱글턴이 Enum 외의 클래스를 상속해야 한다면 사용 불가
          <br>