### 배경

> 자바 라이브러리에는 close 메서드로 호출해 직접 닫아줘야하는 자원이 존재한다. 하지만 자원 닫기는 클라이언트가 놓치기 쉬워서 성능 문제로 이어질 수 있다. `finalizer`로 해결이 가능은 하지만 믿을만하지 못하다(item8)
>

그렇기 때문에 전통적으로 `try finally`문을 사용해서 close처리를 해줬다. 하지만 `try finally` 방식은 자원이 둘 이상일 경우 코드가 지저분해지고 실수를 저지를 가능성이 커진다.

```jsx
public static String inputString() throws IOException {
    BufferedReader br = new BufferedReader(new InputStreamReader(System.in));
    try {
        return br.readLine();
    } finally {
        br.close();
    }
}
```

inputString 메서드의 try 블록을 실행하던 도중 기기에 물리적 문제가 생긴다면 readLine이 정상적으로 실행되지 못하고 예외를 던지게 되고, 같은 이유로 finally 블록의 close 메서드도 예외를 던지게 된다.

### 해결책

`try finally`가 아닌 `try with resources`를 사용하자

```jsx
public static String inputString() throws IOException {
    try (BufferedReader br = new BufferedReader(new InputStream(System.in))) {
        return br.readLine();
    }
}
```

이때 해당 자원이 AutoCloseable인터페이스를 구현해야한다.

### `try with resources`의 장점

1. 버전이 짧고 읽기 수월하다
2. 문제를 진단하기 좋다
3. catch절을 사용할 수 있다

### 핵심정리

> 꼭 회수해야 하는 자원을 다룰 때는 try-finally 말고, try-with-resources를 사용하자. 예외는 없다. 코드는 더 짧고 분명해지고, 만들어지는 예외 정보도 훨씬 유용하다. try-finally로 작성하면 실용적이지 못할 만큼 코드가 지저분해지는 경우라도, try-with-resources로는 정확하고 쉽게 자원을 회수할 수 있다.
>