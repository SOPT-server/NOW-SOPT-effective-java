### 자바가 제공하는 객체 소멸자

- `finalizer`
    - 최상위 Object 클래스에 포함된 메서드이기 때문에  finalize 메서드 재정의(Overriding) 가능
    - finalize메서드를 재정의하면 해당 객체가 Garbege Collection 대상이 되었을 때 finalize메서드가 호출된다. 단, 즉시 호출을 보장받을 수는 없다.
    - 즉시 호출이 보장되지 않기 때문에 한정적 자원 해제시 해제하는 작업을 finalizer로 구현하면 안된다. 이런 경우 try-with-resource 또는 try-finally를 이용해 구현해야 한다.
    - finalize 메서드는 **자바9부터 deprecated** 되었고 이에 대한 대안으로 자바에서는 **Cleaner를 지원**하게 되었다.
- `cleaner`

  cleaner는 API로 제공했던 finalizer처럼 재정의하는 것과 달리 구성을 통해 cleaner를 사용해야 한다. 하지만, cleaner 역시 예측할 수 없고, 느리고, 일반적으로 불필요하다.

</br>

  ## finalizer와 cleaner의 사용을 지양하라.

  ### 첫 번째, finalizer 와 cleaner 는 즉시 실행된다는 보장이 없다.

  > finalizer 나 cleaner 는 즉시 실행된다는 보장이 없어 제때 실행되어야하는 작업은 절대 할 수 없다.
  
- 언제 실행될지 알 수가 없다. 어떤 객체가 필요 없어진 시점에 `finlizer` 또는 `cleaner`가 바로 실행되지 않을 수도 있다. 그 시간이 얼마나 걸릴지는 아무도 모른다. 따라서 타이밍이 중요한 작업을 절대로 `finalizer` 또는 `cleaner`로 해서는 안된다.
- 예를 들어, 파일 리소스를 반납하는 작업을 `finalizer` 또는 `cleaner`로 처리한다면 실제로 그 파일 리소스가 언제 처리될지 알 수 없고 자원 반납이 안되서 더 이상 새로운 파일을 열 수 없는 상황이 발생할 가능성이 있다.


</br>


  ### 두 번째, finalizer 처리는 인스턴스의 자원 회수가 제멋대로 지연될 수 있다.

  > 인스턴스 회수가 지연될 뿐만 아니라 제대로 실행되지 않을수도 있다.
  >
 - `finalizer` 스레드는 다른 애플리케이션 스레드보다 우선 순위가 낮아서 실행될 기회를 제대로 얻지 못할수도 있다. (다른 작업이 바쁘면 뒤로 미뤄진다, 언제 호출될지 모른다는 의미이다.) 따라서 `finlizer` 안에 어떤 작업이 있고, 그 작업을 쓰레드가 처리 못해서 대기하고 있다면 해당 인스턴스는 `GC`가 되지 않고 계속해서 쌓이다가 결국 `OOM(Out Of Memory Exception)`이 발생할 수 있다.
 - `cleaner` 는 자신을 수행할 스레드를 제어할 수는 있지만, 역시나 가비지 컬렉터에 의존하므로 즉시 사용된다는 보장은 없다.

</br>

  ### 세 번째, 수행 여부조차 보장되지 않는다.

  > 상태를 영구적으로 수정하는 작업에서는 절대 finalizer 나 cleaner 에 의존해서는 안된다.
  >
  - 자바 언어 명세는 **finalizer** 나 **clenaer** 의 수행 여부를 보장하지 않는다. 접근할 수 없는 일부 객체에 딸린 종료작업을 수행하지 못한채 프로그램이 종료될 수도 있다.
  - 만약 데이터 베이스 같은 자원의 락을 이것들로 반환하는 작업을 한다면 전체 분산 시스템이 멈춰 버릴 수도 있다. 따라서 **finalizer** 나 **clenaer**로 저장소 상태를 변경하는 일을 하지 말아야 한다.

    ```java
    public class SampleRunner {
        public static void main(String[] args) {
            final SampleRunner sampleRunner = new SampleRunner();
            sampleRunner.run();
            System.gc(); // 실행이 안될수도 있다!
        }
    }
    ```

    - `System.gc` 나 `System.runFinalization` 메서드에 속지 말자 이들은 실행 가능성을 높여줄 순 있으나, 보장하지는 않는다.
    - `System.runFinalizationOnExit` 와 `Runtime.runFinalizationOnExit` 는 실행을 보장해주려 만들었지만 심각한 결함으로 수십년간 `deprecated` 상태이다.
    -

</br>

  ### 네 번째, 예외가 무시된다.

  > finalizer 동작 중 발생한 예외는 무시되며, 처리할 작업이 남았더라도 그 순간 종료된다.
  >

  잡지못한 예외때문에 객체가 훼손될 수 있고, 다른 스레드가 이 훼손된 객체에 접근하게 될 수 있다. **finalizer** 에서 예외가 발생할 경우 경고조차 출력하지 않는다. 그나마 **cleaner** 를 사용하는 라이브러리는 자신의 스레드를 통제하기 때문에 이러한 문제가 발생하지 않는다.

</br>

  ### 다섯 번째, 심각한 성능 문제도 동반한다.
 - `finalizer` 가 가비지 컬렉터의 효율을 떨어뜨리기 때문에 `try-with-resource` 와 비교했을 때 굉장히 느리다. `cleaner` 역시 마찬가지이다.

</br>


  ### 여섯 번째, finalizer는 심각한 보안문제를 일으킬 수 있다.

  - `finalizer` 를 사용한 클래스는 `finalizer` 공격에 노출되어 심각한 보안 문제를 일으킬 수 있다.
  - A, B 클래스가 있다고 가정했을 때, B클래스가 A클래스를 상속받는다. 그리고 `finalize` 를 오버라이딩한 B 클래스의 인스턴스를 생성하는 도중에 예외가 발생하거나, 직렬화 과정에서 예외가 발생하면, 원래 죽었어야 할 객체의 `finalizer`가 실행될 수 있다.

      그럼 `finalize` 메서드 안에서 이 인스턴스가 가진 `static` 필드에 접근할 수 있어서 해당 인스턴스의 레퍼런스를 기록할 수 있고 GC에 의해 수거되지 못하게 할 수도 있다.

      원래는 생성자가 예외를 발생시켜 존재하지 않았어야 하는 인스턴스인데 `finalizer` 때문에 살아남아 있는 것이다.

  - 이 문제에 대한 해결방법은 `final` 클래스들은 하위 클래스를 만들 수 없으니 공격으로부터 안전하다. 따라서 아무일도 하지 않는 `finalizer` 메서드를 만들고 `final` 로 선언하자.

</br>

### 정리

> `cleaner`과 `finalizer`는 안전망 역할이나 중요하지 않은 네이티브 자원 회수용으로만 사용하자. 이런 경우라도 불확실성과 성능 저하에 주의해야 한다.
>