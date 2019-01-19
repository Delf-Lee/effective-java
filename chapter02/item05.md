# 아이템5: 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라
> 사용하는 자원에 따라 동작이 달라지는 클래스에는 정적 유틸리티 클래스나 싱글턴 방식이 적합하지 않다.

대신 클래스가 여러 자원 인스턴스를 지원해야 하며, 클라이언트가 원하는 자원을 사용해야 한다. 이를 위해서 **인스턴스를 생성힐 때 생성자에 필요한 자원을 넘겨준다.**

이 패턴의 쓸만한 변형으로 생성자에서 자원 팩터리를 남겨주는 방식이 있다. 즉, 팩터리 메서드 패턴을 구현한 것이다. `Supplier<T>` 인터페이스가 팩터리를 표현한 완벽한 예다.

> **핵심 정리**  
> 클래스가 내부적으로 하나 이상의 자원에 의존하고, 그 자원이 클래스 동작에 영향을 준다면 싱글턴과 정적 유틸리티 클래스는 사용하지 않는 것이 좋다. 이 자원들을 클래스가 직접 만들게 해서도 안 된다. 대신 필요한 자원을 (혹은 그 자원을 만들어주는 팩터리를) 생성자에 (혹은 정적 팩터리 빌더) 넘겨주자. 의존 객체 주입이라 하는 이 기법은 클래스의 유연성, 재사용성, 테스트 용이성을 개선해준다.


# 이야기해볼 것
- 인스턴스 생성 책임?
  - 객체 생성 처리를 서브 클래스로 분리 해 처리하도록 캡슐화하는 패턴 [#](https://gmlwjd9405.github.io/2018/08/07/factory-method-pattern.html)
  - 싱글톤 패턴은 태생적으로 SRP를 위반한다?

# 참고
## 팩터리 메서드 패턴
- https://gmlwjd9405.github.io/2018/08/07/factory-method-pattern.html
- https://01010011.blog/2016/12/29/java8-consumer-supplier-를-이용한-builder-pattern-구현/

## `GenericBuilder`
``` java
public class GenericBuilder<T> {
    private final Supplier<T> instantiator;
    private List<Consumer<T>> instanceModifiers = new ArrayList<>();

    public GenericBuilder(Supplier<T> instantiator) {
        this.instantiator = instantiator;
    }

    public static <T> GenericBuilder<T> of(Supplier<T> instantiator) {
        return new GenericBuilder<>(instantiator);
    }

    public <U> GenericBuilder<T> with(BiConsumer<T, U> consumer, U value) {
        Consumer<T> c = instance -> consumer.accept(instance, value);
        /*Consumer<T> c = instance -> {
            consumer.accept(instance, value);
        };*/
        instanceModifiers.add(c);
        return this;
    }

    public T build() {
        T value = instantiator.get();
        instanceModifiers.forEach(modifier -> modifier.accept(value));
        instanceModifiers.clear();
        return value;
    }
}
```

``` java
@Setter
public class Foo {
    private String a;
    private String b;

    public static void main(String[] args) {
        Foo myFoo = GenericBuilder.of(Foo::new)
                .with(Foo::setA, "A")
                .with(Foo::setB, "B")
                .build();
    }
}
```

#### 2019. 01. 16.
> 사용하는 자원에 따라 동작이 달라지는 클래스에는 정적 유틸리티 클래스나 싱글턴 방식이 적합하지 않다.
- 정적 유틸리티 클래스가 적합하지 않은건 알겠다. 어처피 멤버 변수를 갖는게 그림은 아니니까.
- 여기서 말하는 사용자 자원이 뭐지? 책의 예제에 따르면 클래스의 멤버변수 정도 되는 것 같은데.
  - 아 싱글톤은 아예 그 자원이 fix 되버리니까, 적합하지 않은거구나
- 이 아이템에서 말하고 싶은 건 생성자에서 해당 클래스가 가지고 있는 멤버 변수를 초기화하는 것... 인것 뿐인가?
  - 내가 아직 다 이해하지 못할뿐인거 같지만, 별거 없어보인다.
  - 지금은 뭔가 "생각하진 않았지만, 납득하고 있던것이었고, 구체적인 지식이 되었다" 정도
  - 이 책에 대해 평가가 높기 때문에 아직은 "내가 이해하지 못한 무언가가 있다"라고 생각 중이다.

### 메모


### References
#### THIS :arrow_forward OTHER
- 아이템 4 (28p)
- 아이템 3 (28p)
- 아이템 17 (29p)
- 아이템 1 (29p)
- 아이템 2 (29p)
- 아이템 31 (29p)
#### OTHER :arrow_forward THIS
