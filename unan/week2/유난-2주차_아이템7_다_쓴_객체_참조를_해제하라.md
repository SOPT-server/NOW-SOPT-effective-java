# 아이템 7 : 다 쓴 객체 참조를 해제하라.

** Java는 기본적으로 GC가 메모리 관리를 해준다. -> 방심하면 안됨.

```Java

public class Stack {

  private Object[] elements;

  private int size = 0;

  private static final int DEFAULT_INITIAL_CAPACITY = 16;

public Stack() {

elements = new Object[DEFAULT_INITIAL_CAPACITY];

}

public void push(Object e) {

  ensureCapacityO;

  elements[size++] = e;

public Object pop() {

  if (size = 0)

  throw new EmptyStackException();

  return elements[—size];
}

/**

***** 원소를 위한 공간을 적어도 하나 이상 확보한다

.

***** 배열 크기를 늘려야 할 때마다 대략 두 배씩 늘린다

.

*/

private void ensureCapacity() {
	if (elements.length = size) elements = Arrays.copyOf(elements, 2 * size + 1);
	}
}

```

코드 상에 문제는 없지만, `메모리 누수`를 갖고 있는 코드.

프로그램에서 객체들을 더 이상 사용하지 않더라도, 다 쓴 참조(obsolete reference)를 여전히 가지고 있기 때문이다.

GC 언어에서는 메모리 누수를 찾기가 까다롭다.

해법은 다 쓴 참조를 null 처리 하는 것 (참조 해제) -> 무분별한 null 처리는 좋지 않음. 예외 적인 상황에서만 null 처리.

Good Practice -> 변수를 유효 범위 밖으로 밀어내는 것.

자기 메모리를 직접 관리하는 클래스라면 프로그래머는 항시 메모리 누수에 주의해야한다.

## 캐시
캐시 또한 메모리 누수  가능성이 높음.

해법
1. 캐시외부에서 키를 참조하는 동안 만 엔트리가 살아있는 캐시가 필요한 상황 WeakHashMap 사용
2. 사용하지 않는 캐시는 청소해줘야한다. 
	1. 백그라운드 스레드 활용 (ScheduledThreadPoolExecutor)
	2. LinkedHashMap remove, EldestEntry 사용

### 리스너, 콜백
콜백을 등록하고, 해지 하지 않으면 콜백이 계속 쌓여간다. -> 콜백을 약한 참조로 저장해서 가비지 컬레거가 수거해가게 하자.**
