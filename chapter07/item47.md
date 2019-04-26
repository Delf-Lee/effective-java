# 아이템 47. 반환 타입으로는 스트림보다 컬렉션이 낫다

## 스트림(Stream)과 컬렉션(Collection)
- 스트림은 외부 반복을 통해 작업하는 컬렉션과는 달리 내부 반복(internal iteration)을 통해 작업을 수행한다.
  - `Iterable`를 확장하지 않았기 때문에 (외부) iteration을 지원하지 않는다.
  - 흥미로운 건 `Stream`은 `Iterable` 인터페이스가 정의한 추상메서드를 포함하고, 그 방식 그대로 동작한다.
- 스트림은 재사용이 가능한 컬렉션과는 달리 단 한 번만 사용할 수 있다.

## `Stream` vs `Collection`
### 사용자의 편의성을 고려
- 어느 하나만 쓰일 것을 알고 있다면, 그것을 반환하도록 하면된다.
- 공개 API를 작성한다면 메서드의 반환타입을 `Collection`이나 그 하위 타입을 쓰는 것이 최선이다.
  - `Colelction` 인터페이스는 손쉽게 스트림도 지원 가능하다.
### 다른 고려해야할 점
- 하지만, 컬렉션을 반환한다는 이유로 덩치 큰 시퀀스를 메모리에 올려서는 안된다.
  - 간단하게 표현할 방법이 있다면 전용 컬렉션을 구현하는 것을 고려하라.
- 그냥 단순하게 "구현하기 쉬운" 쪽을 선택해도 괜찮다.
### 한계
- 리스트(컬렉션)을 스트립으로 변환 어댑터를 사용할 때는 성능 감소와 코드가 지저분해지는 것을 감안해야 한다.

## 의문
### for-each문
``` java
for (type var: iterate) {
    body-of-loop
}
```
iterate에 배열이든, `Iterable`을 구현한 클래스가 들어가야되는건 알겠다. 근데 코드 47-1(248p)에 다음과 같은 구문이 있다.
``` java
for(ProcessHAndle ph : ProcessHandle.allProcesses()::iterator) {
    // 프로세스 처리
}
``` 
`::`은 메서드 참조 연산자가 아닌가? 사실 위의 코드는 틀린 것이다. 컴파일 에러는 낸다. 그렇다 치자.
``` java
for(ProcessHAndle ph : (Iterable<ProcessHandle>) ProcessHandle.allProcesses()::iterator) {
    // 프로세스 처리
}
``` 
권장되지는 않지만, 저게 (문법적으로는) 옳은 코드라고 한다. 여튼 중요한건 그게 아니라,

 `Stream`의 `iterator`메서드의 반환값은 `Iterator<E>`다. 근데 `Iterable<E>`로 캐스팅이 가능하다? 이게 가능하다고?

 분명 `Iterable<E>` 인터페이스에는 구현해야할 추상 메서드가 하나 있다. `iterator` 메서드가.
 `Stream`(정확히는 부모클래스인 `BaseStream`)에도 추상 메서드 `iterator`가 선언되어있기는 하다. 여튼 최종적인 의문은
- `Stream`과 `Iterator<E>` 인터페이스는 그저 구현해야할 추상 메서드가 공통된다는 이유때문에 그냥 캐스팅해서 쓸 수 있다?
- `클래스::메서드`의 결과가 도대체 뭐야? 결국 메서드가 맞다면, 메서드를 인터페이스로 캐스팅한거야?

``` java
public static <E> Iterable<E> iterableOf(Stream<E> stream) {
    return steam::iterator;
}
```
....? 이 구문을 보면, steam::iterator의 값은 그냥 `Iterable`의 구현체이다.
