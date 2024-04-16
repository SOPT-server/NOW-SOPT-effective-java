
- 정적 팩토리, 생성자 -> 선택적 매개변수가 많으면 대응하기 어렵다.
- 자바빈즈패턴
	- 기본생성자로 선언 후 Setter로 값 변경 ->  객체 하나를 생성하려면 여러 메소드를 호출해야 하고, 객체가 완전히 생성되기 전까지는 consistency가 무너진다.
	- 생성이 끝난 객체를 freeze하고 얼리기 전에는 사용할 수 없도록 하는 방법도 있지만, 컴파일러고 freeze 여부를 보장하지 못하므로 런타임 오류에 취약


Builder 생성 예시

```Java
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
  
        // 선택 매개변수  
        private int calories;  
        private int fat;  
        private int sodium;  
        private int carbohydrate;  
  
        public Builder(int servingSize, int servings) {  
            this.servingSize = servingSize;  
            this.servings = servings;  
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
        public NutritionFacts build() {  
            return new NutritionFacts(this);  
        }    }  
    private NutritionFacts(Builder builder) {  
        servingSize = builder.servingSize;  
        servings = builder.servings;  
        calories = builder.calories;  
        fat = builder.fat;  
        sodium = builder.sodium;  
        carbohydrate = builder.carbohydrate;  
    }}
```

Builder 패턴을 사용하면, 각 파라미터를 생성하는 생성자에서 유효성 검사를 진행할 수 있다.


```Java
public abstract class Pizza {  
    public enum Topping {HAM, MUSHROOM, ONION, PEPPER, SAUSAGE}  
  
    final Set<Topping> toppings;  
  
    abstract static class Builder<T extends Builder<T>> {  
        EnumSet<Topping> toppings = EnumSet.noneOf(Topping.class);  
  
        public T addTopping(Topping topping) {  
            toppings.add(Objects.requireNonNull(topping));  
            return self();  
        }  
        abstract Pizza build();  
  
        // 하위 클래스는 이 메서드룔 재정의(overriding)하여  
        // "this"를 반환하도록 해야 한다.  
        protected abstract T self();  
    }  
    Pizza(Builder<?> builder) {  
        toppings = builder.toppings.clone(); // 아이템 50 참조  
    }  
}
```

빌더 패턴은 계층적으로 설계된 클래스와 함께 쓰기에 좋음

```Java
import java.util.EnumSet;  
import java.util.Objects;  
import java.util.Set;  
  
public class NyPizza extends Pizza {  
    
    public enum Size { SMALL, MEDIUM, LARGE }  
    
    private final Size size;  
    
    public static class Builder extends Pizza.Builder<Builder> {  
        private final Size size;  
        
        public Builder(Size size) {  
            this.size = Objects.requireNonNull(size);  
        }        
        
        @Override public NyPizza build() {  
            return new NyPizza(this);  
        }
        
		@Override protected Builder self() { return this; }
		  
    }   
    
    private NyPizza(Builder builder) {  
        super(builder);  
        size = builder.size;  
	}
}
```

위와 같이 작성하면, NyPizza.Builder는 NyPizza를 반환함(공변 반환 타이핑(covariant return typing))

빌더 패턴의 단점.
- 비용이 크지 않지만, 성능에 민감할 때는 문제가 될 수 있다.
- 매개변수가 많아지면 값어치를 한다.
API 설계는 대부분 값이 많아지기 때문에, 빌더 패턴으로 애초에 시작하면 이득일수도 있다.
