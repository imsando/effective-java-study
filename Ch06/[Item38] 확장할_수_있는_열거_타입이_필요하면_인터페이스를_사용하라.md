# 확장할 수 있는 열거 타입이 필요하면 인터페이스를 사용하라

## ⛳️ 목표
열거 타입이 확장될 수 없는 이유를 이해하고 인터페이스를 활용하는 방법을 알아보자

<br>

## 📄 핵심 요약
- 열거 타입은 고정된 값을 사용하기 때문에 상속이 불가능하며 그로 인해 확장 또한 불가하다.
- 열거 타입을 확장하는 대신 인터페이스를 구현하는 여러 타입을 추가하는 방식을 사용할 수 있다.
---

## 타입 안전 열거 패턴(typesafe enum pattern)

- `JDK 1.5` 이전에 열거 타입 대신 사용하던 패턴이다.
- 자유로운 확장이 가능하다.

```java
// 출처 : https://stackoverflow.com/questions/5092015/advantages-of-javas-enum-over-the-old-typesafe-enum-pattern
public class Suit {
    private final String name;

    public static final Suit CLUBS =new Suit("clubs");
    public static final Suit DIAMONDS =new Suit("diamonds");
    public static final Suit HEARTS =new Suit("hearts");
    public static final Suit SPADES =new Suit("spades");    

    private Suit(String name){
        this.name =name;
    }
    public String toString(){
        return name;
    }
}
```
<br>

## 열거 타입과 확장

- 열거 타입은 기본적으로 확장할 수 없다.
- 확장 시에 기반 타입과 확장된 타입들의 원소 모두를 순회할 방법이 없다.
- 확장성을 높이려면 설계와 구현이 더 복잡해진다.

<br>

### 확장이 필요한 경우

연산 코드와 같이 이따금 API가 제공하는 기본 연산 외에 사용자 확장 연산을 추가할 수 있도록 해야 할 때 필요하다. 이때는 인터페이스를 사용할 수 있다.

```
[한계점]
열거 타입끼리 구현을 상속할 수 없다.
→ 아무 상태에도 의존하지 않는 경우 디폴트 구현을 이용해 인터페이스에 추가한다.
→ 공유하는 기능이 많다면 별도의 도우미 클래스로 코드 중복을 줄일 수 있다.
```

<br>

***기존 열거 타입 - 인터페이스 구현***

```java
public interface Operation {
	double apply(double x, double y);
}

public enum BasicOperation implements Operation {
	PLUS("+") {
		public double apply(double x, double y) { return x + y; };	
	},
	MINUS("-") {
		public double apply(double x, double y) { return x - y; };	
	},
	TIMES("*") {
		public double apply(double x, double y) { return x * y; };	
	},
	DIVIDE("/") {
		public double apply(double x, double y) { return x / y; };	
	};
	
	private final String symbol;
	
	BasicOperation(String symbol) {
		this.symbol = symbol;
	}
	
	@Override public String toString() {
		return symbol;
	}
}
```

- 열거 타입인 `BasicOperation`은 확장할 수 없다.
- 인터페이스인 `Operation`은 확장할 수 있고, 이 인터페이스를 연산의 타입으로 사용할 수 있다.

<br>

***연산 추가 시 - 같은 인터페이스 구현***

```java
public enum ExtendedOperation implements Operation {
	EXP("^") {
		public double apply(double x, double y) {
			return MAth.pow(x, y);
		}
	},
	REMAINDER("%") {
		public double apply(double x, double y) {
			return x % y;
		}
	};
	
	private final String symbol;
	
	ExtendedOperation(String symbol) {
		this.symbol = symbol;
	}
	
	@Override public String toString() {
		return symbol;
	}
}
```

<br>

### 확장된 열거 타입 사용

***한정적 타입 토큰 활용***

한정적 타입 토큰 역할을 하는 class 리터럴을 인자로 넘긴다.

```java
public static void main(String[] args) {
	double x = Double.parseDouble(args[0]);
	double y = Double.parseDouble(args[1]);
	test(ExtendedOperation.class, x, y);
}

// Class 객체가 열거 타입인 동시에 Opearation의 하위 타입이어야 함
private static <T extends Enum<T> & Operation> void test(
	Class<T> opEnumType, double x, double y) {
	for (Operation op : opEnumType.getEnumConstants()) {
		System.out.printf("%f %s %f = %f%n",
			x, op, y, op.apply(x, y));
	}	
}
```

- main 메서드는 test 메서드에 `ExtendedOperation`의 class 리터럴을 넘겨 확장된 연산들이 무엇인지 알려준다.

<br>

***한정적 와일드카드 타입 활용***

Class 객체 대신 한정적 와일드카드 타입인 `Collection<? extends Operation>`을 인자로 넘긴다.

```java
public static void main(String[] args) {
	double x = Double.parseDouble(args[0]);
	double y = Double.parseDouble(args[1]);
	test(Arrays.asList(ExtendedOperation.values()), x, y);
}

private static void test(Collection<? extends Operation> opSet, 
	double x, double y) {
	for (Operation op : opSet) {
		System.out.printf("%f %s %f = %f%n",
			x, op, y, op.apply(x, y));
	}	
}
```

- 여러 구현 타입의 연산을 조합해 호출할 수 있게 되었다.

<br>

```
[핵심 정리]
열거 타입 자체는 확장할 수 없지만, 인터페이스와 그 인터페이스를 구현하는 기본 열거 타입을 함께 사용해 같은 효과를 낼 수 있다. 이렇게 하면 클라이언트는 이 인터페이스를 구현해 자신만의 열거 타입(혹은 다른 타입)을 만들 수 있다. 그리고 API가 (기본 열거 타입을 직접 명시하지 않고) 인터페이스 기반으로 작성되었다면 기본 열거 타입의 인스턴스가 쓰이는 모든 곳을 새로 확장한 열거 타입의 인스턴스로 대체해 사용할 수 있다.
```
