# 추상화 수준에 맞는 예외를 던져라

## ⛳️ 목표

저수준 예외가 전파되는 문제를 해결하기 위해 활용할 수 있는 예외 번역과 예외 연쇄를 알아보자

<br>

## 📄 핵심 요약

- 예외 번역을 활용해 저수준 예외가 상위 계층으로 전파되어 노출되는 것을 막을 수 있다.
- 예외 연쇄를 활용하면 문제의 근본 원인에 대해 쉽게 디버깅하고 조치를 취할 수 있다.

---

## 예외 번역(exception translation)

메서드가 저수준 예외를 처리하지 않고 바깥으로 전파해 버리면 수행하려는 코드와 전혀 상관없는 예외가 발생할 수 있다. 이 문제를 피하기 위해 **상위 계층에서는 저수준 예외를 잡아 자신의 추상화 수준에 맞는 예외로 바꿔 던지는 것**을 `예외 번역`이라고 한다.

```java
try {
	... // 저수준 추상화를 이용한다.
} catch (LowerLevelException e) {
	// 추상화 수준에 맞게 번역한다.
	throw new HigherLevelException(...);
}

```

<br>

***e.g. AbstractSequentialList***

리스트 안의 지정한 위치의 원소를 반환하는 메서드이다. index가 범위 밖일 때 `IndexOutOfBoundsException` 예외가 발생한다.

```java
public E get(int index) {
	ListIterator<E> i = listIterator(index);
	try {
		return i.next();
	} catch (NoSuchElementException e) {
		throw new IndexOutOfBoundsException("인덱스: " + index);
	}
}
```

<br>

## 예외 연쇄(exception chaining)

`예외 연쇄`란 **문제의 근본 원인인 저수준 예외를 고수준 예외에 실어 보내는 방식**을 말한다. 예외 번역 시 저수준 예외가 디버깅에 도움이 된다면 예외 연쇄를 사용하는 것이 좋다.

```java
// 1. 예외 반환 시 저수준 예외 전달
try {
	... // 저수준 추상화를 이용한다.
} catch (LowerLevelException cause) {
	// 저수준 예외를 고수준 예외에 실어 보낸다.
	throw new HigherLevelException(cause);
}

// 2. HigherLevelException 생성자
class HigherLevelException extends Exception {
	HigherLevelException(Throwable cause) {
		super(cause);
	}
}
```

- 고수준 예외의 생성자는 상위 클래스의 생성자에게 ‘원인’을 건네주어 최종적으로 Throwable(Throwable) 생성자까지 건네지게 된다.
- 대부분의 표준 예외는 예외 연쇄용 생성자를 갖추고 있따.
- 그렇지 않은 예외는 Throwable.initCause() 메서드를 이용해 ‘원인’을 직접 설정할 수 있다.

<br>

## 주의할 점

- 무턱대고 예외를 전파하는 것보다 예외 번역이 우수하지만, 남용하는 것도 좋지 않다.
- 상위 계층 메서드의 매개변수 값을 하위 계층 메서드로 전달하기 전에 미리 검사하는 방법으로 예외가 발생하지 않게 방지하는 것도 좋다.
- 하위 계층에서의 예외를 피할 수 없다면 상위 계층에서 예외를 처리하고 로깅하여 개발자가 추가 조치를 취할 수 있게 해야 한다.

<br>

> **핵심 정리**  
아래 계층의 예외를 예방하거나 스스로 처리할 수 없고, 그 예외를 상위 계층에 그대로 노출하기 곤란하다면 예외 번역을 사용하라. 이때 예외 연쇄를 이용하면 상위 계층에는 맥락에 어울리는 고수준 예외를 던지면서 근본 원인도 함께 알려주어 오류를 분석하기 좋다.
>