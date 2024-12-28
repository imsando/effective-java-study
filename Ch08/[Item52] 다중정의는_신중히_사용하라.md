# 다중정의는 신중히 사용하라

## ⛳️ 목표

- 다중정의(Overloading) 를 사용할 때, 주의해야할 점들에 대해서 알아보자.

<br>

## 📄 핵심 요약

### 다중정의 (Overloading)
    - 같은 이름의 메서드를 여러 개 정의해야하며 매개변수의 수나 타입이 달라야 한다.
    - 컴파일타임에 호출할 메서드가 결정된다.
    - 매개변수의 **컴파일타임 타입**에 따라 메서드가 선택된다.

### 재정의 (Overriding)
    - 상위 클래스의 메서드를 하위 클래스에서 다시 정의한다.
    - 런타임에 호출할 메서드가 결정된다.
    - 객체의 **런타임 타입**에 따라 메서드가 선택된다.

---

### 1. **다중정의의 혼란을 피하는 방법**

#### 1️⃣ 혼란이 발생하는 이유
- 다중정의는 매개변수의 컴파일타임 타입에 따라 정적으로 선택된다.
- 호출 시점의 런타임 타입은 다중정의 선택에 영향을 미치지 않는다.
- 예상과 다르게 동작할 가능성이 크다.

#### 2️⃣ 해결책
- `instanceof`와 명시적 검사로 동작을 명확히 한다.
  ```java
  public static String classify(Collection<?> c) {
      return c instanceof Set ? "집합" :
             c instanceof List ? "리스트" : "그 외";
  }
  ```

<br>
<br>

### 2. **다중정의의 안전한 사용**

#### 1️⃣ 매개변수 수가 같다면 다중정의 피하기
- 가변인수(varargs)를 사용하는 메서드는 다중정의를 피해야한다.
- 메서드 이름을 다르게 지어 혼란을 방지한다.
  ```java
  void writeInt(int value);
  void writeBoolean(boolean value);
  ```

#### 2️⃣ 안전한 다중정의 조건
- 매개변수 타입이 근본적으로 다르다면 안전하다.
    - `int`와 `String`처럼 명확히 다른 타입
    - 예 : `List`의 `remove(int index)`와 `remove(Object o)`는 문제가 될 수 있다.
  ```java
  list.remove((Integer) 1);  // 명확히 Integer로 변환
  ```

<br>
<br>

### 3. **람다와 메서드 참조에서의 다중정의**

#### 1️⃣ 함수형 인터페이스와 혼란
- 함수형 인터페이스를 인수로 받을 때, 다중정의가 혼란을 초래할 수 있다.
  ```java
  ExecutorService exec = Executors.newCachedThreadPool();
  exec.submit(System.out::println); // Callable과 Runnable 혼동
  ```

#### 2️⃣ 해결책
- 함수형 인터페이스를 다중정의하지 않는다.
- `-XLint:overloads` 컴파일 옵션으로 경고를 활성화한다.

<br>
<br>

### 4. **실제 사례 분석**

#### 1️⃣ 혼란을 일으키는 예
- `List`의 `remove` 메서드
    - `remove(int)`는 인덱스를 제거한다.
    - `remove(Object)`는 객체를 제거한다.

#### 2️⃣ 혼란을 피하는 예
- `ObjectOutputStream`의 `write` 메서드
    - 모든 메서드에 다른 이름을 사용한다.
    - 예 : `writeInt`, `writeBoolean`

<br>
<br>

### 5. **다중정의와 생성자**

- 생성자는 이름을 다르게 지을 수 없으므로 다중정의가 강제로 사용된다.
- 해결책
>    - 정적 팩터리 메서드를 사용해 다중정의 회피한다.
>  ```java
>  public static Thermometer ofCelsius(double value);
>  public static Thermometer ofFahrenheit(double value);
>  ```

<br>
<br>

> 💡 **핵심 정리**
>
> 1. 매개변수 타입이 명확히 다르지 않다면 다중정의를 피하라.
> 2. 가변인수를 사용하는 메서드는 다중정의하지 말라.
> 3. 함수형 인터페이스를 인수로 받는 경우 다중정의를 피하라.
> 4. 필요 시 명시적 타입 변환으로 혼란을 방지하라.
> 5. 가능한 경우 메서드 이름을 다르게 지어라.