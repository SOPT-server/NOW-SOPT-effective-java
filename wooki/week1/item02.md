> 생성자나 정적 팩터리가 처리해야 할 매개변수가 많다면, 빌더 패턴을 선택하자!
매개변수 중 다수가 필수가 아니거나 같은 타입이라면 더욱 더!
빌더는 점층적 생성자(생성자 체이닝)보다 간결하고, 자바빈즈보다 훨씬 안전하다!
>

<aside>
💡 생성자와 정적 팩터리에는 제약이 있다. 선택적 매개변수가 많을 때 적절히 대응하기 어렵다!

</aside>

# 문제. 선택적 매개변수가 많아 대응하기 어렵다

아래 예시처럼 `NutritionFacts` 클래스에서 `servingSize`와 `servings`는 필수이지만, 나머지 필드들은 선택일 때가 있다.

필수로 받아야하는 필드가 있다면 인스턴스 생성 시 생성자를 통해서 강제하는 것이 좋긴 하지만, `코드가 굉장히 지저분해진다.`

```java
public class NutritionFacts {

    private final int servingSize;    //필수

    private final int servings;    //필수

    private final int calories;    //선택

    private final int fat;    //선택

    private final int sodium;    //선택

    private final int carbohydrate;    //선택

    public NutritionFacts(int servingSize, int servings) {
        this.servingSize = servingSize;
        this.servings = servings;
    }

    public NutritionFacts(int servingSize, int servings, int calories) {
        this.servingSize = servingSize;
        this.servings = servings;
        this.calories = calories;
    }

    public NutritionFacts(int servingSize, int servings, int calories, int fat) {
        this.servingSize = servingSize;
        this.servings = servings;
        this.calories = calories;
        this.fat = fat;
    }

    public NutritionFacts(int servingSize, int servings, int calories, int fat, int sodium) {
        this.servingSize = servingSize;
        this.servings = servings;
        this.calories = calories;
        this.fat = fat;
        this.sodium = sodium;
    }

    public NutritionFacts(int servingSize, int servings, int calories, int fat, int sodium, int carbohydrate) {
        this.servingSize = servingSize;
        this.servings = servings;
        this.calories = calories;
        this.fat = fat;
        this.sodium = sodium;
        this.carbohydrate = carbohydrate;
    }
}
```

## 해결 1. 점층적 생성자 패턴(telescoping constructor pattern, 메서드 체이닝) 사용

<aside>
💡 this 생성자를 통해 중복되는 부분을 최소화하여 코드의 복잡성을 줄인다!

</aside>

```java
public NutritionFacts(int servingSize, int servings) {
    this(servings, servings, 0);
}

public NutritionFacts(int servingSize, int servings, int calories) {
    this(servings, servings, calories, 0);
}

public NutritionFacts(int servingSize, int servings, int calories, int fat) {
    this(servings, servings, calories, fat, 0);
}

public NutritionFacts(int servingSize, int servings, int calories, int fat, int sodium) {
    this(servings, servings, calories, fat, sodium, 0);
}

public NutritionFacts(int servingSize, int servings, int calories, int fat, int sodium, int carbohydrate) {
    this.servingSize = servingSize;
    this.servings = servings;
    this.calories = calories;
    this.fat = fat;
    this.sodium = sodium;
    this.carbohydrate = carbohydrate;
}
```

→ 코드의 복잡성은 줄어들었다. 원하는 매개변수를 포함한 생성자 중 가장 짧은 것을 골라 호출하면 된다.

<aside>
💡 하지만 아래처럼 인스턴스 생성 시, 모든 파라미터가 int이기 때문에 `어떤 파라미터를 줘야하는지 알기 힘들다.` 또한 `매개변수의 시퀀스를 잘못 넘겨주어 런타임에 엉뚱한 동작을 할 수도 있다.`

</aside>

IntelliJ의 Command + P를 통해 도움을 받을 수 있긴 하지만..

```java
public static void main(String[] args) {
		NutritionFacts cocaCola 
				= new NutritionFacts(?, ?, ?);  // 어떤 필드를 줘야하지?
}
```

## 해결 2. JavaBeans 패턴 사용

### JavaBeans Specification

JavaBeans는 Java의 표준 spec 중 하나이다.

1. `기본 생성자`를 가져야 함
2. `필드`로 객체의 상태를 저장, private로 선언하여 캡슐화
3. 각 필드에 대해 접근자 메서드인 `getter`를 제공해야 함 (boolean일 경우 isXXX)
4. 각 필드에 대해 설정자 메서드인 `setter`를 제공해야 함

```java
public class NutritionFacts {

    private int servingSize = -1;    //필수, 기본값 없음

    private int servings = -1;    //필수, 기본값 없음

    private int calories = 0;

    private int fat = 0;

    private int sodium = 0;

    private int carbohydrate = 0;
		
		// Setter
		public void setServingSize(int servingSize) {
				this.servingSize = servingSize;
		}
		...
		
		// Getter		
		...
}
```

```java
public static void main(String[] args) {
		NutritionFacts cocaCola = new NutritionFacts();
		
		cocaCola.setServingSize(240);
		cocaCola.setCalories(100);
		...
}
```

`기본 생성자`로 인스턴스 생성, `setter`로 필요한 필드 값을 채워주면 된다.

<aside>
💡 인스턴스 하나를 만드려면 `여러 개의 setter`를 호출해야한다.

</aside>

<aside>
💡 하지만, 필수로 채워져야 할 필드값이 채워지지 않을 수 있다. 
→ 일관성(consistency)가 깨질 수 있다, 클래스를 불변(immutable)으로 만들 수도 없다.

</aside>

## 해결 3. Builder 패턴 사용

점층적 생성자 패턴의 안전성 + 자바 빈즈 패턴의 가독성

```java
public class NutritionFacts {
		private final int servingSize;
    private final int servings;
    private final int calories;
    private final int fat;
    private final int sodium;
    private final int carbohydrate;

    public static class Builder {
        // 필수 매개변수
        private final int servingSize;
        private final int servings;

        // 선택 매개변수 - 기본값으로 초기화
        private int calories = 0;
        private int fat = 0;
        private int sodium = 0;
        private int carbohydrate = 0;

        public Builder(int servingSize, int servings) {
            this.servingSize = servingSize;
            this.servings = servings;
        }

				// setter들의 반환 타입이 Builder!
        public Builder calories(int val) {
            calories = val;
            return this;
        }

        public Builder fat(int val) {
            fat = val;
            return this;
        }

        public Builder sodium(int val) {
            sodium = val;
            return this;
        }

        public Builder carbohydrate(int val) {
            carbohydrate = val;
            return this;
        }

				// build() 메서드를 통해 클래스의 객체 타입(NutritionFacts)으로..!
        public NutritionFacts build() {
            return new NutritionFacts(this);
        }
    }

    private NutritionFacts(Builder builder) {
        servingSize = builder.servingSize;
        servings = builder.servings;
        calories = builder.calories;
        fat = builder.fat;
        sodium = builder.sodium;
        carbohydrate = builder.carbohydrate;
    }
}
```

<aside>
💡 빌더의 setter들은 Builder를 반환하기 때문에 `메서드 체이닝(method chaining)` 가능!

</aside>

```java
public static void main(String[] args) {
		// 필요한 객체를 직접 만들지 않는다!
    NutritionFacts cocaCola = new Builder(240, 8)  // 필수 필드만 호출
            .calories(100)  // 일종의 setter로 optional한 필드 설정
            .sodium(35)  // optional한 필드는 설정해도 되고 안해도 됨
            .carbohydrate(27)
            .build();  // build()로 필요한 객체를 얻기
}
```

<aside>
💡 불변 객체를 만들고 싶다면 빌더 패턴을 고려하자!

</aside>

## 해결 4. Lombok의 @Builder (해결 3.의 연장)

Builder 패턴 사용 시, 코드의 양이 굉장히 많아진다.

```java
@Builder
public class NutritionFacts {
		private final int servingSize;
    private final int servings;
    private final int calories;
    private final int fat;
    private final int sodium;
    private final int carbohydrate;
}
```

→ `@Builder` 어노테이션을 붙여주기만 하면 된다. 개발자가 직접 작성해야 할 코드 양이 엄청 줄어든다!

```java
public static void main(String[] args) {
    NutritionFacts cocaCola = new NutritionFactsBuilder()
            .servingSize(240)    
            .servings(8)
            .calories(100)
            .sodium(35)
            .carbohydrate(27)
            .build();
}
```

### 문제점 1. @AllArgsConstructor 생성, 외부에서 호출 가능하게 됨

해결 가능!

Lombok이 모든 필드를 매개변수로 가지는 생성자를 생성하는데, 외부에서 모든 필드를 매개변수로 가지는 생성자 호출을 막고 싶다면 아래처럼 `AccessLevel을 PRIVATE로` 설정해주면 된다.

```java
@Builder
**@AllArgsConstructor(access = AccessLevel.PRIVATE)**
public class NutritionFacts {
		...
}
```

### 문제점 2. 필수 필드 지정 불가

해결 가능!

- `Builder 패턴`

  Lombok이 제공하는 Builder가 아닌 Builder 패턴을 사용할 때, `servingSize와 servings`는 필수 필드이므로 Builder의 파라미터에 넣어줌으로써 명시가 가능하다.

    ```java
    public static void main(String[] args) {
        NutritionFacts cocaCola = new Builder(240, 8)
                .calories(100)
                .sodium(35)
                .carbohydrate(27)
                .build();
    }
    ```

- `Lombok @Builder`

    ```java
    public static void main(String[] args) {
        NutritionFacts cocaCola = new NutritionFactsBuilder()
                .calories(100)
                .sodium(35)
                .carbohydrate(27)
                .build();
    }
    ```

  `servingSize와 servings`는 필수 필드이지만, 필수로 넣어야 한다는 제약이 없다.

    ```java
    @Builder
    public class NutritionFacts {
    
        private final int servingSize;
        private final int servings;
        private final int calories;
        private final int fat;
        private final int sodium;
        private final int carbohydrate;
    
        public static NutritionFactsBuilder builder(int servingSize, int servings) {
            return new NutritionFactsBuilder()
    		        .servings(servings).servingSize(servingSize);
        }
    }
    ```

    ```java
    public static void main(String[] args) {
        NutritionFacts nutritionFacts2 = NutritionFacts.builder(240, 8)
    				.calories(120)
    				.build();
    }
    ```

  클래스에 builder라는 정적 메서드를 두어서,
  빌더 객체를 생성하기 전에 필수 필드를 넣어야만 하는 제약을 추가해준다.

  → `NutritionFactsBuilder` 타입을 반환하는 `builder` 메서드를 통해
  필수 필드인  `servingSize와 servings`를 받는다!