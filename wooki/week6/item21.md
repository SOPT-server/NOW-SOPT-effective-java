자바8 이전까지는 기존 구현체를 깨뜨리지 않고 인터페이스에 메서드를 추가할 수 없었지만,
자바8부터 도입된 `디폴트 메서드`를 통해 기존 구현체를 깨뜨리지 않고 인터페이스에 메서드를 추가할 수 있게 되었음

인터페이스에 디폴트 메서드를 추가하는 것은 해당 인터페이스의 모든 구현체에 기능을 강제로 삽입하는 것과 다름 없음

컬렉션의 `removeIf` 메서드는 인터페이스 관점에서는 아무 문제가 없지만,
구현체인 `SynchronizedCollection` 입장에서는 굉장히 위험하다.

`SynchronizedCollection`은 모든 오퍼레이션을 동기화하여 멀티 스레드 환경에서 한 번에 한 스레드만 해당 오퍼레이션을 실행하도록 하지만, `removeIf` 메서드에는 동기화와 관련된 코드가 존재하지 않는다. 즉, 멀티 스레드 환경에서 안전하지 않다는 뜻이다. ConcurrentModificationException이 발생할 수 있다.

```java
public interface Collection<E> extends Iterable<E> {
   default boolean removeIf(Predicate<? super E> filter) {
      Objects.requireNonNull(filter);
      boolean removed = false;
      final Iterator<E> each = iterator();
      while (each.hasNext()) {
         if (filter.test(each.next())) {
            each.remove();
            removed = true;
         }
      }
      return removed;
   }
   // 생략
}    
```

이처럼 인터페이스에 추가된 디폴트 메서드가 구현체에 해가 되거나 예상치 못한 동작이라면, 오버라이딩해야 한다.

근데 이건 우리가 관리하는 라이브러리에서나 가능한 일이다.

써드파티 인터페이스에서 어떤 디폴트 메서드가 추가되었는지 일일이 확인하기는 어렵기 때문이다.

어쨌든 인터페이스에 디폴트 메서드를 추가하는 것은 심사숙고하자.
