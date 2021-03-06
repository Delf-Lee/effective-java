# 아이템 78. 공유 중인 가변 데이터는 동기화해 사용하라

## 동기화
  - 한 스레드가 공유자원을 변경할 때(상태가 일관되지 않은 순간), 그 자원을 다른 스레드가 보지 못하게 한다.
  - 뿐만 아니라, 동기화 없이는 한 **스레드가 만든 변화를 다른 스레드에서 확인하지 못할 수 있다.** 
### "일관성이 깨졌다"
  - 여러 스레드에서 같은 공유자원에 접근할 때, 이 자원이 올바른 값임을 보장할 수 없다.

## 공유객체의 가시성(visibility)
> 자바 언어 명세는 스레드가 필드를 읽을 때 항상 '수정이 완전히 반영된' 값을 얻는다고 보장하지만, 한 스레드가 저장한 값이 다른 스레드에게 '보이는가'는 보장하지 않는다. (414p)

무슨 소리인지 이해가 잘 되지 않았지만, 공유객체의 가시성(visibility)이라는 키워드에 대해 알고 검색해보니 감을 잡았다. 책이 있는 설명이 애매한 것 같다. 메모리 관점에서 설명된 글이 더 명확한 듯 하다.

- 참고
  - https://parkcheolu.tistory.com/14


## `volatile` 키워드
### 안전실패
### `Synchronized` 키워드 

### `Synchronized`와 `volatile`의 차이
> 이 상황에 대한 해결책으로는 synchronized 블록을 사용할 수 있다. 이 블록은 한 시점에 오직 하나의 쓰레드만이 특정 코드 영역에 접근할 수 있도록 보장해준다. 또한 volatile 키워드처럼, <u>**이 블록 안에서 접근되는 모든 변수들은 메인 메모리로부터 읽어들여지고, 쓰레드의 실행이 이 블록을 벗어나면 블록 안에서 수정된 모든 변수들이 즉각 메인 메모리로 반영될 수 있도록 해준다**</u> - volatile 키워드가 선언되어 있든 없든.
> 
> [자바 메모리 모델 - 박철우 블로그](https://parkcheolu.tistory.com/14)

궁금증

- 각 스레드들 무조건 다른 CPU에서 실행되는가? - 아마 NO일테지...
- 모든 스레드들은 서로 다른 CPU Cache를 가지고 있는가?
  - CPU Cache의 기준은 무엇인가.
  - 같은 CPU Cache를 공유한다면 가시성 문제는 해결되는가?


### Reference
- [Java의 volatile 키워드에 대한 이해 - IT 세미 덕후](http://kwanseob.blogspot.com/2012/08/java-volatile.html)
- [자바 volatile 키워드 - 박철우 블로그](https://parkcheolu.tistory.com/16)
- [자바 메모리 모델 - 박철우 블로그](https://parkcheolu.tistory.com/14)
- [Java의 고유 락(intrinsic lock)에 대해](http://happinessoncode.com/2017/10/04/java-intrinsic-lock/#재진입-가능성-Reentrancy)