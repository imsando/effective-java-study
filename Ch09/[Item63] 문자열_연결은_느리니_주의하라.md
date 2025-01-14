# 문자열 연결은 느리니 주의하라

## ⛳️ 목표

- 문자열 연결 연산자의 성능 한계를 이해하자.

<br>

## 📄 핵심 요약

---

### **문자열 연결 연산자의 문제점**

1. **성능 저하**
    - 문자열 연결 연산자(+)는 n개의 문자열을 연결할 때 시간 복잡도가 \(O(n^2)\)에 비례한다.
    - 문자열은 불변(Immutable) 객체이기 때문에 새로운 문자열을 생성할 때 기존 문자열을 모두 복사해야 한다.

2. **실제 사례**
    - 다음은 잘못된 문자열 연결의 예로 많은 문자열을 연결하면 성능 문제가 발생할 수 있다.

   ```java
   public String statement() {
       String result = "";
       for (int i = 0; i < numItems(); i++) {
           result += lineForItem(i); // 문자열 연결
       }
       return result;
   }
   ```

    - 위 코드는 문자열의 크기와 품목 수가 증가할수록 성능이 기하급수적으로 저하된다.

---

### **StringBuilder로 성능 개선**

1. **StringBuilder 사용 이유**
    - StringBuilder는 내부적으로 가변 배열을 사용해 문자열 연결의 복사 작업을 줄인다.
    - 시간 복잡도가 \(O(n)\)으로 개선되므로 성능 차이가 크다.

2. **개선된 코드**
   ```java
   public String statement() {
       StringBuilder builder = new StringBuilder(numItems() * LINE_WIDTH);
       for (int i = 0; i < numItems(); i++) {
           builder.append(lineForItem(i));
       }
       return builder.toString();
   }
   ```

3. **실험 결과**
    - 품목 수가 100개, 각 품목 문자열 길이가 80인 경우 StringBuilder를 사용하면 최대 6.5배 빠른 성능을 보인다.
    - 기본 크기 초기화 없이 사용해도 5.5배의 성능 향상을 얻을 수 있다.

---

### **문자열 연결 시 주의사항**

1. **작은 문자열**
    - 한 줄 출력이나 크기가 작은 문자열 연결에는 문자열 연결 연산자(+)를 사용해도 성능 문제가 없다.

2. **대규모 문자열 처리**
    - 대규모 데이터나 반복 작업에서는 반드시 StringBuilder를 사용하는 것이 좋다.
    - 초기 크기를 설정하면 추가적인 성능 최적화를 기대할 수 있다.

3. **대안**
    - StringBuilder 외에도 문자 배열을 사용하거나 문자열을 하나씩 처리하는 방법이 있다.

---

### 💡 **핵심 정리**

- 많은 문자열을 연결할 때는 문자열 연결 연산자(+) 대신 StringBuilder를 사용하라.
- 반복적인 연결 작업이 필요한 경우 성능 차이는 더욱 커진다.