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

### 불완전한 해결 방법
- 물론 `addAll()`메서드를 재정의하지 않으면 당장의 문제를 해결 할 수 있다. 하지만 이 해결법은 `HashSet의` `addAll()`이 `add()` 메서드를 이용해 구현했음을 알아야 가능한 것이다.
- `addAll()`을 재정의 하는 방법도 있으나, 구현에 시간이 걸리고 자칫 성능을 저하시킬 수 있다. 또한 상위 클래스의 private 필드를 써야하는 상황이면 이 방식으로는 구현이 불가능하다.
#### 한계
- 컬렉션을 상속하여 원소를 추가할 때, 특정 조건을 만족해야 하는 프로그램을 생각해보자. **상위 클래스에서 새로운 메서드를 추가**한다면, 하위 클래스에서 재정의 하지 못한 그 새로운 메서드를  사용해 '허용되지 않은' 원소를 추가할 가능성이 생긴다.
- 이 처럼 하위 클래스는 상위 클래스의 변화에 따라 깨지기가 쉽다.

### 불완전한 해결 방법2
메서드를 재정의 하는게 원인이었으나, 클래스를 확장하더라도 새로운 메서드를 추가하면 되지 않을까? 오산이다. 비교적 안전한 것은 맞지만 위험이 전혀없지 않다.
- 상위 클래스에 새 메서드가 추가됐는데, 하위 클래스에서 작성한 메서드들이 의도치 않게 메서드 오버로딩(시그니처가 같고 반환타입이 다름) 또는 오버라이딩(반환 타입까지 같음)을 구현하게 될 수 있다.
- 심지어 이 메서드들은 상위 클래스에서 만들어지기 전에 구현되서 상위 클래스의 메서드가 요구하는 규약을 만족하지 못할 가능성이 크다.

## 모든 것을 해결할 묘안!
기존 클래스를 확장하는 대신 **컴포지션(Composition; 구성)** 을 사용하자.
- 클래스를 상속받는 것이 아닌, 기존 클래스에 `private` 필드로 인스턴스를 참조하도록 만드는 것이다.
- 새 클래스의 인스턴스 메서드들은 기존 클래스의 대응하는 메서드를 호출해 그 결과를 반환한다. 이 방식을 전달 메서드(forwarding method)라 불린다.

### 장점
- 새로운 클래스는 기존 클래스의 내부 구현 방식의 영행에서 벗어나며, 심지어 기존 클래스에 새로운 메서드가 추가되더라도 전혀 영양받지 않는다.

### 컴포지션 예
``` java
// 코드 18-2 래퍼 클래스 - 상속 대신 컴포지션을 사용했다. (117-118쪽)
public class InstrumentedSet<E> extends ForwardingSet<E> {
    private int addCount = 0;

    public InstrumentedSet(Set<E> s) {
        super(s);
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
        InstrumentedSet<String> s = new InstrumentedSet<>(new HashSet<>());
        s.addAll(List.of("틱", "탁탁", "펑"));
        System.out.println(s.getAddCount());
    }
}
```

``` java
// 코드 18-3 재사용할 수 있는 전달 클래스 (118쪽)
public class ForwardingSet<E> implements Set<E> {
    private final Set<E> s;
    public ForwardingSet(Set<E> s) { this.s = s; }

    public void clear()               { s.clear();            }
    public boolean contains(Object o) { return s.contains(o); }
    public boolean isEmpty()          { return s.isEmpty();   }
    public int size()                 { return s.size();      }
    public Iterator<E> iterator()     { return s.iterator();  }
    public boolean add(E e)           { return s.add(e);      }
    public boolean remove(Object o)   { return s.remove(o);   }
    public boolean containsAll(Collection<?> c)
                                   { return s.containsAll(c); }
    public boolean addAll(Collection<? extends E> c)
                                   { return s.addAll(c);      }
    public boolean removeAll(Collection<?> c)
                                   { return s.removeAll(c);   }
    public boolean retainAll(Collection<?> c)
                                   { return s.retainAll(c);   }
    public Object[] toArray()          { return s.toArray();  }
    public <T> T[] toArray(T[] a)      { return s.toArray(a); }
    @Override public boolean equals(Object o)
                                       { return s.equals(o);  }
    @Override public int hashCode()    { return s.hashCode(); }
    @Override public String toString() { return s.toString(); }
}
```

- `InstrumentedSet`클래스는 `HashSet`의 모든 기능을 정의한 `Set` 인터페이스를 활용해 설계되어 견고하고 아주 유연하다.
- 상속 방식은 구체 클래스 각각을 따로 확장해야 하며, 지원하고 싶은 상위 클래스의 생성자 각각에 대응하는 생성자를 별도로 정의해줘야 한다.
- 컴포지션 방식은 한 번만 구현해두면 어떠한 `Set` 구현체리도 계측할 수 있으며, 기존 생성자들과 함께 사용할 수 있다.

### Wrapper 클래스
- Set인스턴스를 감싸고(warp)있다는 뜻에서 InstrumentedSet 같은 클래스를 래퍼 클래스라고 하며, 다른 Set에 계측 기능을 덧씌운다는 뜻에서 데코레이터 패던이라고 한다.
- 컴포지션과 전달의 조합은 넓은 의밀로 위임(delegation)이라고 부른다.
  - 단 엄밀히 따지면 래퍼 객체가 내부 객체에 자기 자신의 참조를 넘기는 경우만 위임에 해당한다.
#### 단점
래퍼 클래스는 단점이 거의 없지만, 콜백(callback) 프레임워크와는 어울리지 않는다는 점만 주의하면 된다.
- 콜백 프레임워크에서는 자시 자신의 참조를 다른 객체에 넘겨서 다음 호출(콜백) 때 사용하도록 한다.
  - 내부 객체는 자신을 감싸고 있는 래퍼의 존재를 모르니 대신 자신(this)의 참조를 넘기고, 콜백 때는 래퍼가 아닌 내부 객체를 호출하게 된다.
  - 이를 SELF 문제라고 한다.

### 전달 메서드와 레퍼 클래스의 성능에 관하여
전달 메서드와 래퍼 객체가 성능에 영향을 끼칠 것이라 걱정할 수 있지만, 실전에는 둘 다 별다른 영향이 없다고 밝혀졌다. 

## 컴포지션을 써야 할 때
- 상속은 반드시 하위 클스가 상위 클래스의 '진짜' 하위 타입인 상황에서만 쓰여야 한다. 그렇지 않다면 컴포지션을 사용하는 것이 정답인 경우가 대다수다.
- 컴포지션을 써야 할 상황에서 상속을 사용하는 건 내부 구현을 불필요하게 노출하는 꼴이다.
  - 그 결과 API가 내부 구현에 묶이고 그 클래스의 성능도 영원히 제한된다.
- 확장하려는 클래스의 API에 아무런 결함이 없는가? 결함이 있다면 클래스의 API까지 전파돼도 괜찮은가?
  - 컴포지션으로는 이 결함을 숨기는 새로운 API를 설계할 수 있지만, 상속은 상위 클래스의 API를 '그 결함까지도' 그대로 승계한다.

### 읽고나서... - 19.02.06
'아직 내 얘기가 아니구나'라는 것을 다시 한번 깨닳았다. 물론 이 공부가 의미없다고는 생각하지 않는다. 분명 나중에 '아...! 그때!'라고 생각날 때가 있겠지. 

콜백에 관한 설명에서는 이해가 조금 힘들었다. 나중에 알게될 날이 있을까?

