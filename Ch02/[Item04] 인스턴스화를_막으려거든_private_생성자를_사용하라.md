# private 사용으로 인스턴스화를 막라

## ⛳️ 목표

- **유틸리티 클래스**를 만들 때 **private 생성자**를 사용하여 **인스턴스화 방지**하는 방법을 이해하자

<br>

## 📄 핵심 요약

***유틸리티 클래스***

- 정적 메서드만을 모아둔 클래스를 말하며 인스턴스로 생성할 필요가 없는 클래스이다
- 인스턴스화를 막기 위해 **private 생성자**를 사용한다

    ```java
        public class MathUtils {

          // private 생성자를 통해 인스턴스화를 방지
          private MathUtils() {
              // 이 클래스는 인스턴스화될 수 없습니다.
              throw new UnsupportedOperationException("This class cannot be instantiated");
          }

          // 두 수의 합을 구하는 정적 메서드
          public static int add(int a, int b) {
              return a + b;
          }
        }
    ```

***

### 1. 인스턴스화를 방지하는 이유

- 유틸리티 클래스는 그 자체로 인스턴스를 만들 필요가 없다
- 예를 들어 **Math** 클래스처럼 정적 메서드만 모아둔 클래스는 객체로 만들지 않고 메서드만 호출하는 것이 목적이다

***

### 2. 구현방식: private 생성자 사용

- 생성자를 `private`으로 선언하면 해당 클래스는 외부에서 인스턴스화할 수 없다
- 컴파일러가 기본 생성자를 자동으로 추가하지 않으므로 인스턴스 생성을 확실하게 막을 수 있다

<br>

#### 코드 예시:

```java
public class UtilityClass {

    // 인스턴스화를 막기 위해 private 생성자를 사용
    private UtilityClass() {
        throw new AssertionError("Cannot instantiate UtilityClass");
    }

    // 유틸리티 메서드들
    public static int add(int a, int b) {
        return a + b;
    }
    
    public static int subtract(int a, int b) {
        return a - b;
    }
}

// 사용 예시
int result = UtilityClass.add(3, 5); // 인스턴스 없이 호출 가능
```

***설명:***

- `private UtilityClass()` 생성자는 클래스 바깥에서 호출할 수 없으며 만약 내부에서 실수로 생성자를 호출하면 `AssertionError`가 발생한다
- 이렇게 하면 **인스턴스화가 절대 불가능**하고 클래스는 정적 메서드만으로 사용된다

***

### 3. 유틸리티 클래스의 특징

- 상속 불가 : private 생성자로 인해 해당 클래스를 상속하여 사용할 수 없다
- 객체 생성 차단 : 잘못된 객체 생성이나 오용을 방지할 수 있다

***

### 4. 주의사항

- 이 방식은 직관적이지 않으므로 주석을 달아 인스턴스화를 막는 이유를 명시하는 것이 좋다
- 단순히 **정적 메서드**만 제공하는 클래스일 때 사용하면 적합하다