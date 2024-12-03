# 명명 패턴보다 애너테이션을 사용하라

## ⛳️ 목표

- 명명 패턴보다 애너테이션을 사용해야하는 이유에 대해 알아보자.

<br>

## 📄 핵심 요약

### **명명 패턴이란?**

- 코드 요소(클래스, 메서드, 변수 등)의 이름을 정하는 데 사용하는 규칙을 의미한다.
- 자바에서 명명 패턴은 표준화된 코드 스타일을 유지하거나 특정 도구나 프레임워크가 코드 요소를 식별할 때 사용된다.

#### **명명 패턴의 문제점**
1. **오타로 인한 오류**
    - 테스트 메서드 이름에 오타가 있으면 무시되거나 잘못 실행된다.
2. **제한된 적용 범위**
    - 프로그램 요소에 대한 적합성을 검증할 수 없다.
3. **매개변수 전달의 한계**
    - 문자열로 값을 전달하는 방식은 가독성이 떨어지고 오류에 취약하다.

---

### 1. 마커 애너테이션 설계

- 테스트 메서드를 식별하는 간단한 애너테이션 설계 방식이다.
- 런타임 유지 및 메서드 전용으로 제한된다.

예시코드
```java
import java.lang.annotation.*;

@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface Test {
}
```

---

### 2. 테스트 러너 설계

- 애너테이션을 활용한 테스트 메서드 실행한다.
- 리플렉션을 통해 애너테이션이 붙은 메서드만 실행한다.

예시코드
```java
import java.lang.reflect.Method;

public class RunTests {
    public static void main(String[] args) throws Exception {
        int tests = 0;
        int passed = 0;
        Class<?> testClass = Class.forName(args[0]);

        for (Method m : testClass.getDeclaredMethods()) {
            if (m.isAnnotationPresent(Test.class)) {
                tests++;
                try {
                    m.invoke(null);
                    passed++;
                } catch (Throwable t) {
                    System.out.println(m + " 실패: " + t.getCause());
                }
            }
        }
        System.out.printf("성공: %d, 실패: %d%n", passed, tests - passed);
    }
}
```

---

### 3. 매개변수를 가진 애너테이션

- 특정 예외를 던져야 성공하는 테스트를 구현한다.
- 애너테이션에 매개변수를 추가하여 더 풍부한 테스트 케이스 지원한다.

예시코드
```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTest {
    Class<? extends Throwable> value();
}
```

---

### 4. 매개변수를 활용한 테스트 러너

- 애너테이션에서 매개변수를 추출해 올바른 예외를 검증한다.
- 리플렉션으로 테스트 메서드 실행 중 발생한 예외를 처리한다.

예시코드
```java
public class RunTests {
    public static void main(String[] args) throws Exception {
        int tests = 0;
        int passed = 0;
        Class<?> testClass = Class.forName(args[0]);

        for (Method m : testClass.getDeclaredMethods()) {
            if (m.isAnnotationPresent(ExceptionTest.class)) {
                tests++;
                try {
                    m.invoke(null);
                    System.out.printf("테스트 실패: 예외를 던지지 않음%n", m);
                } catch (Throwable t) {
                    Throwable exc = t.getCause();
                    Class<? extends Throwable> excType = m.getAnnotation(ExceptionTest.class).value();
                    if (excType.isInstance(exc)) {
                        passed++;
                    } else {
                        System.out.printf("테스트 실패: 기대 예외 %s, 발생 예외 %s%n", excType.getName(), exc);
                    }
                }
            }
        }
        System.out.printf("성공: %d, 실패: %d%n", passed, tests - passed);
    }
}
```

---

### 5. 배열 매개변수를 활용한 확장

- 여러 예외를 명시하고, 그중 하나라도 발생하면 성공하도록 확장한다.

Class<? extends Throwable> 배열 매개변수
```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTest {
    Class<? extends Throwable>[] value();
}
```

실제사용코드
```java
@ExceptionTest({IndexOutOfBoundsException.class, NullPointerException.class})
public static void doublyBad() {
    List<String> list = new ArrayList<>();
    list.addAll(5, null); // IndexOutOfBoundsException 또는 NullPointerException 발생
}
```

---

### 6. 반복 가능한 애너테이션

- `@Repeatable` 메타애너테이션을 사용하여 동일한 애너테이션을 여러 번 적용 가능하도록 확장한다.

예시코드
```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
@Repeatable(ExceptionTestContainer.class)
public @interface ExceptionTest {
    Class<? extends Throwable> value();
}

@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface ExceptionTestContainer {
    ExceptionTest[] value();
}
```

실제사용코드
```java
@ExceptionTest(IndexOutOfBoundsException.class)
@ExceptionTest(NullPointerException.class)
public static void doublyBad() {
    List<String> list = new ArrayList<>();
    list.addAll(5, null);
}
```

---

> 💡 **핵심 정리**
>
> 1. 애너테이션은 명명 패턴의 문제를 해결하며, 더 강력하고 유지보수하기 쉬운 설계를 지원한다.
> 2. 매개변수와 배열 매개변수를 활용하면 더 다양한 테스트 케이스를 지원할 수 있다.
> 3. 반복 가능한 애너테이션은 코드 가독성과 유지보수성을 높인다.
> 4. 애너테이션을 통해 테스트를 설계하면 명확성과 확장성이 크게 향상된다.