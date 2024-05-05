## 아이템9. try-finally보다는 try-with-resources를 사용하라

<br>

자바 라이브러리에는 close 메서드를 호출하여 직접 닫아줘야하는 자원이 많다. 그 예시로는 InputStream, OutputStream, java.sql.Connection 등이 있다. 자원을 닫아줘야하는 것은 클라이언트가 놓치는 경우가 많으므로 성능 이슈로 이어질 수도 있다. 

close의 안전망으로 finalizer를 사용할 수도 있지만 우리가 Item8에서 알아봤듯이 믿을만하지 못하다. 따라서 전통적으로는 try-finally를 사용했다. 

```java
static String firstLineOfFile(String path) throws IOException {
    BufferedReader br = new BufferedReader(new FileReader(path));
    try {
        return br.readline();
    } finally {
        br.close();
    }
}
```

이렇게 작성하게 되면 나쁘지 않지만 자원을 더 사용한다면 어떨까?

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
    }
}
```

자원이 두 개 이상이 되면 try-finally 방식이 너무 지저분하다.  또한 예외는 try블록과 finally블록 모두에서 발생할 수 있는데 이렇게 되면 두 번째 예외가 첫 번째 예외를 완전히 집어 삼켜버리게 된다. 물론 두번째 예외 대신 첫 번째 예외를 기록하도록 코드를 수정할 수 있지만 코드가 지저분해지고 결국 똑같이 하나는 집어삼켜지는 문제가 발생할 것이다.

<br>

따라서 try-with-resources 를 이용하여 해결할 수 있다. 

이 구조를 사용하려면 해당 자원이 AutoCloseable 인터페이스를 구현해야한다. AutoClosable은 단순히 void를 반환하는 close 메서드 하나만 덩그러니 정의한 인터페이스이다. 

try-with-resources를 사용하여 코드를 작성한 예시는 아래와 같다.

```java
static String firstLineOfFile(String path) throws IOException {
    try (BufferedReader br = new BufferedReader(
                    new FileReader(path))) {
        return br.readline();
    }
}
```

```java
static void copy(String src, String dst) throws IOException {
    try (InputStream in = new FileInputStream(src);
        OutputStream out = new FileOutputStream(dst)) {
            byte[] buf = new byte[BUFFER_SIZE];
            int n;
            while ((n = in.read(buf)) >= 0)
                out.write(buf, 0, n);
    }
}
```

이렇게 작성하면 읽기 수월할 뿐만 아니라 문제를 진단하기에도 훨씬 좋다. 

<br>

위의 코드에서 보면 첫번째 메서드에서 예외가 발생하면 close에서 발생한 예외는 숨겨지고 readline에서 발생한 예외가 기록된다. 이렇게 보여줄 예외 하나만 보존되고 여러개의 예외가 숨겨질 수 있는데 이러한 예외들은 스택 추적 내역에 숨겨져있다. 

<br>

보통의 try-finally처럼 try-with-resources에서도 catch 절을 사용할 수 있는데 try문을 중첩하지 않고도 다수의 예외를 처리할 수 있다.

```java
static String firstLineOfFile(String path, String defaultVal) throws IOException {
    try (BufferedReader br = new BufferedReader(
                    new FileReader(path))) {
        return br.readline();
    } catch (IOException e) {
        return defaultVal;
    }
}
```

<br>

> ✅ **핵심 정리**
꼭 회수해야하는 자원을 다룰 때는 try-finally말고 try-with-resources를 사용하자. 코드도 더 짧고 분명해지고 더 유용한 예외정보를 얻을 수 있다. 또한 try-with-resources 로는 정확하고 더 쉽게 자원을 회수할 수 있다.
