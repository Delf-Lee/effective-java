# 아이템 54. null이 아닌, 빈 컬렉션이나 배열을 반환하라
null을 반환한다면 그에 대한 방어 코드를 추가해줘야 한다. 이는 예상치 못한 오류를 발생시킬 가능성을 만들고, 코드도 복잡해진다.

``` java
public ArrayList(Collection<? extends E> c) {
    elementData = c.toArray();
    if ((size = elementData.length) != 0) {
        // c.toArray might (incorrectly) not return Object[] (see 6260652)
        if (elementData.getClass() != Object[].class)
            elementData = Arrays.copyOf(elementData, size, Object[].class);
    } else {
        // replace with empty array.
        this.elementData = EMPTY_ELEMENTDATA;
    }
}
```

``` java
private static final Cheese[] EMPTY_CHEESE_ARRAY = new Cheese[0];

private static Cheese[] getCheeses() {
    return emptyList.toArray(EMPTY_CHEESE_ARRAY);
}

private static Cheese[] getCheeses() {
    return emptyList.toArray(EMPTY_CHEESE_ARRAY);
}
```

- Shipilёv16에 대하여
  - https://shipilev.net/blog/2016/arrays-wisdom-ancients/