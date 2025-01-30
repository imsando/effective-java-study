## ⛳️ 목표

- 표준 예외를 사용해야 하는 이유에 대해 알아보자.

<br>

## 📄 핵심 요약

#### **표준예외란?**
- 자바에서 기본적으로 제공하는 예외 클래스들을 의미한다.
- 자바 표준 라이브러리(java.lang 패키지)에 포함되어 있으며, 일반적인 예외 상황을 처리하기 위해 제공된다.

---

### 1. **표준 예외를 재사용해야 하는 이유**
#### 1️⃣ **가독성과 유지보수성 향상**
    - 자바 개발자들에게 익숙한 예외를 사용하면 API 학습 비용이 줄어든다.
    - 코드의 가독성이 향상되며, 유지보수가 쉬워진다.

#### 2️⃣ **중복 코드 및 클래스 감소**
    - 새로운 예외 클래스를 정의할 필요 없이 기존 예외를 활용하여 코드의 복잡성을 줄일 수 있다.
    - 불필요한 예외 클래스를 줄이면 애플리케이션의 메모리 사용량도 감소한다.

#### 3️⃣ **일관된 예외 처리 방식 유지**
    - 표준 예외를 사용하면 동일한 예외 처리 패턴을 유지할 수 있다.
    - 특정 상황에서 어떤 예외가 발생할지 예상하기 쉬워진다.

<br>

### 2. **자주 사용되는 표준 예외 및 활용 사례**

| 예외 이름                         | 사용 사례 |
|----------------------------------|-------------------------------------------------|
| `IllegalArgumentException`      | 메서드 인자로 부적절한 값이 전달되었을 때 |
| `IllegalStateException`         | 객체 상태가 메서드 실행을 허용하지 않을 때 |
| `NullPointerException`          | `null`을 허용하지 않는 메서드에서 `null`이 전달되었을 때 |
| `IndexOutOfBoundsException`     | 인덱스가 허용된 범위를 벗어났을 때 |
| `ConcurrentModificationException` | 허용되지 않은 동시 수정이 감지되었을 때 |
| `UnsupportedOperationException` | 객체가 해당 연산을 지원하지 않을 때 |

<br>

### 3. **표준 예외 사용 예제**

#### ✅ `IllegalArgumentException` 사용 예
```java
public void setAge(int age) {
    if (age < 0) {
        throw new IllegalArgumentException("나이는 0 이상이어야 합니다.");
    }
    this.age = age;
}
```

#### ✅ `IllegalStateException` 사용 예
```java
public void start() {
    if (isRunning) {
        throw new IllegalStateException("이미 시작된 상태입니다.");
    }
    isRunning = true;
}
```

#### ✅ `NullPointerException` 사용 예
```java
public void setName(String name) {
    Objects.requireNonNull(name, "이름은 null이 될 수 없습니다.");
    this.name = name;
}
```

#### ✅ `IndexOutOfBoundsException` 사용 예
```java
public String getItem(int index) {
    if (index < 0 || index >= items.size()) {
        throw new IndexOutOfBoundsException("인덱스 범위를 초과했습니다: " + index);
    }
    return items.get(index);
}
```

#### ✅ `ConcurrentModificationException` 사용 예
```java
Iterator<String> iterator = list.iterator();
while (iterator.hasNext()) {
    list.add("newItem"); // ConcurrentModificationException 발생 가능
}
```

#### ✅ `UnsupportedOperationException` 사용 예
```java
List<String> immutableList = List.of("A", "B", "C");
immutableList.add("D"); // UnsupportedOperationException 발생
```

<br>

### 4. **표준 예외를 선택하는 기준**
#### 1️⃣ **입력값이 잘못된 경우** → `IllegalArgumentException`
#### 2️⃣ **객체 상태가 메서드 실행을 허용하지 않는 경우** → `IllegalStateException`
#### 3️⃣ **`null`이 허용되지 않는 상황에서 `null`이 전달된 경우** → `NullPointerException`
#### 4️⃣ **인덱스 범위를 벗어난 경우** → `IndexOutOfBoundsException`
#### 5️⃣ **동시 수정이 발생할 가능성이 있는 경우** → `ConcurrentModificationException`
#### 6️⃣ **객체가 특정 동작을 지원하지 않는 경우** → `UnsupportedOperationException`

---

### 💡 **핵심 정리**
- **표준 예외를 재사용하면 코드 가독성, 유지보수성, 성능이 향상된다.**
- **예외를 정의하기 전에 표준 예외 중 적절한 것이 있는지 먼저 확인하라.**
- **필요하다면 표준 예외를 확장하여 추가 정보를 제공할 수도 있다.**