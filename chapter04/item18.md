# 아이템 18. 상속보다는 컴포지션을 사용하라 
#### 상속은 코드를 재사용하는 강력한 수단이지만, 항상 최선은 아니다.
여기서 말하는 상속은 *(클래스가 클래스를 확장하는)* **구현 상속**을 말하며, *(클래스가 인터페이스를 구현하거나 이터페이스가 다른 인터페이스르 확장하는)* **인터페이스 상속**과는 무관하다.

#### 예외
상위 클래스와 하위 클래스를 한 패키지 내에서 모두 같은 프로그래머가 통제하는 상황이거나, 확장할 목적으로 설계되었고 문서화도 잘 된 클래스를 확장하는 것은 안전하다.



## 잘못된 확장
> 메서드 호출과 달리 상속은 캡슐화를 깨뜨린다.[Snyder86]

#### 상위 클래스가 어떻게 구현되느냐에 따라 하위 클래스의 동작에 이상이 생길 수 있다.
상위 클래스의 릴리스마다 내부 구현이 바뀔 수 있고, 그 여파로 코드 한 줄 건드리지 않은 하위 클래스가 오작동할 수 있다. 때문에 확장을 고려하지 않았거나, 문서화가 잘 되지 않은 클래스의 하위 클래스는 변화에 맞추어 수정해야 한다.

### 잘못된 확장의 예
`HashSet`을 확장하여 생성된 이후 원소가 몇 개 더해졌는지 카운트가 가능한 기능을 추가하였다고 가정해보자. (size와 무관하다)

`addCount`필드를 추가하고 `add()`메서드와 `addlAll()`메서드를 재정의하였다.

[InstrumentedHashSet.java](/src/effectivejava/chapter4/item18/InstrumentedHashSet.java)
``` java
public class InstrumentedHashSet<E> extends HashSet<E> {
    // 추가된 원소의 수
    private int addCount = 0;

    public InstrumentedHashSet() {
    }

    public InstrumentedHashSet(int initCap, float loadFactor) {
        super(initCap, loadFactor);
    }

    @Override public boolean add(E e) {
        addCount++;
        return super.add(e);
    }

    @Override public boolean addAll(Collection<? extends E> c) {
        addCount += c.size();
        return super.addAll(c);
    }

    public int getAddCount() {
        return addCount;
    }

    public static void main(String[] args) {
        InstrumentedHashSet<String> s = new InstrumentedHashSet<>();
        s.addAll(List.of("틱", "탁탁", "펑"));
        System.out.println(s.getAddCount());
    }
}
```

이 클래스는 잘 동작하지 않는데, `addAll()`메서드가 `add()`메서드를 사용하여 구현하였기 때문이다. 따라서 한 요소당 두번 카운트되고 `main()`을 실행하면 6이라는 결과가 온다.

### 해결과 다른 문제
- 물론 `addAll()`메서드를 재정의하지 않으면 당장의 문제를 해결 할 수 있다. 하지만 이 해결법은 `HashSet의` `addAll()`이 `add()` 메서드를 이용해 구현했음을 알아야 가능한 것이다.
- `addAll()`을 재정의 하는 방법도 있으나, 구현에 사긴이 걸리고 자칫 성능을 저하시킬 수 있다. 또한 private 필드를 써야하는 상황이면 이방식으로는 구현이 불가능하다.