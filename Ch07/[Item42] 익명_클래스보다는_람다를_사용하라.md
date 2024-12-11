# 익명 클래스보다는 람다를 사용하라

## ⛳️ 목표

익명 클래스 대신 람다를 사용했을 때의 이점을 알아보자

<br>

## 📄 핵심 요약

- 익명 클래스 방식은 코드가 길어 함수형 프로그래밍에 적합하지 않다.
- 자바 8에 도입된 람다식은 코드가 간결하여 동작을 명확히 드러낼 수 있다.
- `this` 키워드로 람다는 바깥 인스턴스를, 익명 클래스는 자기 자신을 가리킬 수 있다.

---

## 익명 클래스

JDK 1.1이 등장하면서 함수 객체를 만드는 주요 수단이 되었다.

```java
Collections.sort(words, new Comparator<String>() {
	public int compare(String s1, String s2) {
		return Integer.compare(s1.length(), s2.length());
	}
})
```

- Comparator 인터페이스가 정렬을 담당하는 추상 전략을 뜻하며, 문자열을 정렬하는 구체적인 전략을 익명 클래스로 구현했다.
- 익명 클래스 방식은 코드가 너무 길기 때문에 자바는 함수형 프로그래밍에 적합하지 않았다.

<br>

## 람다식(Lambda expression)

자바 8에서는 인터페이스들의 인스턴스를 람다식을 사용해 만들 수 있다.

```java
Collections.sort(words,
	(s1, s2) -> Integer.compare(s1.length(), s2.length()));
```

- 함수나 익명 클래스와 개념은 비슷하지만 코드가 훨씬 간결하여 어떤 동작을 하는지가 명확하게 드러난다.
- 컴파일러가 문맥을 살펴 문맥을 추론해주는데 상황에 따라 프로그래머가 직접 명시할 수 있다.
- 인수 `words`가 매개변수화 타입인 `List<String>`이 아닌 원시 타입의 `List`였다면 컴파일 오류가 발생한다.

<br>

**[1] 비교자 생성 메서드**

```java
Collections.sort(words, comparingInt(String::length));
```

<br>

**[2] List.sort()**

```java
words.sort(comparingInt(String::length));
```
<br>

**[3] 열거 타입**

`AS-IS`

```java
public enum Operation {
	PLUS("+") {
		public double apply(double x, double y) { return x + y };
	};
	
	private final String symbol;
	
	Operation(String symbol) { this.symbol = symbol; }
	
	public abstract double apply(double x, double y);
}
```
<br>

`TO-BE`

```java
public enum Operation {
	PLUS ("+", (x, y) -> x + y);
	
	private final String symbol;
	private final DoubleBinaryOperator op;
	
	Opretion(String symbol, DoubleBinaryOperator op) {
		this.symbol = symbol;
		this.op = op;
	}
	
	public double apply(double x, double y) {
		return op.applyAsDouble(x, y);
	}
}
```

<br>

## 정리

- 람다는 이름도 없고 문서화도 하지 못하므로 코드 자체로 동작히 명확히 설명되지 않거나 코드 줄 수가 많아지면 쓰지 말아야 한다.
- 람다가 길거나 읽기 어렵다면 더 간단히 줄여보거나 람다를 쓰지 않는 쪽으로 리팩토링해야 한다.
- 추상 클래스의 인스턴스를 만들 때 람다를 쓸 수 없으니, 익명 클래스를 써야 한다.
- `this` 키워드는 람다에서 외부 인스턴스를, 익명 클래스에서는 인스턴스 자신을 가리킨다. 함수 객체가 자신을 참조해야 한다면 반드시 익명 클래스를 써야 한다.

<br>

```
핵심 정리

자바가 8로 판올림되면서 작은 함수 객체를 구현하는 데 적합한 람다가 도입되었다. 익명 클래스는 (함수형 인터페이스가 아닌) 타입의 인스턴스를 만들 때만 사용하라. 람다는 작은 함수 객체를 아주 쉽게 표현할 수 있어 (이전 자바에서는 실용적이지 않던)함수형 프로그래밍의 지평을 열었다.

```