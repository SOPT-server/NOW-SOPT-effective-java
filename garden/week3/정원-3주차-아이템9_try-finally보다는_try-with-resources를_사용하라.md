try-finally

- 자바 라이브러리에는 InputStream, OutputStream, java.sql.Connection 등과 같은 입출력 클래스는 close 메서드를 호출해 직접 닫아줘야 하는 자원이 많다.

→ 전통적으로 자원이 제대로 닫힘을 보장하는 수단으로 try- finally가 사용되었다.

문제점

```java
static String firstLineOfFile(String path) throws IOException {

	BufferedReader br = new BufferedReader(new InputStreamReader(TryFinallyTest.class.getResourceAsStream(path)));
	try {
		return br.readLine();
	}finally {
		br.close();
	}
}
```

```java
static void copy(String src, String dst) throws IOException {
    InputStream in = new FileInputStream(src);
    try {
        OutputStream out = new FileOutputStream(dst);
        try {
            byte[] buf = new byte[BUFFER_SIZE];
            int n;
            while ((n = in.read(buf)) >= 0)
                out.write(buf, 0, n);
        } finally {
            out.close(); 
        }
    } finally {
        in.close();
    }
}
```

- try-finally 방식에서 try 블럭과 finally 블럭에서 둘다 예외가 발생하면 try 블럭에서 발생한 예외는 무시되고 finally 블럭에서 발생한 예외만 출력된다.
- close해야 할 자원이 둘 이상이라면 try-finally 구문이 복잡해진다.

→ 이러한 문제들은 try-with-resources로 해결되었다.

try-with-resources

```java
static String firstLineOfFile(St ring path) throws lOException {
	try (BufferedReader br = new BufferedReader(
		new FileReader(path))) {
	return br. readLineO;
	}
}
```

```java
static void copy(String src, String dst) throws lOException {
	try (Inputstream in = new FilelnputStream(src);
		Outputstream out = new FileOutputStream(dst)》 {
	byte[] buf = new byte[BUFFER_SIZE];
	int n;
	while ((n = in.read(buf)) >= 0)
		out.write(buf, 0, n);
	}
}
```

- try-with-resources 방식은 try에 자원 객체를 전달하면 try 코드 블록이 종료되면 자동으로 자원을 close하는 방식이다.
- 해당 자원이 AutoCloseable 인터페이스를 구현해야 한다.(단순히 void를 반환하는 close 메서드 하나만 덩그러니 정의한 인터페이스다.)
- 자바 라이브러리와 서드파티 라이브러리들의 수많은 클래스와 인터페이스가 이미 AutoCloseable을 구현하거나 확장했다.

장점

1. 다중 예외가 발생한 경우 무시되지 않고 기록된다.
    
    → 예를 들어 readLine과 close 호출 양쪽에서 예외가 발생하면 close에서 발생한 예외는 숨겨지고 readLine에서 발생한 예외가 기록된다. close에서 발생한 예외는 suppressed라는 꼬리표를 달고 출력된다. 또한, Throwable에 추가된 getSuppressed 메서드를 이용하면 프로그램 코드에서 가져올 수 있다.
    
2. finally문을 사용하지 않기 때문에 복잡해지지 않아 가독성이 좋아진다.
    
    → catch절을 사용할 수 있다.
    
    ```java
    static String firstLineOfFile(St ring path, String defaultVal) {
    	try (BufferedReader br = new BufferedReader(
    			new FileReader(path))) {
    		return br.readLineO;
    	} catch (lOException e) {
    		return defaultVal;
    	}
    }
    ```
    

핵심 정리

→ 꼭 회수해야 하는 자원을 다룰 때는 try-finally 말고, try-with-resources를 사용하자. 예외는 없다. 코드는 더 짧고 분명해지고, 만들어지는 예외 정보도 훨씬 유용하다. try-finally로 작성하면 실용적이지 못할 만큼 코드가 지저분해지는 경우라도, try-with-resources로는 정확하고 쉽게 자원을 회수할 수 있다.