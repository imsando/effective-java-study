# 매개변수가 유효한지 검사하라

## ⛳️ 목표

메서드 구현부가 실행되기 전 매개변수 검사를 시행해야 함을 이해하자

<br>

## 📄 핵심 요약

- 매개변수의 유효성 검증이 이루어지지 않았을 때 개발자의 의도와 다른 버그가 발생할 수 있다.
- public, protected 메서드는 매개변수 관련 예외를 문서화 해야 한다.
- public이 아닌 메서드에는 단언문을 활용해 매개변수 유효성 검증을 할 수 있다.
- Objects를 통해 null 검증, 예외 메시지 지정, 범위 검사 기능을 활용할 수 있다.

---

## 매개변수 검사의 중요성

매개변수 검사를 제대로 하지 못했을 때 발생할 수 있는 몇 가지 문제점을 살펴보자.

- 메서드가 수행되는 중간에 모호한 예외를 던지며 실패한다.
- 메서드가 문제없이 수행되지만 잘못된 결과를 반환한다.
- 메서드가 문제없이 수행되지만 어떤 객체를 의도와 다른 형태로 생성되어 알 수 없는 시점에 관련 없는 오류가 발생한다.

<br>

## 매개변수 검사 방법

1. public과 protected 메서드는 매개변수 값이 잘못됐을 때 던지는 예외를 문서화 해야 한다.

```java
/**
* .. 생략
* @throws ArithmeticException m이 0보다 작거나 같으면 발생한다.
/*
public BigInteger mod(BigInteger m) {
	if (m.signum() <= 0) {
		throws new ArithmeticException("계수(m)은 양수여야 합니다. " + m);
	}
	// 생략
}
```

<br>

2. 자바 7에 추가된 `java.util.Objects.requireNonNull` 메서드로 null 검사를 수행한다.

```java
this.strategy = Objects.requireNonNull(strategy, "전략");
```

- 원하는 예외 메시지를 지정할 수 있다.
- 입력을 그대로 반환하므로 값을 사용하는 동시에 null 검사를 수행할 수 있다.
- 반환값은 무시하고 필요한 곳 어디서든 순수한 null 검사 목적으로 사용해도 된다.

<br>

3. 자바 9에서는 `Objects`에 범위 검사 기능이 더해졌다.
- `checkFromIndexSize`, `checkFromToIndex`, `checkIndex`라는 메서드들을 활용할 수 있다.
- 대신 예외 메시지를 지정할 수 없고, 리스트와 배열 전용으로 설계되었다.

<br>

4. public이 아닌 메서드일 때 단언문(assert)를 사용해 매개변수 유효성을 검증할 수 있다.

```java
private static void sort(long a[], int offset, int length) {
	assert a != null;
	assert offset <= 0 && offset <= a.length;
	assert length >= 0 && length <= a.length - offset;
	// 생략
}
```

- 단언문들은 자신이 단언한 조건이 무조건 참이라고 선언한다.
- 실패하면 `AssertionError`를 던진다.
- 런타임에 아무런 효과도, 아무런 성능 저하도 없다.

<br>

## 매개변수 검사 예외

- 유효성 검사 비용이 지나치게 높거나 실용적이지 않을 때, 혹은 계산 과정에서 암묵적으로 검사가 수행될 때는 유효성 검사를 해야한다는 규칙에서 예외로 둘 수 있다.
- `Collection.sort()`의 경우 정렬 과정에서 비교가 이뤄지므로 미리 검사해봐야 별다른 실익이 없다.
- 계산 중 잘못된 매개변수 값을 사용해 발생한 예외와 API 문서에서 던지기로 한 예외가 다를 수 있을 때는 `예외 번역(exception translate)` 관용구를 사용하여 API 문서에 기재된 예외로 번역해줘야 한다. (아이템 73 내용)