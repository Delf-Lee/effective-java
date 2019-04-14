# 아이템 46. 스트림에서는 부작용 없는 함수를 사용하라 
스트림은 그저 또 하나의 API가 아닌, 함수형 프로그래밍에 기초한 패러다임이다.
## 스트림 패더다임
스트림 패러다임의 핵심은 계산을 일련의 변환(transformation)으로 재구성 하는 부분이다.
  - 이때 각 변환 단계는 가능한 한 이전 단계의 결과를 받아 처리하는 **순수 함수**여야한다.
  - 순수 함수란 오직 입력만이 결과에 영향을 주는 함수이다.

## 수집기(Collector)
스트림을 시용히려면 꼭 배워야 하는 새로운 개념이다. 상당히 복잡하저만 세부 내용을 잘 몰라도 이 API의 장점을 대부분 활용할 수 있다. 그저 축소(reduction) 전력을 캡슐화한 블랙박스 객체라고 생각하기 바란다.
- `toList()`, `toSet()`, `toCollection(collectionFactory)`

### toMap()
- `toMap(keyMapper, valueMapper)`
- `toMap(keyMapper, valueMapper, (oldVal, newVal) -> new Val)`

### groupBy()
입력으로 분류함수를 받고 출력을 원소들을 카테별로 모아 놓은 맵을 담은 수집기를 반환한다.
- 추가 매개변수
  - 리스트 외의 값으을 갖는 수집기를 반환하고 싶다면 다운스트림을 명시하면 된다.
  - 맵 팩터리를 명시하여 맵의 종류를 변경할 수도 있다.
- groupByConcurrent¬

### maxBy(), minBy()
Steam 인터페이스의 min과 max 메서드를 살짝 일반화한 것.
  - `java.util.function.BinaryOperator`의 minBy와 maxBy 메서드가 반환하는 이진 연산자의 수집기 버전

### joining()
`CharSequence` 타입의 구문문자(delemiter)를 매개 변수로 받는다. 연결 부위에 이 구문 문자를 삽입하한 문자열을 반환한다.
