## ⛳️ 목표
- 지연 초기화를 신중히 해야 하는 이유에 대해 알아보자.

<br>  

## 📄 핵심 요약

#### **지연 초기화(Lazy Initialization)란?**
- 필드의 초기화를 **그 값이 실제로 필요할 때까지 늦추는 기법**이다.
- 불필요한 초기화를 방지하여 **성능을 최적화**할 수 있지만, **필드 접근 비용이 증가**할 수 있다.
- 정적(static) 필드와 인스턴스 필드 모두에 적용할 수 있다.

#### **지연 초기화를 신중히 해야 하는 이유**
- **초기화 비용과 접근 빈도를 고려해야 한다.**
    - 초기화 비용이 크고 사용 빈도가 낮다면 유용하지만, 자주 사용되는 필드라면 오히려 성능이 저하될 수 있다.
- **멀티스레드 환경에서는 동기화가 필요하다.**
    - 동기화를 올바르게 처리하지 않으면 **데이터 불일치 및 심각한 버그**가 발생할 수 있다.
- **일반적인 초기화가 더 효율적인 경우가 많다.**
    - 대부분의 필드는 지연 초기화 없이 **즉시 초기화하는 것이 더 안정적이고 빠르다.**

---
<br>  

### 1. **일반적인 필드 초기화 방식**
```java
private final FieldType field = computeFieldValue();
```
- **final 필드를 사용**하여 객체 생성 시 즉시 초기화하는 방법이다.
- 대부분의 경우 **가장 단순하고 효과적인 방식**이다.

<br>  

### 2. **인스턴스 필드 지연 초기화 방법**
#### ✅ `synchronized` 접근자 사용 (기본적인 방법)
```java
private FieldType field;

private synchronized FieldType getField() {
    if (field == null) {
        field = computeFieldValue();
    }
    return field;
}
```
- 동기화된 접근자(synchronized getter)를 사용하여 필드를 초기화한다.
- **초기화 순환 문제를 해결**할 때 유용하다.

#### ✅ 이중 검사(Double-Check) 방식 (성능 최적화)
```java
private volatile FieldType field;

private FieldType getField() {
    FieldType result = field;
    if (result != null) {
        return result;
    }
    synchronized (this) {
        if (field == null) {
            field = computeFieldValue();
        }
        return field;
    }
}
```
- 첫 번째 검사에서는 동기화를 사용하지 않아 성능을 향상시킨다.
- `volatile` 키워드를 사용하여 JVM의 명령어 재정렬을 방지해야 한다.
- 멀티스레드 환경에서 안전한 고성능 지연 초기화 방법이다.

<br>  

### 3. **정적 필드 지연 초기화 방법**
#### ✅ 정적 필드용 지연 초기화 홀더 클래스 (Lazy Initialization Holder)
```java
private static class FieldHolder {
    static final FieldType field = computeFieldValue();
}

private static FieldType getField() {
    return FieldHolder.field;
}
```
- 클래스가 처음 사용될 때 초기화되므로 동기화 없이도 안전하다.
- 일반적인 정적 필드 지연 초기화 방법 중 가장 효율적이다.

<br>  

### 4. **반복 초기화가 허용되는 경우 (단일 검사 방식)**
```java
private volatile FieldType field;

private FieldType getField() {
    FieldType result = field;
    if (result == null) {
        field = result = computeFieldValue();
    }
    return result;
}
```
- 필드가 여러 번 초기화될 가능성이 있지만, 성능이 중요할 때 사용 가능하다.
- 모든 스레드가 필드를 다시 계산해도 무방할 때 적합하다.

<br>  

### 💡 **핵심 정리**
- 대부분의 필드는 즉시 초기화하는 것이 더 좋다.
- 초기화 비용이 크거나, 초기화 순환 문제를 해결해야 할 때만 지연 초기화를 고려하라.
- 멀티스레드 환경에서는 반드시 동기화를 신경 써야 한다.
- 인스턴스 필드는 이중 검사(Double-Check) 기법을, 정적 필드는 지연 초기화 홀더 클래스를 사용하라.
- 반복 초기화가 허용된다면 단일 검사(single-check) 기법도 고려할 수 있다.