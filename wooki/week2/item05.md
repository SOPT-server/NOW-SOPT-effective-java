> 클래스가 내부적으로 하나 이상의 자원에 의존하고, 그 자원이 클래스 동작에 영향을 준다면 `의존 객체 주입`을 도입하자.
>

<aside>
💡 사용하는 자원에 따라 동작이 달라지는 클래스에는 `정적 유틸리티 클래스`나 `싱글턴 패턴`이 적절하지 않다!

</aside>

<aside>
💡 `의존성 주입(Dependency Injection)` 패턴을 사용해라!

</aside>

자원을 명시한다 = 직접 new 를 통해 객체를 생성한다.

# 많은 클래스는 하나 이상의 자원에 의존한다

많은 클래스는 하나 이상의 자원에 의존하는데 이를 정적 유틸리티 클래스나 싱글턴으로 구현한다.

- `정적 유틸리티 클래스`로 구현

    ```java
    public class SpellChecker {
        **private static final Dictionary dictionary = new Dictionary();**
    
        private SpellChecker() {}
    
        public static boolean isValid(String word) {
            // TODO 여기 SpellChecker 코드
            return dictionary.contains(word);
        }
    
        public static List<String> suggestions(String typo) {
            // TODO 여기 SpellChecker 코드
            return dictionary.closeWordsTo(typo);
        }
    }
    ```

- `싱글톤`으로 구현

    ```java
    public class SpellChecker {
        **private final Dictionary dictionary = new Dictionary();**
    
        private SpellChecker() {}
    
        public static final SpellChecker INSTANCE = new SpellChecker();
    
        public boolean isValid(String word) {
            // TODO 여기 SpellChecker 코드
            return dictionary.contains(word);
        }
    
        public List<String> suggestions(String typo) {
            // TODO 여기 SpellChecker 코드
            return dictionary.closeWordsTo(typo);
        }
    }
    ```


## 단점 1. 여러 자원에 의존하는 경우 확장에 용이하지 않다. (유연하지 않다)

SpellChecker가 다른 종류의 dictionary를 사용하는 경우 코드가 더러워진다.

‘책임을 분리하라’는 관점에서 봤을 때 SpellChecker는 dictionary를 사용하기만 하면 되지, 어떻게 생성되는지는 몰라도 된다. 하지만 생성에 대한 책임도 가지고 있다.

## 단점 2. 테스트가 어렵다.

Dictionary를 mocking 할 수 없다.

SpellChecker를 테스트하는 것이 목적인데, SpellChecker가 Dictionary에 의존하고 있는 것이 문제. Dictionary는 어떤 수많은 클래스에 의존하고 있는지/Dictionary에 의존하기 위해 얼마나 많은 리소스를 태워야 하는지.. → 결국 인터페이스를 도입함으로써 강한 의존성을 끊어내고 mocking 할 수 있게 된다!

## 해결! 의존성 주입(Dependency Injection)

```java
public class SpellChecker {
    **private final Dictionary dictionary;**

    public SpellChecker(Dictionary dictionary) {
        this.dictionary = dictionary;
    }
    
    public boolean isValid(String word) {
        // TODO 여기 SpellChecker 코드
        return dictionary.contains(word);
    }

    public List<String> suggestions(String typo) {
        // TODO 여기 SpellChecker 코드
        return dictionary.closeWordsTo(typo);
    }
}
```

Dictionary라는 인터페이스에 의존하게 한다.

### 여러 자원에 의존해도 확장에 용이하다.

‘책임을 분리하라’는 관점에서 봤을 때 SpellChecker는 dictionary를 사용하기만 하면 되지, 어떻게 생성되는지는 몰라도 된다. 하지만 생성에 대한 책임도 가지고 있다.
→ 생성에 대한 책임은 외부(클라이언트)로 넘겼다.(SRP)

가능한 상위 타입(인터페이스)으로 인자를 받음으로써 확장에 열려있다.
→ 구현체가 아닌 인터페이스에 의존함으로써 느슨한 결합을 가지고, 구현체는 런타임에 지정된다.(DIP)
→ 인터페이스를 구현하는 새로운 클래스가 생겨도 영향받지 않는다.(OCP) 재사용도 가능하다!

### 테스트에 용이하다.

이런 식으로 Dictionary를 구현하는 구현체를 얼마든지 만들 수 있다.

```java
public class MockDictionary implements Dictionary{
    @Override
    public boolean contains(String word) {
        return false;
    }

    @Override
    public List<String> closeWordsTo(String typo) {
        return null;
    }
}
```

또 이런 mocking을 목적으로 얼마든지 만들어낸 클래스들을 통해 테스트가 가능하다.

SpellChecker를 테스트하고 싶은 목적이기에 Dictionary에 구애받지 않을 수 있다. (외부 의존성에 영향을 받지 않는다) 즉, 테스트의 격리를 보장할 수 있다.

```java
class SpellCheckerTest {
    @Test
    void isValid() {
        SpellChecker spellChecker = new SpellChecker(MockDictionary::new);
        spellChecker.isValid("test");
    }
}
```