# 아이템8 : finalizer, cleaner 사용올 피하라

Java의 두 가지 객체소멸자 -> 즉시 수행된다는 보장이 없음.
1. finalizer
	- 기본적으로 안 쓰는 것이 좋다.
2. cleaner
	- 조심히 사용

finalizer, cleaner는 C++ Destructor와 다른 개념.

상태를 영구적으로 수정하는 작업에서 객체소멸자를 사용하며 안됨. (ex. 데이터베이스 영구 락 해제)
(System.gc()를 믿지말자.)

### 왜 쓰면 안됨?
1. finalizer 동작 중 발생한 예외는 무시, 처리할 작업이 남아도 종료됨.
2. 성능 문제
3. 보안 문제

종료해야할 자원을 담고 있는 객체에서 객체 소멸자를 구현할 방법 -> **==AutoCloseable**

finalizer, cleaner어디서 쓸까?
- 클라이언트가 하지 않은 자원 회수를 늦게라도 해주는 것. FileInputStream, FileOutputStream, ThreadPoolExecutor
- Native Peer와 연결된 객체 -> Native 자원 회수 -> 언제 회수 할지 모르기 때문에, 성능 저하 

cleaner를 구현할 때는 Cleaner 객체를 갖고 있는 클래스를 직접 참조하면 안된다. (예외 발생 무시)

```Java
	public class Room implements AutoCloseable {

	private static final Cleaner cleaner = Cleaner.create();

	// 청소가 필요한 자원
	private static class State implements Runnable {

	int numJunkPiles; // 방 안의 쓰레기 수

	State(int numJunkPiles) {
		this.numJunkPiles = numJunkPiles;
	}

	// close 메서드나 cleaner가 호출한다
	@Override public void run() {
	System.out. println("방 청소");
	numJunkPiles = 0;
	}
}

	// 방의 상태 cleanable과 공유한다
	private final State state;

	// cleanable 객체 수거 대상이 되면 방을 청소한다
	private final Cleaner.Cleanable cleanable;

	public Room(int numJunkPiles) {
		state = new State(numJunkPiles);
		cleanable = cleaner.register(this, state);
	}

	@Override public void close() {
		cleanable.clean();
	}
}
```



