# 가변인수는 신중히 사용하라

## ⛳️ 목표

가변인수 메서드의 성능 저하 포인트를 이해하고 활용 방법을 알아보자

<br>

## 📄 핵심 요약

- 가변인수 메서드를 호출할 때 인수의 개수의 길이와 같은 배열을 할당하고 이 배열에 인수들을 저장하는 작업이 필요하다.
- 필수 매개변수를 먼저 정의하고 사용하거나, 성능을 고려해 다중정의 메서드를 활용할 수 있다.

---

## 가변인수(varargs) 메서드

```java
static int sum(int... args) {
	int sum = 0;
	for (int arg: args) {
		sum += arg;
	}
	return sum;
}
```

- 명시한 타입의 인수를 0개 이상 받을 수 있다.
- 가변인수 메서드를 호출할 때
    - 인수의 개수와 길이가 같은 배열을 만든다.
    - 인수들을 이 배열에 저장하여 가변인수 메서드에게 전달한다.

<br>

### Bad Case::인수가 1개 이상이어야 하는 가변인수 메서드

```java
static int sum(int... args) {
	if (args.length == 0) {
		throw new IllegalArgumentException("인수가 1개 이상 필요합니다.");
	}
	
	int min = args[0];
	for (int i = 1; i < args.length; i++) {
		if (args[i] < min) {
			min = args[i];
		}
	}
	return min;
}
```

- 인수를 0개만 넣어 호출하면 런타임에 실패한다.
- 코드가 지저분하다.

<br>

### Good Case::인수가 1개 이상이어야 할 때 가변인수 활용하는 법

```java
static int sum(int firstArg, int... remainingArgs) {
	int min = firstArg;
	for (int arg : remainingArgs) {
		if (arg < min) {
			min = arg;
		}
	}
	return min;
}
```

- 첫 번째로 평범한 매개변수를 전달받는다.
- 두 번째로 가변인수를 전달받는다.

<br>

## 가변인수의 성능 문제

가변인수 메서드는 호출할 때마다 배열을 새로 하나 할당하고 초기화하므로 성능에 문제가 발생할 수 있다.

<br>

### Good Case::가변인수 메서드 호출의 퍼센티지 활용

해당 메서드 호출의 95%가 인수를 3개 이하로 사용한다고 했을 때

```java
public void foo() {}
public void foo(int a1) {}
public void foo(int a1, int a2) {}
public void foo(int a1, int a2, int a3) {}
public void foo(int a1, int a2, int a3, int... rest) {}
```

- 다중정의 메서드가 인수 4개 이상인 5%의 호출을 담당한다.