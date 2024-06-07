# 중첩 클래스

다른 클래스 안에 정의된 클래스

중첩 클래스는 자신을 감싼 바깥 클래스에서만 쓰여야 하고, 그 외의 쓰임새가 있다면 톱레벨 클래스로 만들어야 한다.

- 중첩 클래스의 종류
    - 정적 멤버 클래스 (static nested class)
    - 멤버 클래스 (inner class)
    - 익명 클래스 (anonymous class)
    - 지역 클래스 (local class)

## 정적 멤버 클래스 (static nested class)

```java
class Outer {

		private static int outerClassValue = 3;
		private int outerInstanceValue = 2;
		
		static class Inner {
				private int innerInstanceValue = 1;
				
				public void print() {
						// 자신의 멤버에는 당연히 접근 O
						System.out.println(innerInstanceValue);
						
						// 바깥 클래스의 인스턴스 멤버 - 접근 X
						//System.out.println(outerInstanceValue);
						
						// 바깥 클래스의 클래스 멤버 - 접근 O
						// Outer.outClassValue를 outClassValue로 사용해도 됨
						System.out.println(outerClassValue);
				}
		}
}
```

중첩 클래스도 바깥 클래스와 같은 클래스에 있기 때문에, 중첩 클래스가 바깥 클래스의 `private` 접근 제어자에 접근할 수 있다는 점만 특별하다.

→ 그냥 클래스 2개를 따로 만든 것과 비슷하다.

<aside>
💡 Inner 클래스의 인스턴스는 Outer 클래스의 인스턴스와 관계 없이 생성 가능!

</aside>

```java
public static void main(String[] args) {
		Outer outer = new Outer();
		
		// 생성: new 바깥클래스.중첩클래스()
		// -> new Outer.Inner();
		// 접근: 바깥클래스.중첩클래스
		// -> Outer.Inner
		Outer.Inner inner = new Outer.Inner();
}
```

## 멤버 클래스 (inner class)

```java
class Outer {

		private static int outerClassValue = 3;
		private int outerInstanceValue = 2;
		
		class Member {
				private int memberInstanceValue = 1;
				
				public void print() {
						// 자신의 멤버에는 당연히 접근 O
						System.out.println(memberInstanceValue);
						
						// 외부 클래스의 인스턴스 멤버 - 접근 O
						System.out.println(outerInstanceValue);
						
						// 외부 클래스의 클래스 멤버 - 접근 O
						System.out.println(Outer.outerClassValue);
		}
}
```

Inner 클래스는 Outer 클래스의 인스턴스에 소속됨!

<aside>
💡 → Outer 클래스의 인스턴스가 없다면 Member 클래스의 인스턴스를 생성할 수 없음!

</aside>

<aside>
💡 Member 클래스에서 Outer 클래스에 접근할 일이 없다면 무조건 static을 붙여서 정적 멤버 클래스로 만들자!

</aside>

- static을 생략하고 멤버 클래스로 만든다면 바깥 인스턴스로의 숨은 외부 참조를 가지게 되고, 이 참조를 저장하려면 시간과 공간이 소비된다.
- GC가 바깥 클래스의 인스턴스를 수거하지 못하는 memory leak이 발생할 수도 있다

```java
public static void main(String[] args) {
		Outer outer = new Outer();
		
		// 생성: new 바깥클래스의 인스턴스 참조.내부 클래스()
		// -> new outer.Member();
		Outer.Member member = outer.new Member();
}
```

## 익명 클래스 (anonymous class)

클래스의 이름을 생략하고, 클래스의 선언과 생성을 한 번에 처리할 수 있다.

```java
// 지역 클래스일 때
class LocalPrinter implements Printer {
		// body
}

Printer printer = new LoaclPrinter();

// 익명 클래스일 때
Printer printer = new Printer() {
		// body
}
```

## 지역 클래스 (local class)

지역 클래스는 멤버 클래스(inner class)의 특별한 종류의 하나이다.

→ 바깥 클래스의 인스턴스 멤버에 접근 가능!

지역 변수처럼 코드 블럭 안에서 정의됨

```java
class Outer {
		
		public void doSomething() {
				int localValue = 0;
				
				class Local { }
				
				Local local = new Local();
		}
}
``` 
