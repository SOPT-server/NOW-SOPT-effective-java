## 아이템13. clone 재정의는 주의해서 진행하라

Cloneable은 복제해도 되는 클래스임을 명시하는 용도의 믹스인 인터페이스이다. clone은 원본 객체의 필드값과 동일하게 새로운 객체를 생성해주는 메소드이다. Cloneable을 구현하는 것만으로는 외부 객체에서 clone 메서드를 호출할 수 없다. 

<br> 

그렇다면 Cloneable은 무슨 일을 할까? 

이 인터페이스는 Object의 protected 메서드인 clone의 동작 방식을 결정한다. Cloneable을 구현한 클래스의 인스턴스에서 clone을 호출하면 그 객체의 필드들을 하나하나 복사한 객체를 반환하며, 그렇지 않은 클래스의 인스턴스에서 호출하면 CloneNotSupportedException 예외를 발생시킨다. 또한 생성자를 호출하지 않고도 객체를 생성할 수 있게 된다.

clone 메서드의 규약은 다음과 같다.

1. 어떤 객체 x에 대해 다음 식은 참이다.
    
    ```java
    x.clone() ≠ x
    x.clone().getClass() == x.getClass()
    ```
    
2. 다음 식도 일반적으로 참이지만, 필수는 아니다
    
    ```java
    x.clone().equals(x)
    ```
    
3. 관례상 이 메서드가 반환하는 객체는 super.clone을 호출하여 얻어야한다.
    
    ```java
    x.clone().getClass() == x.getClass()
    ```
    

이 클래스의 하위 클래스에서 super.clone을 호출한다면 잘못된 클래스의 객체가 만들어져 하위 클래스의 clone 메서드가 제대로 동작하지 않을 것이다. 하지만 final 클래스의 clone 메서드가 super.clone을 호출하지 않는다면 Cloneable을 구현할 필요가 없다. 

```java
@Override public PhoneNumber clone() {
    try {
        return (PhoneNumber) super.clone();
    } catch (CloneNotSupportedException e){
        throw new AssertionError();
    }
}
```

위의 코드에서 이 메서드가 동작하게 하려면 PhoneNumber 클래스 선언에 Cloneable을 구현해주어야한다. 

또한 super.clone()을 try-catch로 감싼이유는 CloneNotSupportedException 예외를 잡기 위해서이다. 

<br> 

이 클래스를 복제할 수 있도록 만들어보자.

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

    public Object pop() {
        if (size == 0)
            throw new EmptyStackException();
        Object result = elements[--size];
        elements[size] = null; // Eliminate obsolete reference
        return result;
    }

    private void ensureCapacity() {
        if (elements.length == size)
            elements = Arrays.copyOf(elements, 2 * size + 1);
    }
}
```

clone을 하게 되면 반환된 Stack인스턴스의 size필드는 올바른 값을 가지지만 elements 필드는 원본 Stack 인스턴스와 같은 배열을 참조하게되어 원본이나 복제본 중 하나를 수정하면 다른 것도 수정이 될 것이다. 따라서 프로그램이 이상해지거나, NPE가 발생할 것이다. 

<br>

Stack의 clonea메서드는 제대로 동작하려면 스택 내부 정보를 복사해야하는데, 가장 쉬운 방법은 elements 배열의 clone을 재귀적으로 호출해주는 것이다. 

```java
@Override public Stack clone() {
    try {
        Stack result = (Stack) super.clone();
        result.elements = elements.clone();
        return result;
    } catch (CloneNotSupportedException e) {
        throw new AssertionError();
    }
}
```

배열의 clone은 런타임타입과 컴파일타임 타입 모두가 원본 배열과 똑같은 배열을 반환한다. 따라서 배열을 복사할때는 clone사용이 권장되지만 final일 경우에는 사용할 수 없다. 

clone을 재귀적으로 호출하는 것만으로는 충분하지 않은 경우도 있는데 해시테이블용 clone이 속한다.

해시테이블 내부는 버킷들의 배열이고, 각 버킷은 키-값 쌍을 담는 연결리스트의 첫번째 엔트리를 참조한다. clone을하게 되면 원본과 복제본 모두 원치 않게 동작할 가능성이 있다. 따라서 각 버킷을 구성하는 연결리스트를 복제해야한다. 이를 위해서는 재귀적 clone 방식과 반복자를 써서 순회하는 방식이 있다. 

<br>

요약하자면 Cloneable을 구현한 모든 클래스는 clone을 재정의해야한다. 이때 접근제한자는 public으로 반환타입은 클래스 자신으로 변경한다. super.clone을 호출한 후 필요한 필드를 전부 적절히 수정해야한다. 

객체의 내부 깊은 구조에 숨어있는 가변 객체를 복사하고, 복제본이 가진 객체 참조 모두가 복사된 객체들을 가리키게 해야한다. 이런 내부 복사는 재귀적으로 호출하지만 더 나는 객체 복사 방식으로 복사 생성자과 복사 팩터리 방식을 사용할 수 있다. 여기서 복사생성자란 단순히 자신과 같은 클래스의 인스턴스를 인수로 받는 생성자를 말한다. 

<br>

복사생성자와 복사팩터리는 Clonable/clone 방식보다 더 좋은 점이 많다.

1. 위험한 객체 생성 메커지늠을 사용하지 않는다.
2. final과도 충돌하지 않는다.
3. 불필요한 예외도 던지지않고, 형변환도 필요하지 않다.
4. 해당 클래스가 구현한 인터페이스 타입의 인스턴스를 인수로 받을 수 있다. → 변환생성자/변환팩터리

> **핵심정리**
Cloneable을 구현하는 클래스는 clone()을 재정의해야 한다. clone()은 깊은 구조를 따져보며 모두 재귀적으로 복사를 해줘야 한다. 기본 원칙은 ‘복제 기능은 생성자와 팩터리를 이용하는 것이 최고’라는 것이다. 단 배열은 clone 메서드 방식이 가장 깔끔하다.
