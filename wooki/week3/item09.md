> 꼭 회수해야 하는 리소스는 `try-with-resources`를 사용하자!
코드도 짧아지고, 분명해지고, 만들어지는 예외 정보도 유용하다!
>

# 문제. `try-finally` 사용

### 하나의 리소스만 다룰 때

```java
static String firstLineOfFile(String path) throws IOException {
    BufferedReader br = new BufferedReader(new FileReader(path));
    try {
        return br.readLine();    //exception 발생
    } finally {
        br.close();    //exception 발생 - stack trace에는 얘만 보이게 된다.
    }
}
```

잘못된 코드는 아니지만, 한개가 아니라 여러개의 리소스를 다뤄야 한다면 지저분하다.

<aside>
💡 단점1. 마지막에 발생한 예외 이외의 예외는 삼켜져버린다.

</aside>

엄밀히 따지면 문제가 있는데 try절에서 예외를 던진다면, finally절이 실행은 되겠지만 예외가 던진다. 이런 경우 첫번째 예외(먼저 발생한 예외)는 두번째 예외(나중에 발생한 예외)를 삼켜버린다.

### 여러개의 리소스를 다룰 때

- 정말 잘못된 예시 - `memory leak` 발생

    <aside>
    💡 단점2. 자원을 해제하는 코드를 꼭 실행한다는 보장이 없다. 즉, memory leak이 발생할 우려가 있다.

    </aside>

    ```java
    static void copy(String src, String dst) throws IOException {
        InputStream in = new FileInputStream(src);
        OutputStream out = new FileOutputStream(dst);
        try {
    		    byte[] buf = new byte[BUFFER_SIZE];
    		    int n;
    		    while ((n = in.read(buf)) >= 0)
                    out.write(buf, 0, n);
        } finally {
            in.close();    //exception 발생 시,
            out.close();    //이 자원은 닫히지 않는다 -> leak 염려가 있다
        }
    }
    ```

- 여러개의 리소스를 try-finally로 다루는 예시

  위 예시보다 낫기는 하다. out을 해제할 때 이슈가 생겨도, 2번째 finally절의 in을 해제하려고는 하기 때문!

  하지만 권장되는 방법은 아니다. 역시나 예외가 삼켜질 수 있기 때문

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


# 해결. `try-with-resources` 사용

## try-with-resources 구조

<aside>
💡 try절에 사용할 자원에 대한 정의를 한 것, 개발자가 명시적으로 close()를 수행하지 않는다.

</aside>

Java7에서 도입되었다. 이 구조를 사용하려면 해당 자원이 `AutoCloseable` 인터페이스를 구현해야한다. 물론 `catch절과 함께 사용도 가능`하다.

아래 예시의 BufferedReader는 Reader의 구현체인데, Reader는 Closeable를 구현하고 있다.

![스크린샷 2024-05-05 22.17.30.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/fbc70dbe-7cd1-4fee-b7ab-3b08ed457a1b/ce81d0ec-0d0d-44d2-a6f5-400778dc65b7/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2024-05-05_22.17.30.png)

리소스를 다루는 많은 클래스에서 AutoCloseable을 구현하고 있다!

### 하나의 리소스만 다룰 때

- AS - IS

    ```java
    static String firstLineOfFile(String path) throws IOException {
        **try (BufferedReader br = new BufferedReader(new FileReader(path)))** {
            return br.readLine();
        }
    }
    ```


### 여러개의 리소스를 다룰 때

여러개의 리소스를 다룰 때 더 발한다.

- AS - IS

    ```java
    static void copy(String src, String dst) throws IOException {
        **try (InputStream in = new FileInputStream(src);
             OutputStream out = new FileOutputStream(dst))** {
            byte[] buf = new byte[BUFFER_SIZE];
            int n;
            while ((n = in.read(buf)) >= 0)
                out.write(buf, 0, n);
        }
    }
    ```

    <aside>
    💡 먼저 발생된 예외는 그냥 버려지지 않고, stack trace 내역에 `suppressed`라는 키워드와 함께 출력된다.

    </aside>

  Thorwable의 getSuppressed 메서드를 이용하여 코드에서 가져올 수도 있다.
