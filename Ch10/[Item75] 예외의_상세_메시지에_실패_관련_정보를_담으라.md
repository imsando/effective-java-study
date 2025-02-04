## ⛳️ 목표

- 예외의 상세 메시지에 실패 관련 정보를 담는 방법에 대해 알아보자.

<br>

## 📄 핵심 요약

### **예외 메시지를 상세하게 작성해야하는 이유?**
- 예외 메시지는 디버깅 및 오류 분석에 있어 중요한 역할을 한다.
- 특히, 실패 원인을 재현하기 어려운 경우 예외 메시지가 유일한 단서가 될 수 있다.
- 예외 메시지에는 관련된 모든 매개변수와 필드 값을 포함해야 한다.

---

### 1. **예외 메시지 작성 원칙**
#### 1️⃣ **실패 원인과 관련된 정보를 최대한 포함할 것**
- 예 : `IndexOutOfBoundsException`의 경우 최소/최대 범위와 잘못된 인덱스 값을 포함해야 한다.
- 예외 발생 시점의 상태를 포착하여 문제 해결을 돕는다.

#### 2️⃣ **불필요한 정보는 제외할 것**
- 스택 추적에 포함된 정보(파일명, 줄 번호 등)는 중복할 필요가 없다.
- 보안적으로 민감한 정보(비밀번호, API 키 등)는 포함해서는 안 된다.

#### 3️⃣ **예외 메시지와 사용자 메시지를 혼동하지 말 것**
- 예외 메시지는 개발자를 위한 것이므로 기술적인 정보를 포함해야 한다.
- 사용자에게는 친절하고 이해하기 쉬운 메시지를 별도로 제공해야 한다.

<br>

### 2. **예외 클래스 개선 방법**

#### ✅ 개선 전 (기본 생성자 사용)
```java
throw new IndexOutOfBoundsException("잘못된 인덱스 접근");
```
- 단순한 메시지로 인해 디버깅이 어려울 수 있다.

#### ✅ 개선 후 (추가 정보를 포함한 생성자)
```java
public class CustomIndexOutOfBoundsException extends IndexOutOfBoundsException {
    private final int lowerBound;
    private final int upperBound;
    private final int index;

    public CustomIndexOutOfBoundsException(int lowerBound, int upperBound, int index) {
        super(String.format("최솟값: %d, 최댓값: %d, 인덱스: %d", lowerBound, upperBound, index));
        this.lowerBound = lowerBound;
        this.upperBound = upperBound;
        this.index = index;
    }

    public int getLowerBound() { return lowerBound; }
    public int getUpperBound() { return upperBound; }
    public int getIndex() { return index; }
}
```
- 예외 메시지에 실패 관련 정보를 포함하여 디버깅이 용이해진다.
- 접근자 메서드 제공으로 예외 정보를 프로그래밍적으로 활용 가능하다.

<br>

### 3. **검사 예외 vs 비검사 예외 활용**
| 예외 유형 | 특징 | 사용 예시 |
|-----------|----------------|----------------|
| **검사 예외 (Checked Exception)** | 반드시 `try-catch`로 처리하거나 `throws`로 전달해야 함. | 파일 입출력 오류, 네트워크 실패 등 |
| **비검사 예외 (Unchecked Exception)** | `RuntimeException`을 상속받아 `try-catch` 없이 호출자에게 전파됨. | `NullPointerException`, `IndexOutOfBoundsException` 등 프로그래밍 오류 |

<br>

### 💡 **핵심 정리**
- 예외 메시지에는 발생 원인을 정확히 담아야 한다.
- 예외 클래스 내부에서 고품질의 상세 메시지를 생성하는 것이 좋다.
- 검사 예외는 복구 가능성이 있을 때, 비검사 예외는 프로그래밍 오류 발생 시 사용하라.