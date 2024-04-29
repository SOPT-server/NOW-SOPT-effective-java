# 메모리 누수 예시1. 다 쓴 참조(obsolete reference)

- stack을 간단히 구현한 예시

  pop하는 경우 element[--size]를 해서 반환할 뿐, 꺼내진 객체에 대해서 GC가 일어나지 않는다.

  → 즉, 다 쓴 객체에 대해 메모리를 해제하지 않아서 메모리 누수가 발생한다.

    ```java
    public class Stack {
        private Object[] elements;
        private int size = 0;
        private static final int DEFAULT_INITIAL_CAPACITY = 16;
    
        public Stack() {
            elements = new Object[DEFAULT_INITIAL_CAPACITY];
        }
    
        public void push(Object e) {
            ensureCapacity();
            elements[size++] = e;
        }
    
    		// AS - IS
        public Object pop() {
            if (size == 0)
                throw new EmptyStackException();
            return elements[--size];
        }
    
        private void ensureCapacity() {
            if (elements.length == size)
                elements = Arrays.copyOf(elements, 2 * size + 1);
        }
    }
    ```


## 해결.  null로 메모리를 해제한다.

<aside>
💡 다 쓰면 null로 해제하여 GC가 일어나도록 하자!

</aside>

```java
// TO - BE
public Object pop() {
    if (size == 0)
        throw new EmptyStackException();
    Object result = elements[--size];
    **elements[size] = null;** // 다 쓴 참조 해제
    return result;
}
```

다 쓴 참조에 대해 null로 해제해줌으로써 GC가 일어나도록 한다.

# 메모리 누수 예시2. 캐시

일반적으로 캐시를 HashMap으로 만든다. Map 안에 Key와 Value가 put되면 사용여부와 관계없이 삭제되지 않는다. Key에 해당하는 객체가 null이 되어도 해당 entry는 그대로 map에 남아있다. 이러한 이유때문에 메모리 누수가 발생한다.

### 해결. WeakHashMap을 사용한다.

`WeakHashMap`을 사용하여 Key에 해당하는 객체가 null이 된다면 더이상 사용되지 않는다고 판단하고 GC할 수 있다.

# 메모리 누수 예시3. 이벤트 기반 리스너/콜백 이용

클라이언트가 콜백을 등록만 하고 명확히 해지하지 않는다면, 계속해서 콜백이 쌓이게 된다.

### 해결. Weak Reference를 사용한다.

콜백을 `Weak Reference`로 저장하여 GC가 즉시 수거하도록 한다.