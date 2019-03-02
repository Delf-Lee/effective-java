# 아이템 33.타입 안전 이종 컨테이너를 고려하라
## 타입 안전 이종 컨테이너(Type safe heterogeneous conatainer pattern)
- 기존의 컨테이너는 하나(또는 둘)의 단일원소만 사용되고 매개변수화할 수 있는 타입의 수가 제한된다.
  - `Set<E>` `Map<K,V>`등의 컬릭션, `ThreadLocal<T>`, `AtomicReference<T>` 등.
- 데이터베이스의 행(row)과 열(column)이 여러 개이듯 여러 타입을 **타입 안전**하게 넣을 수 있는 디자인 패턴이다.
  - 컨테이너 대신 키를 매개변수화 한 다음, 컨테이너에 값을 넣거나 뺄 때 매개변수화한 키를 함께 제공하면된다.

## 예
타입별로 즐겨 찾는 인스턴스를 저장하고 검색할 수 있는 `Favorites` 클래스를 생각해보자.
```java
public class Favorites {
    private Map<Class<?>, Object> favorites = new HashMap<>();

    public <T> void putFavorite(Class<T> type, T instance) {
        favorites.put(Objects.requireNonNull(type), instance);
    }

    public <T> T getFavorite(Class<T> type) {
        return type.cast(favorites.get(type));
    }
}
```
- class의 클래스가 제네릭이기 때문에 각 타입의 Class 객체를 매개변수화한 키 역할로 사용이 가능하다.
  - class의 리터럴 타입은 `Class`가 아닌 `Class<T>`이다.
  - `String.class`의 타입은 `Class<String>`이고, `Integer.class`의 타입은 `Class<Integer>`

### 타입 토큰(Type token)
컴파일 타임 타입 정보와 런타임 타입 정보를 알아내기 위해 메서드들이 주고받는 class 리터럴을 타입 토큰이라 한다.

### 알아두어야 할 점
```java
private Map<Class<?>, Object> favorites = new HashMap<>();
```
- 맵이 아니라 키가 와일드 카드이이고 이 와일드 카드 타입은 중첩(nested)되었다.
  - 이는 모든 키가 서로 다른 매개변수화 타입일 수 있다는 의미다.
- `favorites` 맵의 값 타입은 단순히 `Object`이다.
  - 키와 값 사이의 타입 관계를 보증하지 않는다.
  - 하지만 `getFavorite` 메서드에서 이 관계를 살릴 수 있다.
  - 해당 타입 `Class`의 `cast` 메서드를 이용해 동적으로 형변환 한다.

``` java
public class Class<T> {
    T cast Object(Object obj);
}
```
- `cast`의 반환 타입은 Class 객체의 타입 매개변수와 같다.
  - `cast` 메서드의 시그니처가 Class 클래스가 제네릭이라는 이점을 완벽히 활용한다.


## 제약
지금의 Favorites 클래스에는 알아둬야할 제약이 두 가지 있다.
### 1. 로 타입으로 인한 불안정성
악의적인 클라이언트가 Class 객체를 (제네릭이 아닌) 로 타입으로 넘기면 Favorite 인스턴스의 타입 안정성이 쉽게 깨진다.
``` java
f.putFavorite((Class) Integer.class, "Integer의 인스턴스가 아닙니다.");
int favoriteInteger = f.getFavorite(Integer.class);
```
위의 코드에서 `putFavorite`를 호출할 때는 아무 불평이 없다가 `getFavorite`를 호출할 때 `ClassCastException`을 던진다.

Favorites가 타입 불변식을 어기는 일이 없도록 보장하려면 `putFavorite`메서드에서 인수로 주어진  `instance`의 타입이 type으로 명시한 타입과 같은지 확인하면 된다.

``` java
public <T> void putFavorite(Class<T> type, T instance) {
    favorites.put(Objects.requireNonNull(type), type.cast(instance));
}
```
java.util.Colletions에는 `checkedSEt`, `chedckedList`, `checkedMap`같은 메서드가 있는데, 바로 이런 방식을 적용한 컬렉션 래퍼들이다.

### 2. 실체화 불가 타입에는 사용할 수 없다.
- `String`이나 `String[]`은 저장할 수 있어도 `List<String>`은 저장할 수 없다.
- `List<String>`과 `List<Integer>`는 `List.class`라는 같은  Class 객체를 공유한다.

#### 슈퍼 타입 토큰(suepr type token)
두 번째 제약을 슈퍼 타입 토큰(suepr type token)으로 해결하려는 시도도 있다. 예제의 Favorite에 슈퍼 타입 토큰을 적용하면 타음 코드처럼 제네릭 타입도 문제없이 저장될 수 있다.

``` java
Favorite f = new Favorite();
List<String> pets = Arrays.asList("개", "고양이", "앵무");

f.putFavorite(new TypeRef<List<String>>(){}, pets);
List<String> listofStrings = f.getFavorite(new new TypeRef<List<String>>(){});
```
- 닐 게프터의 글
  - http://gafter.blogspot.com/2006/12/super-type-tokens.html

이 방식도 한계가 존재하고 완벽하지 않으니 주의헤서 사용해야한다.
  
---

# 메모
- 타입 안전 이종 컨테이너? 타입에 관계없이 저장가능?