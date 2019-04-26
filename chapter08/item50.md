# 아이템 50. 적시에 방어적 복사본을 만들라
자바는 안전한 언어디지만, 다른 클래스로부터의 침범을 아무런 노력 없이 막을 수 있는 것은 아니다. 어떤 객체든 그 객체의 허락 없이는 외부에서 내부를 수정하는 일은 불가능하게 해야한다.

## 방어적으로 복사하기
### 1. 내부 필드로 불변 객체를 사용한다.
### 2. 생성자에서 받은 가변 매개변수 각각을 방어적으로 복사한다.
- 생성자에서 동시성 문제 때문에 복사본을 만들고 **그 복사본**으로 유효성을 검사하는 것이 좋다.
- 복사 시, `clone` 메서드 사용에 주의해야 한다.
  - 악의를 가진 하위 클래스가 private 필드를 공개할 수도 있기 때문
  - 따라서, 매겨변수가 제 3자에 의해 확장될 수 있는 타입이라면 방어적 본사본을 만들 때 `clone` 메서드를 사용해서는 안된다.
- 접근자에서 방어적 본사본을 반환한다.

## 가변 객체와 불변 객체
- 클래스 내부의 필드가 가변이라면 항상 방어적 복사본을 만들지 고려하라.
- 되도록 불변 객체들을 조합해 객체를 구성해야 방어적 복사를 할 일이 줄어든다.

## 방어적 복사의 생략
- 호출자가 컴포넌트 내부를 수정하지 않으리라는 확신이 있다면 방어적 복사를 생략할 수 있다. 단 수정 금지임을 문서화하는 게 좋다.
- 때로는 메서드나 생성자의 매개변수로 넘기는 행위 자체가 **그 객체의 통제권을 명백히 이전함**을 뜻한다.
  - 이런 때는 방어적 복사를 사용할 필요는 없지만, 이 객체를 넘겨주는 메서드를 호출하는 클라이언트는 **해당 객체를 더이상 직접 수정하는 일이 없다고 약속**해야한다.
  - 이와 같이 통제권을 넘겨받는다고 기대하는 메서드나 생성자에 그 사실을 문서화로 남기는 것이 좋다.
- 통제권이 넘을 넘겨받은 클래스에서는 악의적인 클라이언트에 공격에 취약할 것이다.
  - 때문에 방어적 복사를 생략해도 되는 상황은 해당 클래스와 그 클라이언트가 상호 신뢰할 수 있을 때, 혹은 불변식이 깨지더라도 그 영향이 오직 호출한 클라이언트로 국한될 때로 한정해야한다.