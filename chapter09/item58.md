# 아이템 58. 전통적인 for 문 보다는 for-each 문을 사용하라
``` java
for (Iterator<Elemet> i = c.iterator(); i.hasNext();) {
    Element e = i.next();
    // e로 무언가를 한다.
}
```

``` java
for (int i=0 i<.a.lengh; i++) {
    // a[i]로 무언가를 한다.
}
```


``` java
for (Element e : elements) {
    // e로 무언가를 한다.
}
```
향상된 for 문(enhanced for statement)은 반복자와 index를 사용하지 않으니 코드가 깔끔해 지고, 어떤 컬렉션과 배열이든 처리할 수 있어 편리하다.

### for-each 사용이 불가능한 상황
- 파괴적인 필터링(destructive filtering)
- 변형(transforming)
- 병렬 반복(parallel itertion)

#### 개인적인 추가
이것 이외에 대표적으로 '인덱스에 의미가 있는 경우'인 것 같다. 몇번 인덱스부터 시작해야한다거나, 몇 번째 인덱스의 값을 뽑아야 된다거나 등... 보통은 컬렉션보다 배열을 쓸때 이런 경우가 많겠지


