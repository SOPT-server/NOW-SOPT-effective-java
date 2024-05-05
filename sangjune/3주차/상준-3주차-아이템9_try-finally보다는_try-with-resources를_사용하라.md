# try-finally보다는 try-with-resources를 사용하라

## 직접 닫아줘야 하는 자원

자바에서는 일반적으로 자원을 열고 닫는 기준으로 자동으로 해제되지 않는 시스템 자원을 사용한다. 이러한 자원에는 파일 시스템 객체, 네트워크 연결, 데이터베이스 연결 또는 사용자 입력 스트림 등이 포함된다. 이러한 자원들은 사용 후 명시적으로 해제하거나 닫아야 메모리 누수나 기타 자원 낭비를 방지할 수 있다.

## try-finally

전통적으로 try-finally를 자원을 닫는 수단으로 사용해왔다. try - catch 문에서 마지막에 실행되는 finally 블록에 자원의 해제를 사용하는 방식이다.

### 사용 방식

대부분 다음과 같은 형태로 구현됐을 것이다.

```java
    BufferedReader br = new BufferedReader(new FileReader(path));
    try {
        return br. readl_ine();
    } finally {
        br.close();
    }
```

### 문제점

위와 같이 구현할 경우 문제점이 몇개 있다.
첫째로 자원을 둘 이상 사용해야 할 경우 try-finally 문이 중첨으로 사용되며 코드를 어지럽힐 수 있다. 아래가 그 예시인데 코드의 가독성을 해치고 플로우를 따라가는 것을 어렵게 할 수 있다.

```java
    InputStream in = new FilelnputStream(src);
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
```

또 기기에 물리적인 문제가 생긴다면 firstLineOfFile 메소드 안의 readLine 메서드가 예외를 던지고 같은 close 메소드도 실패하게 될 것이다. 이럴 경우 close에서 일어난 예외가 첫 번째 예외를 가리게 되어 디버깅에 어려움을 겪을 수 있다.

ex) finally는 무조건 실행되므로 try에서 예외가 일어난 후 catch에서 에러가 일어나서 가려질 수 있다

## try-with-resources

하지만 위의 문제점을 해결하는 문법이 자바 7에서 등장했는데 바로 try-with-resources 문법이다. try-with-resources 문법은 당연히 catch 절도 사용할 수 있다. 그러므로 자원의 해제가 필요한 코드에 대해서는 **꼭 try-finally 대신 꼭 try-with-resources를 사용하자**

### 사용 방식

try-with-resources 문법은 다음과 같이 사용할 수 있다.

```java
    try (BufferedReader br = new BufferedReader(new FileReader(path))) {
            return br. readLineO;
    }
```

```java
    try (Inputstream in = new FilelnputStream(src);
        Outputstream out = new FileOutputStream(dst)
    ) {
        byte[] buf = new byte[BUFFER_SIZE];
        int n;
        while ((n = in.read(buf)) >= 0)
        out.write(buf, 0, n);
    }
```

python을 사용한 적 있는 사람이라면 python의 with-open과 비슷하다고 생각할 수 있을 것 같다. 다만 python은 try-catch를 사용하려면 내부에 따로 구현해야한다는 점이 차이점으로 꼽을 수 있다.
