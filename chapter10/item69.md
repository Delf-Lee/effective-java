# 아이템 69. 예외는 진짜 예외 상황에만 사용하라
예외는 오직 예외 상황에서만 써야한다. 절대로 일상적인 제어 흐름용으로 쓰여선 안 된다.
- 예외를 생성하고 던지고 잡는 것은 비용이 많이 드는 작업이다.
- 예외처리에 대한 코드는 JVM의 최적화 대상에서 빠질 수 있다. 

## 예외 처리 관련 정보
- 하나의 `try` 블럭 안에서 모든 exception을 `catch(Exception e)` 하나로 잡으려 하지 말고, 각각의 예외가 발생할 수 있는 상황에 대하여 `try-catch` 를 따로따로 사용하라.
  - (지금은 다중 `catch`를 지원하니, 그걸 사용하면 될 것 같다)
- 프로그램의 흐름을 제어하기 위한 인위적인 exception handling을 하지 마라.
- `throws` 절에는 `Exception`을 사용하지 말고 보다 상세한 (`FileNotFoundException` 같은) Exception의 하위 클래스를 사용하라.
- exception handling을 자주 사용하라. exception이 발생하지 않을때 exception 처리를 해서 추가되는 오버헤드는 아주 작다. exception이 발생하는 상황에서만 실행시간의 오버헤드가 생긴다.
- DB connection, file, socket connection 등의 리소스를 해제할 때에는 항상 `finally` 블럭을 사용하라. 이것은 leak을 방지해준다.
- 메소드 호출을 할때 항상 exception이 발생하는 메소드 내에서 exception을 처리하라. 특별히 필요한 경우가 아니라면 호출하는 메소드에 exception을 떠넘기지 마라. 호출하는 메소드에 exception을 넘기는 것은 더 많은 실행시간이 걸리기 때문에 로컬에서 처리하는 것이 효율적이다.
- 루프 안에서 exception handling을 하지마라. `try-catch`안에 루프를 넣는 것이 좋다.

### 개인 주석
대표적으로 배열을 다룰 때, index validation을 일일히 하는것도 귀찮았고 다음과 같이 작성하는게 깔끔하다고 생각했다.
``` java
try {
    // 배열 접근 (ex. 2차원 배열에서 임의의 인덱스 기준으로 4방향 또는 8방향에 위치하는 인덱스에 접근)
} catch(ArrayIndexOutOfBoundsException e) {} // 상황에 따라 주석을 남김
```
괜찮은 해결법이라고 생각해왔는데, 최근에 얻은 지식으로는 그렇지 않다는걸 깨달았다. 

### Reference
- https://www.slipp.net/questions/350
- https://mentor75.tistory.com/entry/Exception전략-Java의-Exception-처리-최적화
- https://www.slipp.net/questions/350