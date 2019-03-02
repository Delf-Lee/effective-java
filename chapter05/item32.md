 # 아이템 32. 제네릭과 가변인수를 함께 쓸 때는 신중하라 
 ## 가변인수(varargs) 메서드
 - 명시한 타입의 매개변수를 0개 이상 받을 수 있는 메서드. 
 - 해당 인수들을 배열로 저장해서 건네준다.
 ``` java
 public sum(int... args) {
     int sum = 0;
     for(int n : args) sum += n
     return sum;
 }
 ```

## 힙 오염(Heap pollution)
- 힙 오염은 파라미터화된 타입의 변수가 파라미터화된 타입이 아닌 객체를 참고할 때 발생한다.
  - 이 상황은 컴파일 시 unchecked 경고를 발생하는 코드를 수행할 때 일어난다.
  - unchecked 경고가 컴파일 또는 런타임에 발생하면, 파라미터화 된 타입을 포함하는 동작의 올바름을 검증할 수 없다. 
  - 예를들어 힙 오염은 로 타입과 파라미터화된 타입을 섞어 쓸 때나, unchecked 경고를 내는 타입 캐스트를 수행할 때 발생한다.
- **reference**
  - https://reiphiel.tistory.com/entry/potential-heap-pollution-via-varargs-parameter
  - https://en.wikipedia.org/wiki/Heap_pollution
  - http://josh.prever.co.kr/java/whatisheappollution

## 힙 오염과 가변인수
- **매개변수화 타입**의 변수가 **다른 타입의 객체**를 참조하면 힙 오염이 발생한다.
  - 다른 타입의 객체를 참조하는 상황(힙 오염이 발생한 상황)에서는 컴파일러의 자동형변환이 실패할 수 있다.

``` java
    static void dangerus(List<String>... stringLists) {
        List<Integer> intList = List.of(42);
        Object[] objects = stringLists;
        objects[0] = intList;             // 힙 오염 발생   
        String s = stringLists[0].get(0); // ClassCastException
    }
```

### varargs의 모순
- 제네릭 배열을 (프로그래머가) 직접 생성하는 것은 허용되지 않는다.
  - (ex. `new E[]`, `new List<E>`, `new List<String>`)
- 그런데 제네릭 vargars 매개변수를 받는 메서드르 선언할 수 있다.
  - 결과적으로 메서드 내에서 제네릭 배열을 사용할 수 있다.
- 이것을 허용한 이유는 **실무에서 매우 유용하기 때문**이다.
- 자바 라이브러리에도 이런 메서드를 여럿 제공한다.
  - `Arrays.asList(T... a)`
  - `Collections.addAll(Collection< super T> c, T... elements`
  - `EnumSet.of(E first, E... reset)`
  - ...
  
## `@SafeVarargs`
- 그 메서드가 타입 안전함(type-safe)을 보장한다면 `@SafeVarargs` 에너테이션으로 경고를 숨길 수 있다.
- 하지만 메서드가 안전한 게 확실하지 않다면 사용하지 말도록 하자

### type-safe한 메서드인지 판단하기
- varargs 매개변수를 담는 제네릭 배열이 만들어지고 다음을 만족할 때, 타입 안전하다
  - 메서드가 이 배열에 아무것도 저장하지 않는다. - (그 매게변수를 덮어쓰지 않는다)
  - 그 배열의 참조가 밖으로 노출되지 않는다. - (신뢰할 수 없는 코드가 배열에 접근할 수 없다) 
- 이 때, varargs 매개변수 배열에 아무것도 저장하지 않고도 타입 안정성을 깰 수 있으니 주의해야한다.

### 안전하지 않은 예
제네릭 매개변수 배열의 참조를 노출한 예
``` java
static <T> T[] toArray(T... args) {
    return args;
}
```

``` java
static <T> T[] pickTwo(T a, T b, T c) {
    switch(ThreadLocalRandom.current().nextInt(3)) {
        case 0: return toArray(a ,b);
        case 1: return toArray(b, c);
        case 2: return toArray(c, a);
    }
    throw new AssertionError();
}
```
``` java
public static void main(String[] args) {
    String[] attrubute = picTwo("좋은, "빠른", 저렴한");
}
```

- varargs로 만들어진 배열은 `Object`배열이다.
- attrubute(`String` 배열)가 `Object` 배열을 참조했다.
  - 컴파일러는 내부에서 `String[]`으로 형변환을 시도한다.
  - 하지만 `Object[]`는 `String[]`의 하위타입이 아니므로 `ClassCastExceiption`이 발생한다.

이 예는 제네릭 varargs 매개변수 배열에 다른 메서드가 접근하도록 허용하면 안전하지 않다는 점을 다시한번 상기 시킨다.

### 예외
- `@SafeVarargs`가 제대로 어노테이트 되있는 또 다른 varargs 메서드에 넘기는 것은 인전하다.
- 이 배열 내용의 일부 함수를 호출하는 일반 메서드에 넘기는 것도 안전하다.

### `@SafeVarargs` 사용 규칙
- 제네릭이나 매개변수화 타입의 varargs 매개변수를 받는 모든 메서드에 `@SafeVarargs`를 달라.
  - 단 진짜 안전하지 점검하라.
- 다음 두 조건을 모두 만족한다면 그 제네릭 varargs 메서드는 안전하다.
  - varargs 매개변수 배열에 아무것도 저장하지 않는다.
  - 그 배열(혹은 복제본)을 신회할 수 없는 코드에 노출하지 않는다.

### 다른 우회 방법
- 타입 안전함을 체크하고 `@SafeVarargs`를 사용하는 것도 좋지만, 유일한 정답은 아니다.
  - 아이템 28의 조언을 따라 varargs대신 List 매개변수로 바꿀 수 있다.
    - 이 방식의 장점은 컴파일러가 이 메서드의 안전성을 검증하고 보장한다.
    - 단점은 속도가 살짝 느릴 수 있고, 코드가 살짝 지저분해질 수 있다.

---
# 메모
> 힙 오염은 컴파일 시 unchecked 경고를 발생하는 코드를 수행할 때 일어난다.
힙 오염이 일어날만한 짓을 했을 때 unchecked 경고를 발생하는게 아니라?

- 힙 오염
  - 지금 이해한걸론... 힙오염은 로타입을 이용한 잘못된 타입캐스팅을 시도할 때 발생하는 것 같다.
  - 근데 왜 "힙" 오염일까?