# 아이템 47. 반환 타입으로는 스트림보다 컬렉션이 낫다
## 스트림과 컬렉션
### 스트림
Stream은 Iterable를 확장하였다.
스트림은 반복(iteration)을 지원하지 않는다.


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
