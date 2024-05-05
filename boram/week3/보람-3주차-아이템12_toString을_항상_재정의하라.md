### 배경

> `toString`을 재정의하지 않으면 `Object`의 기본 `toString` 메서드를 사용하게 된다.`Object`의 기본 `toString`은`클래스이름@해시코드`가 출력된다.하지만 보통의 개발자들이 `toString` 메서드에게 기대하는 것은 그 클래스의 `필드 값`일 것이다. 따라서 `toString`은 모든 구체클래스에서 재정의되어야 한다.
>

### **toString 생성 시 주의할 점**

- **객체가 가진 주요 정보를 모두 반환하는게 좋다.**

  주요 정보를 모두 반환해야 하는게 좋다. 만약 일부 정보만 담겨 있다면, 특정 상황에서의 혼란을 야기할 수 있다.

- **포맷을 명시하기로 했으면, 명시한 포맷에 맞는 문자열과 객체를 상호전환 할 수 있는 정적 팩터리나 생성자를 함께 제공하면 좋다.**

  값 클래스의 경우에는 더욱 포맷 명시, 명시한 포맷에 맞는 객체 생성 코드를 작성해야 한다.

  생성자로 명시한 포맷에 따라 `phoneNumber` 객체 만들기

    ```java
    private PhoneNumber(String phoneNumberString) {
        String[] split = phoneNumberString.split("-");
        this.areaCode = Short.parseShort(split[0]);
        this.prefix   = Short.parseShort(split[1]);
        this.lineNum  = Short.parseShort(split[2]);
    }
    ```

  문자열을 `phoneNumber` 객체로 변환하는 정적팩터리 메서드

    ```java
        public static PhoneNumber of(String phoneNumberString) {
            String[] split = phoneNumberString.split("-");
            PhoneNumber phoneNumber = new PhoneNumber(
                    Short.parseShort(split[0]),
                    Short.parseShort(split[1]),
                    Short.parseShort(split[2]));
            return phoneNumber;
        }
    ```

- **toString이 반환한 값에 포함된 정보를 얻어올 수 있는 API를 제공하자.**

  어차피 `toString`으로 노출한 정보들은 외부에 노출되어 있으니, 각 필드를 따로 따로 조회할 수 있는 API(ex.`Getter`)를 만들어두자는 내용이다. 만약 만들지 않는다면, 클라이언트 입장에서 문자열을 파싱해서 써야 한다.


### toString 재정의가 필요 없는 경우

- 정적 유틸리티 클래스
- 열거 타입 : 이미 완벽한 toString 제공 함.

### 핵심정리

> 모든 구체 클래스에서 Object의 toString을 재정의하자. 상위 클래스에서 이미 알맞
게 재정의한 경우는 예외다. toString을 재정의한 클래스는 사용하기도 즐겁고 그 클래스를 사용한 시스템을 디버깅하기 쉽게 해준다. toString은 해당 객체에 관한 명확하고 유용한 정보를 읽기 좋은 형태로 반환해야 한다.
>