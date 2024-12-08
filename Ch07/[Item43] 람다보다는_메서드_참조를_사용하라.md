# 람다보다는 메서드 참조를 사용하라

## ⛳️ 목표

- 람다 표현식보다 메서드 참조 사용을 권장하는 이유에 대해 알아보자.

<br>

## 📄 핵심 요약

### **람다와 메서드 참조**

- 람다 표현식 : 익명 함수로 간결한 함수 객체를 생성한다.
- 메서드 참조 : 특정 메서드 또는 생성자를 직접 참조하여 함수 객체를 생성한다.

#### **람다 표현식의 문제점**
1. 매개변수 과잉
    - 단순한 연산에 불필요한 매개변수 이름이 필요하다.
2. 가독성 저하
    - 람다의 길이가 늘어나면 코드를 이해하기 어려워진다.

---

### 1. 람다에서 메서드 참조로 변환

#### **예제코드 : `Map.merge`와 메서드 참조**
```java
map.merge(key, 1, (count, incr) -> count + incr); // 람다
map.merge(key, 1, Integer::sum);                 // 메서드 참조
```

- `Integer::sum`을 사용하면 매개변수 이름을 제거해 코드가 간결해진다.

#### **람다로 할 수 없는 일**
- 메서드 참조 역시 람다로 할 수 없는 일을 할 수는 없다.
- 다만, 람다식이 복잡하고 긴 경우 메서드 참조가 더 적합하다.

<br>

### 2. 메서드 참조 유형

#### **메서드 참조 유형별 정리**
| 유형            | 예시                   | 같은 기능의 람다            |
|----------------|-----------------------|------------------------|
| 정적 메서드 참조    | `Integer::parseInt`   | `str -> Integer.parseInt(str)` |
| 한정적 인스턴스 메서드 참조 | `Instant.now()::isAfter` | `t -> then.isAfter(t)` |
| 비한정적 인스턴스 메서드 참조 | `String::toLowerCase`   | `str -> str.toLowerCase()` |
| 클래스 생성자 참조    | `TreeMap<K,V>::new`    | `() -> new TreeMap<K,V>()` |
| 배열 생성자 참조     | `int[]::new`          | `len -> new int[len]` |

<br>

### 3. 메서드 참조의 활용과 한계

#### **활용**
- 메서드 참조는 함수 이름을 그대로 사용할 수 있어 명확하다.
- 주로 **스트림 API**나 **함수형 인터페이스**를 사용할 때 자주 사용된다.

#### **한계**
- 람다가 메서드 참조보다 더 간결한 경우도 있다.
  ```java
  service.execute(() -> action());         // 람다
  service.execute(GoshThisClassNameIsHumongous::action); // 메서드 참조
  ```
    - 람다가 짧고 명확하면 람다를 사용하는 것이 좋다.

<br>

### 4. 메서드 참조의 예제

#### **스트림에서의 활용**
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

### 5. 제네릭 함수 타입과 메서드 참조

#### **제네릭 람다 불가능 사례**
- 람다로는 불가능하지만 메서드 참조로 구현 가능한 유일한 예는 **제네릭 함수 타입** 이다.
  ```java
  interface G1 { <E extends Exception> Object m() throws E; }
  interface G2 { <F extends Exception> String m() throws Exception; }
  interface G extends G1, G2 {}
  ```

<br>

> 💡 **핵심 정리**
>
> 1. 메서드 참조는 람다보다 간결하고 명확한 대안이 될 수 있다.
> 2. 람다로 작성한 코드가 복잡하거나 길다면 메서드 참조를 고려하라.
> 3. 람다가 더 간결한 경우에는 람다를 사용하라.
> 4. 메서드 참조는 코드의 가독성과 유지보수성을 높이는 데 효과적이다.