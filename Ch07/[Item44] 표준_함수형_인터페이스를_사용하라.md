# 표준 함수형 인터페이스를 사용하라

## ⛳️ 목표

- 표준 함수형 인터페이스를 활용하여 코드 가독성과 유지보수성을 높이는 방법에 대해 알아보자.

<br>

## 📄 핵심 요약

### **템플릿 메서드 패턴 vs 함수형 인터페이스**

- 템플릿 메서드 패턴 : 상속을 통해 기본 동작을 재정의
- 함수형 인터페이스 : 함수 객체를 매개변수로 전달하여 동작을 유연하게 변경.

<br>

---

### 1. 표준 함수형 인터페이스 활용

#### LinkedHashMap 예제
```java
// 기존 방식: 템플릿 메서드 패턴 사용
protected boolean removeEldestEntry(Map.Entry<K, V> eldest) {
    return size() > 100;
}

// 현대 방식: 함수 객체를 사용하는 정적 팩터리 메서드
static <K, V> LinkedHashMap<K, V> createCache(int maxSize, BiPredicate<Map<K, V>, Map.Entry<K, V>> removalPredicate) {
    return new LinkedHashMap<>() {
        @Override
        protected boolean removeEldestEntry(Map.Entry<K, V> eldest) {
            return removalPredicate.test(this, eldest);
        }
    };
}

// 사용 예시
LinkedHashMap<String, String> cache = LinkedHashMap.createCache(
    100,
    (map, eldest) -> map.size() > 100
);
```

- `BiPredicate<Map<K, V>, Map.Entry<K, V>>`는 표준 함수형 인터페이스로 `EldestEntryRemovalFunction` 대신 사용한다.
- 표준 인터페이스를 활용하면 API가 더 직관적이고, 다른 코드와의 호환성이 높아진다.

<br>

### 2. 함수형 인터페이스 유형

#### 자주 사용하는 표준 함수형 인터페이스
| 인터페이스       | 함수 시그니처                       | 사용 예                             |
|----------------|--------------------------------|---------------------------------|
| `UnaryOperator<T>` | `T apply(T t)`                  | `String::toLowerCase`            |
| `BinaryOperator<T>`| `T apply(T t1, T t2)`           | `BigInteger::add`                |
| `Predicate<T>`      | `boolean test(T t)`             | `Collection::isEmpty`            |
| `Function<T,R>`     | `R apply(T t)`                  | `Arrays::asList`                 |
| `Supplier<T>`       | `T get()`                       | `Instant::now`                   |
| `Consumer<T>`       | `void accept(T t)`              | `System.out::println`            |


<br>


### 3. 직접 함수형 인터페이스를 작성하는 경우

- 표준 인터페이스가 없는 경우에만 작성한다.
- 특정 규약을 강제하거나 유용한 디폴트 메서드를 제공하려는 경우에 활용한다.
- 작성예제 : `Comparator<T>`
> - 용도 : 두 객체를 비교한다.
> - 디폴트 메서드 제공 : `thenComparing`

#### 작성 예시
```java
@FunctionalInterface
public interface EldestEntryRemovalFunction<K, V> {
    boolean remove(Map<K, V> map, Map.Entry<K, V> eldest);
}
``` 

> ⚠️ 주의사항
> - `@FunctionalInterface`를 추가하여 함수형 인터페이스임을 명시한다.
> - 추상 메서드가 하나만 있어야 컴파일되며, 유지보수 과정에서 실수를 방지할 수 있다.

<br>


### 4. 함수형 인터페이스 활용 예제

#### 캐시 구현 (최대 크기 제한)
```java
LinkedHashMap<String, String> cache = LinkedHashMap.createCache(
    100,
    (map, eldest) -> map.size() > 100
);
```

#### 스트림과의 결합
```java
List<String> names = List.of("Alice", "Bob", "Charlie");

// 람다
names.stream()
     .map(name -> name.toUpperCase())
     .forEach(System.out::println);

// 메서드 참조
names.stream()
     .map(String::toUpperCase)
     .forEach(System.out::println);
```

<br>


### 5. 표준 함수형 인터페이스의 장점

#### 장점
1. 코드 간결화
    - 함수 객체를 전달할 때 간결하고 명확한 코드 작성 가능하다.
2. 재사용성
    - 표준 인터페이스는 다양한 라이브러리에서 활용 가능하다.
3. 상호 운용성
    - 스트림 API 등과의 통합이 자연스럽다.

#### 단점
- 특정 규약을 강제하거나 추가 기능이 필요할 경우 직접 작성해야 한다.

<br>

---

> #### 💡 핵심 정리
> 1. 함수형 인터페이스는 유연한 API 설계에 유용하다.
> 2. 표준 인터페이스(`BiPredicate`, `Function` 등)를 우선적으로 활용하라.
> 3. 필요한 경우에만 새 인터페이스를 작성하되, `@FunctionalInterface`를 명시하라.
> 4. 자바의 함수형 스타일을 적극적으로 활용하여 가독성과 유지보수성을 높이자.
