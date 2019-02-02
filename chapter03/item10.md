# 아이템 10. `equals`는 일반 규약을 지켜 재정의하라
## `equals`를 재정의할 필요가 없을 때
다음 열거한 상황 중 하나에 해당한다면 `equals`를 재정의하지 않는 것이 최선이다.
- 각 인스턴스가 본질적으로 고유하다.
- 인스턴스의 '논리적 동치성(logical equality)'을 검사할 일이 없다.
- 상위 클래스에서 재정의한 equals가 하위 클래스에도 딱 들어맞는다.
- 클래스가 private이거나 package-private이고 `equals` 메서드를 호출할 일이 없다.

## `equals`를 재정의해야할 때
- 상위 클래스의 `euqlas`가 **논리적 동치성**을 비교하도록 재정의 되지 않았을 때.
  - 객체 식별성(object identity; 두 객체가 물리적으로 같은가)이 아니다.
  - 주로 **값 클래스**(`Integer`, `String` 등)가 여기에 해당한다.
  - 두 객체의 값 비교가 가능하다면 `Map`이나 `Set`의 원소로도 사용할 수 있다.
  - 값 클래스라고 해도, 같은 인스턴스가 둘 이상 만들어지지 않음을 보장한다면 `euqals`를 정의하지 않아도 된다.
    - 예를들어 인스턴스 통제 클래스(아이템 1), `Enum` class (아이템 34)
## `equals` 메소드를 재정의할 때 지켜야하는 규약
- **반사성**(reflexivity): null이 아닌 모든 참조 값 x에 대해, `x.equals(x)`는 true이다.
- **대칭성**(symmetry): null이 아닌 모든 참조 값 x, y에 대해  `x.equals(y)`가 true면 `y.equals(x)`도 true이다.
- **추이성**(transitivity): null이 아닌 모든 참조 값 x, y, z에 대해 `x.equals(y)`가 true이고 `y.equals(x)`도 true이면 `z.equals(x)`도 true이다.
- **일관성**(consistency): null이 아닌 모든 참조 값 x, y에 대해 `x.equals(y)`를 반복해서 호출하면 항상 true를 반환하거나 한상 false를 반환한다.
- **null이 아님**: null이 아닌 모든 참조 값 x에 대해, `x.equals(null)`은 false이다.
### 
### References
#### THIS :arrow_forward OTHER
- 아이템 1
- 아이템 34
#### OTHER :arrow_forward THIS아이템