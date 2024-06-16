# 아이템 18. 상속 보다는 컴포지션을 사용하라
상속은 코드를 재사용하는 강력한 수단, 항상 최선은 아니다.

메서드 호출과 달리 상속은 캡슐화를 깨트림.
- 상위 클래스의 구현 여부에 따라 하위 클래스의 동작에 문제가 생길 수 있다.


잘못된 상속 예시
```Java

public class InstrumentedHashSet<E> extends HashSet<E> {
	private int addCount = 0;

	public InstrumentedHashSet() {
	
	}
	@Override public boolean add(E e) {
		addCount++;
		return super.add(e);		
	}

	@Override public boolean addAll(Collection<? extends E> c) {
		addCount += c.size();
		return super.addAll(c);
	}

	public int getAddCount() {
		return addCount;
	}
}
```

해당 method는 6을 반환함.
```Java
InstrumentedHashSet<String> s = new InstrumentedHashSet();
s.addAll(List.of("틱", "탁탁", "펑"));
```

why? HashSet의 addAll이 add method를 구현함.
InstrumentedHashSet의 addAll은 addCount에 3을 더한 후, HashSet의 addAll 구현을 호출.
HashSet의 addAll은 각 원소를 add 메서드를 호출해 추가해줌. 따라서 중복으로 덧셈이 이루어짐.


이 경우에 addAll method를 재정의하지 않으면 문제를 고칠 수 있다. -> 자신의 다른 부분을 사욯나는 self-use 여부는 해당 클래스의 내부 구현방식에 해당 
-> 과연 Java가 새롭게 릴리즈되도 유지 될까?


## 해결방법
기존 클래스를 확장하는 대신 새로운 클래스를 만들고, private field로 기존 클래스 인스턴스를 참조하게 하자.
기존 클래스가 새로운 클래스의 구성요소로 쓰인다고 해서 이러한 설계를 Composition이라고 함.

새 클래스의 인스턴스 메소드들은 기존 클래스의 대응하는 메소드를 호출해 그 결과를 반환함. (Forwarding) 

새 클래스의 메소드들을 forwarding method라고 한다. 


상속 대신 컴포지션 사용
```Java
public class InstrumentedSet<E> extends ForwardingSet<E> {  
    private int addCount = 0;  
    public InstrumentedSet (Set<E> s) {  
        super(s);  
    }        @Override public boolean add (E e){  
            addCount++;  
            return super.add(e);  
        }        @Override public boolean addAll (Collection<? extends E> c){  
            addCount += c.size();  
            return super.addAll(c);  
        }  
        public int getAddCount() {  
            return addCount;  
        }    }
```
```
```

재사용할 수 있는 Forwarding class
```Java
public class ForwardingSet<E> implements Set<E> {  
  
    private final Set<E> s;  
    public ForwardingSet(Set<E> s) {  
        this.s = s;  
    }  
    public void clear() {  
        s.clear();  
    }    
    public boolean contains(Object o) {  
        return s.contains(o);  
    }    
    public boolean isEmpty() {  
        return s.isEmpty();  
    }    
    public int size() {  
        return s.size();  
    }    
    public Iterator<E> iterator() {  
        return s.iterator();  
    }    
    
    public boolean add(E e) {  
        return s.add(e);  
    }    
    
    public boolean remove(Object o) {  
        return s.remove(o);  
    }    
    
    public boolean containsAll(Collection<?> c) {  
        return s.containsAll(c);  
    }    
    
    public boolean addAll(Collection<? extends E> c) {  
        return s.addAll(c);  
    }  
    
    public boolean removeAll(Collection<?> c) {  
        return s.removeAll(c);  
    }    
    
    public boolean retainAll(Collection<?> c) {  
        return s.retainAll(c);  
    }    
    
    public Object[] toArray() {  
        return s.toArray();  
    }    
    
    public <T> T[] toArray(T[] a) {  
        return s.toArray(a);  
    }    
    
    public boolean equals(Object o) {  
        return s.equals(o);  
    }    
    
    public int hashCode() {  
        return s.hashCode();  
    }    
    
    public String toString() {  
        return s.toString();  
    }  
}
```

`InstrumentedSet`을 이용하면 대상 Set 인스턴스를 특정 조건하에서만 임시로 계측할 수 있다.

```Java
static void walk(Set<Dog> dogs) {  
    InstrumentedSet<Dog> iDogs = new InstrumentedSet<>(dogs);  
    ...
```

`InstrumentedSet` 과 같은 class를 Wrapper Class라고 하며, Set에 계측 기능을 덧씌운다는 뜻에서 Decorator Pattern이라고 한다.

Composition, Forwarding은 넓은 의미로 Delegation이라고 한다.
단 래퍼 객체가 내부 객체에 자기 자신의 참조를 넘기는 경우만 위임에 해당한다.


Wrapper class 사용시 주의
- callback framework 어울리지 않음.
- callback framework에서는 자기 자신의 참조를 다른 객체에 넘겨서 콜백 시에 사용하도록 한다.
- 내부 객체는 자신을 감싸고 있는 래퍼의 존재를 모르니 대산 자신의 참조를 넘기고, 콜백 때는 래퍼가 아닌 내부 객체를 호출한다. (Self 문제)


Forwarding method 성능, Wrapper class 메모리 사용량 영향은 크지 않다고 밝혀짐. Forwarding method를 작성하는 것이 지루할 수 있지만, interface를 잘 정의하면 쉽게 구현할 수있다.


### 정리

상속은 강력하지만 캡슐화를 해친다는 문제가 있다.
상속은 반드시 하위 클래스가 상위 클래스의 진짜 하위 타입인 상황에서만 쓰자.
클래스 B가 클래스 A와 순수한 is-a 관계일 때만 클래스 A를 상속해야 한다.
is-a 일때도 하위 클래스의 패키지가 상위 클래스와 다르고, 상위 클래스가 확장을 고려해 설계하지 않았다면 문제가 될 수 있다.

<자바 플랫폼 라이브러리의 잘못된 상속 예시>
- Stack extends Vector
- Properties extends Hashtable

Composition 대신 상속을 하면 내부 구현을 불필요하게 노출하게 됨.

따라서 상속의 취약점을 피하려면 Composition, Forwarding을 사용하자.

