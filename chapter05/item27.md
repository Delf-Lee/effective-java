# 아이템 27. 비검사 경고를 제거하라
제네릭을 사용한다면 컴파일러 경고를 많이 보게 될것이다.
## 코드 수정을 통한 경고 제거
- 보통은 컴파일러가 알려준대로 수정하면 경고는 사라진다.
- 제거하기 더 어려운 경고도 있지만 **되도록이면 모든 비검사 경고를 제거하라**
  
## 경고 숨기기
### `@SuppressWarnings` 애너테이션
- 경고를 제거할 수는 없지만 타입이 **안전하다고 확신**할 수 있다면, `@SuppressWarnings("unchecked")` 애너테이션을 달아 경고를 숨기자
  - 단, 타입 **안전함을 검증하지 않은 채** 경고를 숨기면 스스로에게 잘못된 보안 인식을 심어주는 꼴이다.
  - 그 코드는 컴파일은 되겠지만, 런타임에는 여전히 `ClassCastException`을 던질 수 있다.
- 안전하다고 검증된 비검사 경고를 (숨기지 않고) 그대로 두면, **진짜 문제를 알리는 새로운 경고**가 나와도 눈치채지 못할 수 있다.
  - `@SuppressWarnings` 애너테이션은 개별 지역변수 선언부터 클래스 전체까지 어떤 선언에도 달 수 있다.
  - **하지만 항상 가능한 한 좁은 범위에서 사용하자**
    - 보통은 변수 선언, 아주 짧은 메서드, 혹은 생성자가 될것이다.
    - 자칫 심각한 경고를 놓칠 수 있으니 절대 클래스 전체에 적용해서는 안된다.
    - 한 줄이 넘는 메서드나 생성자에 달린 `@SuppressWarnings` 애너테이션을 발견하면 지역변수 선언 쪽으로 옮기자.
      - 이를 위해 지역 변수를 새로 언언하는 수고를 해야할 수 도 있지만, 그만한 값어치가 있을 것이다.

### 예시
#### 변경 전
``` java
public <T> T[] toArray(T[] a) {
    if(a.length < size)
        return (T[]) Arrays.copyOF(elements, size, a.getClass()); // 이 부분
    System.arraycopy(elements, 0m a, 0, size);
    if(a.length > size)
        a[size]= null;
    return a;
}
```
#### 변경 후
``` java 
public <T> T[] toArray(T[] a) {
    if(a.length < size) {
        // 생성한 배열과 매개변수로 받은 배열의 타입이 모두 T[]로 같으므로 올바른 형변환이다.
        @SuppressWarnings("unchecked")
        T[] result = (T[]) Arrays.copyOF(elements, size, a.getClass());
        return result;
    }
    System.arraycopy(elements, 0m a, 0, size);
    if(a.length > size)
        a[size]= null;
    return a;
}
```

### 주의할 점
`@SuppressWarnings("unchecked")` 애너테이션을 사용할 때면 그경고를 무시해도 안전한 이유를 항상 주석으로 남겨야한다.
