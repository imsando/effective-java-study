# 스트림 병렬화는 주의해서 적용하라

## ⛳️ 목표

- 스트림 병렬화를 주의해서 사용해야 하는 이유에 대해 알아보자.

<br>

## 📄 핵심 요약

### **스트림 병렬화의 위험성**

- 잘못된 병렬화는 성능 저하 및 잘못된 결과 초래 가능
- 병렬화 이전에 신중한 성능 테스트 필요

<br>

---

### 1. 스트림 병렬화의 잠재적 문제점

#### 성능 저하 사례
```java
// 메르센 소수 생성 예제 - 병렬화 실패
public static void main(String[] args) {
    primes()
        .parallel() // 병렬화 시도
        .map(p -> TWO.pow(p.intValueExact()).subtract(ONE))
        .filter(mersenne -> mersenne.isProbablePrime(50))
        .limit(20)
        .forEach(System.out::println);
}
```

#### 주요 병렬화 실패 원인
> - 1. 부적절한 데이터
> - 2. 중간 연산의 제한 
> - 3. 종단 연산의 특성

<br>

### 2. 효과적인 병렬 스트림 데이터 소스

#### 병렬화에 적합한 자료구조 종류
> - ArrayList
> - HashMap
> - HashSet
> - ConcurrentHashMap
> - 배열
> - int/Long 범위

#### 자료구조의 핵심 특징
- 정확하고 쉬운 데이터 분할이 가능한 구조이다.
- 뛰어난 참조 지역성을 가진다.
- 메모리 연속성이 좋다.

<br>

### 3. 병렬 스트림 성능 최적화 전략

#### 성능 평가 방법
> 원소 수 * 원소당 연산 코드 줄 수
- 최소 수십만 이상일 때 성능 향상을 기대할 수 있다.

#### 최적의 종단 연산
- 축소(Reduction) 연산에 적합
- `reduce()`, `min()`, `max()`, `count()`, `sum()`
- `anyMatch()`, `allMatch()`, `noneMatch()` 등

<br>

### 4. 병렬 스트림 사용 예시

#### 성공적인 병렬 스트림 예제
```java
// 소수 계산 병렬 스트림
static long pi(long n) {
    return LongStream.rangeClosed(2, n)
        .parallel()
        .mapToObj(BigInteger::valueOf)
        .filter(i -> i.isProbablePrime(50))
        .count();
}
```

#### 난수 생성 시 주의사항
- `SplittableRandom` 사용 권장
- `ThreadLocalRandom` 차선
- `Random` 사용 지양

<br>

### 5. 병렬 스트림 사용 시 주의사항

#### 안전성 고려사항
> - 1. 함수 객체의 side-effect 최소화
> - 2. 상태 비보존(stateless) 연산
> - 3. 결합법칙(associative) 준수
> - 4. 간섭 없는(non-interfering) 연산

#### 성능 테스트 필수
- 실제 운영 환경과 유사한 조건에서 테스트를 진행한다.
- 성능 지표를 면밀히 관찰한다.
- 성능 향상 확실할 때만 적용한다.

<br>

---

> #### 💡 핵심 정리
> 1. 스트림 병렬화는 만능이 아니다.
> 2. 성능 향상 확신 없이는 병렬화 시도하지 말라.
> 3. 병렬 스트림은 신중하게 적용하고 반드시 성능을 검증하라.
> 4. 특정 분야(머신러닝, 대규모 데이터 처리)에서는 유용할 수 있다.