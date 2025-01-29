## ⛳️ 목표

- 검사 예외와 비검사 예외의 적절한 사용상황에 대해 알아보자.

<br>

## 📄 핵심 요약


#### **검사 예외 (Checked Exception)**
    - 반드시 `try-catch` 문으로 처리하거나 `throws` 키워드를 사용해 호출자에게 전파해야 한다.
    - 예외 상황에서 복구할 가능성이 있는 경우 적절하다.
    - 하지만 남용하면 API 사용성이 저하되고, 스트림 등과 함께 사용하기 어려워진다.

#### **비검사 예외 (Unchecked Exception)**
    - `RuntimeException`을 상속받으며 `try-catch` 없이도 호출자에게 전파된다.
    - 복구할 방법이 없거나, 프로그래밍 오류(예: `NullPointerException`, `IndexOutOfBoundsException`)인 경우 적절하다.

<br>

### 1. **검사 예외를 피하는 방법**

#### 1️⃣ 옵셔널 반환 (`Optional<T>`)
- 예외를 던지지 않고 빈 옵셔널을 반환하여 예외를 처리하도록 유도할 수 있다.
- 하지만, 예외 원인에 대한 추가 정보를 제공하기 어렵다.

```java
public Optional<Result> findResult(String key) {
    return Optional.ofNullable(cache.get(key));
}
```

#### 2️⃣ 상태 검사 메서드 활용
- 예외를 던지는 대신, 먼저 상태 검사 메서드를 호출하여 예외 발생 여부를 확인할 수 있다.
- 하지만 멀티스레드 환경에서는 동기화 문제가 발생할 수 있다.

```java
if (obj.isActionPermitted(args)) {
    obj.performAction(args); // 예외 없이 실행 가능
} else {
    // 예외 상황 처리
}
```

<br>

### 2. **검사 예외 사용상황**

#### 1️⃣ API 사용자가 예외를 복구할 수 있을 때
    - 파일이 존재하지 않을 때 새로운 파일을 생성하는 경우
    - 네트워크 요청 실패 시 재시도 가능성이 있는 경우

#### 2️⃣ 예외 발생 이유를 명확하게 제공해야 할 때
    - 예외 타입과 메시지를 통해 디버깅 및 오류 처리를 쉽게 만들 수 있다.

#### 3️⃣ 대안이 없는 경우
    - 옵셔널이나 상태 검사 메서드로 처리할 수 없다면 검사 예외를 사용하는 것이 적절할 수 있다.

<br>

### 3. **비검사 예외 사용상황**

1️⃣ **복구할 방법이 없는 경우**
    - `NullPointerException`, `IndexOutOfBoundsException`처럼 프로그래밍 오류에 해당하는 경우

2️⃣ **API 사용성이 저하되는 경우**
    - 불필요한 `try-catch` 남용을 초래하는 경우

<br>

### 4. **리팩터링 예제**

#### ✅ 리팩터링 전 (검사 예외 사용)
```java
try {
    obj.action(args);
} catch (TheCheckedException e) {
    // 예외 상황에 대처한다.
}
```

#### ✅ 리팩터링 후 (상태 검사 메서드 활용)
```java
if (obj.isActionPermitted(args)) {
    obj.action(args);
} else {
    // 예외 상황에 대처한다.
}
```

---

### 💡 **핵심 정리**

- 복구 가능성이 없거나 예외 처리가 부담스러우면 **비검사 예외**를 사용하라.
- 복구 가능성이 있는 경우라도 **옵셔널 반환**이나 **상태 검사 메서드**를 고려하라.
- 검사 예외를 사용할 때는 API 사용성을 고려하여 최소한으로 사용하라.