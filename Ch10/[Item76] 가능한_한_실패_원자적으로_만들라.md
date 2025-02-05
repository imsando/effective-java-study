## ⛳️ 목표

- 실패 원자성에 대해 알아보자.

<br>

## 📄 핵심 요약

#### **실패 원자성이란?**
- 메서드가 실패하더라도 객체의 상태가 메서드 호출 이전과 동일하게 유지되는 성질을 의미한다.
- 예외가 발생해도 객체가 정상적으로 사용 가능한 상태여야 한다.
- 주어진 입력이 유효하지 않을 경우, 예외를 발생시키되 객체의 일관성이 깨지지 않아야 한다.

---

### 1. **실패 원자성을 달성하는 방법**

#### 1️⃣ **불변 객체(Immutable Object) 사용**
- 불변 객체는 한 번 생성되면 내부 상태가 변경되지 않으므로 본질적으로 실패 원자성을 가진다.
- 메서드가 실패하더라도 기존 객체의 상태는 유지된다.

#### 2️⃣ **작업 수행 전 입력값 유효성 검사**
- 객체 상태를 변경하기 전에 입력값을 검증하여 예외 가능성을 사전에 방지한다.
- 잘못된 입력이 들어오면 즉시 예외를 던져 객체가 변경되지 않도록 한다.
  ```java
  public void setAge(int age) {
      if (age < 0) {
          throw new IllegalArgumentException("나이는 0 이상이어야 합니다.");
      }
      this.age = age;
  }
  ```

#### 3️⃣ **객체의 상태 변경을 최종 단계에서 수행**
- 실패할 가능성이 있는 모든 연산을 수행한 후, 마지막에 객체의 상태를 변경한다.
- 중간 연산 중 예외가 발생하면 객체 상태가 변경되지 않는다.
  ```java
  public Object pop() {
      if (size == 0) {
          throw new EmptyStackException();
      }
      Object result = elements[size - 1];
      elements[size - 1] = null; // 다 쓴 참조 해제
      size--;
      return result;
  }
  ```

#### 4️⃣ **임시 복사본에서 작업 수행 후 원본 객체에 반영**
- 데이터의 임시 복사본에서 작업을 수행한 후, 성공적으로 완료되면 원본 객체와 교체한다.
- 정렬과 같은 연산에서 많이 활용된다.
  ```java
  public List<Integer> sort(List<Integer> list) {
      List<Integer> tempList = new ArrayList<>(list); // 원본 복사
      Collections.sort(tempList); // 정렬 수행
      return tempList; // 정렬된 리스트 반환
  }
  ```

#### 5️⃣ **예외 발생 시 복구 코드 작성**
- 예외 발생 시 객체 상태를 원래대로 복구하는 코드(rollback)를 작성하여 일관성을 유지한다.
- 트랜잭션과 같은 시스템에서 사용된다.
  ```java
  public void transfer(Account from, Account to, int amount) {
      int originalBalance = from.getBalance();
      try {
          from.withdraw(amount);
          to.deposit(amount);
      } catch (Exception e) {
          from.setBalance(originalBalance); // 원래 상태로 복구
          throw e;
      }
  }
  ```
    
<br>

### 2. **실패 원자성이 항상 필요한 것은 아니다**
- 동기화 없이 여러 스레드가 같은 객체를 수정하면 일관성이 깨질 수 있다.
- `ConcurrentModificationException`이 발생한 경우 객체가 정상 상태라고 단정할 수 없다.
- `Error`는 복구할 수 없으므로 `AssertionError` 등에 대해 실패 원자성을 보장할 필요가 없다.
- 성능이나 복잡도 문제로 인해 실패 원자성을 보장하는 것이 어렵다면 명세에 실패 시 객체 상태를 명확히 문서화해야 한다.

---

### 💡 **핵심 정리**
- 메서드가 실패하더라도 객체의 상태를 이전 상태로 유지해야 한다.
- 불변 객체를 활용하면 기본적으로 실패 원자성을 보장할 수 있다.
- 객체 변경을 수행하기 전 입력값 검증 및 예외 발생 가능성이 있는 연산을 먼저 수행해야 한다.
- 복사본을 활용하거나 예외 발생 시 복구하는 방식을 고려할 수 있다.

