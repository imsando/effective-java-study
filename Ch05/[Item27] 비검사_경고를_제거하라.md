# 비검사 경고를 제거하라

## ⛳️ 목표

- 비검사 경고에 대해 깊이 이해하고 안전하게 경고를 제거하는 방법을 익힌다

<br>

## 📄 핵심 요약

***비검사 경고란?***

- 비검사 경고는 제네릭 타입 사용 시 **컴파일러가 타입 안전성을 보장할 수 없을 때 발생**하는 경고이다
- 제네릭의 목적은 타입 안전성을 높이는 것이기 때문에 비검사 경고는 가급적 제거하는 것이 좋다

<br>

### 1. 비검사 경고가 발생하는 이유

> - **⓵ 비검사 형변환 경고**: 제네릭 타입을 사용하지 않고 형변환을 하면 타입 안전성을 보장할 수 없어 경고가 발생한다
> - **② 비검사 메서드 호출 경고**: 제네릭 타입을 사용하지 않은 메서드를 호출할 때 발생하는 경고이다
> - **⓷ 비검사 매개변수 경고**: 제네릭 타입을 사용하지 않고 매개변수를 전달할 때 발생한다

### 2. 경고 억제 방법

> - @SuppressWarnings("unchecked")` 애너테이션을 사용해 비검사 경고를 억제할 수 있다
> - **경고를 억제할 때는 주의가 필요**하며 타입 안전성을 검증한 경우에만 사용해야 한다


---

### [비검사 경고 억제 예시]

> - `@SuppressWarnings("unchecked")` 애너테이션을 사용할 때는
> - 타입 안전성을 검증한 후 가능한 한 **좁은 범위**에 적용해야 한다
> - 예를 들어 메서드 전체보다는 **지역 변수**에 적용하는 것이 좋다

```java
import java.util.Arrays;

public class ArrayUtils {
    @SuppressWarnings("unchecked") // 안전한 형변환임을 검증 후에만 사용
    public static <T> T[] toArray(T[] a, Object[] elements) {
        if (a.length < elements.length) {
            // 생성한 배열과 매개변수 배열의 타입이 동일하므로 올바른 형변환
            @SuppressWarnings("unchecked")
            T[] result = (T[]) Arrays.copyOf(elements, elements.length, a.getClass());
            return result;
        }
        System.arraycopy(elements, 0, a, 0, elements.length);
        return a;
    }
}
```
---
### 3. 주의사항

1. **애너테이션 적용 범위**: 가능한 한 좁은 범위에 `@SuppressWarnings`를 적용해야한다. 메서드 전체에 적용하면 중요한 경고를 놓칠 수 있다.
2. **주석 작성**: `@SuppressWarnings` 애너테이션을 사용할 때는 **경고를 무시해도 안전한 이유**를 주석으로 남겨 다른 개발자에게 타입 안전성에 대해 명확히 전달해야 한다.

<br>

> **💡핵심 정리**  
> 1. 비검사 경고는 **런타임 `ClassCastException`의 잠재적 원인**이므로 가급적 제거하자
> 2. 경고를 억제해야 하는 경우 코드가 타입 안전함을 검증하고 `@SuppressWarnings("unchecked")` 애너테이션을 **가능한 한 좁은 범위**에 적용한다