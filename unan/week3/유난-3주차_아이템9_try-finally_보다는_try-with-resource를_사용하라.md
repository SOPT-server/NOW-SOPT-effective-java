# Item 09. try-finally 보다는 try-with-resource를 사용하라.

회수해야하는 자원을 다룰 때는 try-finally 대신 try-with-resource를 사용하자.

Java에서 직접 close method를 호출해 직접 닫아줘야 하는 자원이 있다. ex) InputStream, OutputStream, java.sql.Connection

```Java
static void copy(String src, String dst) throws IOException {
	InputStream in = new FileInputStream (sc) ;
	try {
		OutputStream out = new FileOutputStream(dst) ;
		try {
			bytel[] buf = new byte (BUFFER_SIZE];
			int n;
			while ((n = in. read (buf)) >= 0)
				out. write(buf, 0, n);
		} finally {
				out. close);
				} 
			}
		finally {
			in. close();
		}
	}
```
try-finally 문을 제대로 사용한 앞의 두 코드 예제에조차 미묘한 결점이 있다.

예외는 try 블록과 finally 블록 모두에서 발생할 수 있는데, 예컨대 기기에 물리적인 문제가 생긴다면 firstLineOfFile 메서드 안의 readLine 메서드가 예를 던지고, 같은 이유로 close 메서드도 실패할 것이다.
이런 상황이라면 두 번째 예외가 첫 번째 예외를 완전히 집어삼켜 버린다.
그러면 스택 추적 내역에 첫 번째 예외에 관한 정보는 남지 않게 되어 디버깅이 어려워진다.(일반적으로 문제를 진단하려면 처음 발생한 예외를 보고 싶을 것이다). 물론 두 번째 예외 대신 첫 번째 예외를 기록하도록 코드를 수정할 수는 있지만, 코드가 너무 지저분해져서 실제로 그렇게까지 하는 경우는 거의
없다.

try-with-resource 적용
- try에 자원 객체를 전달
	-  이 때 자원 객체는 AutoCloseable을 구현하고 있어야한다.
```Java
static void copy (String src, String dst) throws IOException {
	try (InputStream in = new FileInputStream(src);
		OutputStream out = new FileOutputStream(dst) {
		byte[] buf = new byte [BUFFER_SIZE];
		int n;

		while ((n = in. read (buf)) >= 0)
			out.write(buf, 0, n);
	}
}
```

catch를 사용하여, try를 중첩하지 않고 다수의 예외를 처리할 수 있다.
```Java

static String firstLineOfFile(String path, String defaultVal) {
  try (BufferedReader br = new BufferedReader(
    new FileReader(path))) {
      return br.readLine();
  } catch (IOException e) {
      return defaultVal;
  }
}

```
