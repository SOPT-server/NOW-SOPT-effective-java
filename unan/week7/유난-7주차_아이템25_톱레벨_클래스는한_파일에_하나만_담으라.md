# 아이템 25. 톱레벨 클래스는 한 파일에 하나만 담으라.

소스 파일 하나에 Top-Level class를 여러 개 선언해도 문제는 없다. -> But, 득이 없다

```Java
// Utensil.java에 정의된 2개 클래스

class Utensil {
	static final String NAME = "pan";
}

class Dessert {
	static final String NAME = "pie";
}
```

이와 같이 정의되면 때에 따라 어떤 소스파일을 컴파일 하냐에 따라서 동작이 달라질 수 있다.

따라서 분리하자.
