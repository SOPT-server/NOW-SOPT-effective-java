# Item 05. 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라.

**사용하는 자원에 따라 동작이 달라지는 클래스에는 정적 유틸리티 클래스나 싱글턴 방식이 적합하지 않다.**
클래스가 내부적으로 하나 이상에 의존하고, 그 자원이 클래스 동작에 영향을 준다면 싱글톤, 정적 유틸리티 클래스 --> 의존 객체를 주입.

```Java

public class SpellChecker {  
    private final Lexicon dictionary;  
  
    public SpellChecker(Lexicon dictionary) {  
        this.dictionary = Objects.requireNonNull(dictionary);  
    }  
    public boolean isValid(String word) {  
        ...  
    }  
  
    public List<String> suggestions(String typo) {  
        ...  
    }  
}

```

생성자에 자원 팩토리를 넘겨주는 응용 방법도 있다.
의존 객체주입 -> 유연성과 테스트 용이성을 개선해주지만 의존성이 커지면 복잡해짐.
