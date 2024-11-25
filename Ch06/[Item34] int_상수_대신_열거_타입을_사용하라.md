# int 상수 대신 열거 타입을 사용하라

## ⛳️ 목표

정수 및 문자열 열거 타입의 단점을 열거 타입을 통해 어떻게 극복할 수 있는지 알아보자

<br>

## 📄 핵심 요약

- 열거 타입으로 상수를 다룰 때의 타입 안정성을 보장할 수 있다.
- 상수의 의미를 파악하기 쉬우며 메서드나 필드를 추가해 추가 연산을 수행할 수 있다.

---

## 열거 타입 지원 전

**[1] 정수 열거 패턴**

```java
public static final int APPLE_FUJI = 0;
public static final int APPLE_PIPPING = 1;
public static final int APPLE_GRANNY_SMITH = 2;
```

- 타입 안정성을 보장할 수 없다.
- 문자열로 출력했을 때 숫자만 조회되기 때문에 의미를 파악하지 쉽지 않기에 표현력이 좋지 않다.
- 상수를 나열한 것이라 상수의 값이 바뀌면 반드시 다시 컴파일 해야 한다.
- 상수가 몇 개인지 파악할 수 없다.

<br>

**[2] 문자열 열거 패턴(string enum pattern)**

- 상수의 의미를 출력할 수 있다.
- 하드코딩한 문자열을 이용하기 대문에 런타임 버그가 발생한다.
- 문자열 비교에 따른 성능 저하가 발생한다.

<br>

## 열거 타입(enum type)

- 자바의 열거 타입은 완전한 형태의 클래스라 단순한 정숫값일 뿐인 다른 언어의 열거 타입보다 활용하기 좋다.
- 상수 하나당 자신의 인스턴스를 하나씩 만들어 `public static final` 필드로 공개한다.
- 클라이언트가 인스턴스를 직접 생성하거나 확장할 수 없기 때문에 싱글턴을 보장할 수 있다.
- 컴파일타임 타입 안정성을 제공한다. 다른 타입의 값을 넘기려고 할 때 컴파일 오류가 발생한다.
- 새로운 상수를 추가하거나 순서를 바꿔도 다시 컴파일 하지 않아도 된다.
- 메서드나 필드를 추가할 수 있다.

    ```java
    public enum Planet {
    	MERCURY(3.302e+23, 2.439e6),
    	VENUS(4.869e+24, 6.052e6),
    	...;
    
    	private final double mass;
    	private final double radius;
    	private final double suffaceGravity;
    
    	private static final double G = 6.67300E-11;
    
    	// 생성자
    	Planet(double mass, double radius) {
    		this.mass = mass;
    		this.radius = radius;
    		surfaceGravity = G * mass / (radius * radius);
    	}
    
    	public double mass() { return mass; }
    
    	...
    
    	public double surfaceWeight(double mass) {
    		return mass * surfaceGravity;
    	}
    }
    ```

    - 열거 타입 각각을 특정 데이터가 연결지으려면 생성자에서 데이터를 받아 인스턴스 필드에 저장하면 된다.
    - 열거 타입은 근본적으로 불변이라 모든 필드는 `final`이어야 한다.
- 내부에 정의된 상수들의 값을 배열에 담아 반환하는 정적 메서드인 `values()`를 제공한다.

<br>

### 상수별 메서드 구현

열거 타입은 상수 별로 다르게 동작하는 코드를 작성하는 데 도움을 준다.

- 열거 타입에 `apply`라는 추상 메서드를 선언하고 각 상수에서 자신에 맞게 재정의할 수 있다.
- 이를 `상수별 메서드 구현(constant-specific method implementation)`이라고 한다.

<br>

***Operation.class***

```java
public enum Operation {
	PLUS {public double apply(double x, double y) {return x + y}},
	MINUS {public double apply(double x, double y) {return x - y}},
	...;

	public abstract double apply(double x, double y);
}
```

- apply 메서드가 상수 선언 바로 옆에 붙어 있다.
- 추상 메서드이므로 재정의하지 않으면 컴파일 오류로 알려준다.

<br>

### 열거 타입 제약

- 열거 타입의 정적 필드 중 생성자에서 접근할 수 있는 것은 상수 변수뿐이다.
- 열거 타입 생성자가 실행되는 시점에는 정적 필드들이 아직 초기화되기 전이라, 자기 자신을 추가하지 못하게 하는 제약이 꼭 필요하다.
- 열거 타입 생성자에서 같은 열거 타입의 다른 상수에도 접근할 수 없다.

<br>

### Switch 문 활용

- 기존 열거 타입에 상수별 동작을 혼합해 넣을 때는 switch 문이 좋은 선택이 될 수 있다.
- 추가하려는 메서드가 의미상 열거 타입에 속하지 않거나, 종종 쓰이지만 열거 타입 안에 포함할만큼 유용하지 않은 경우 아래의 방식을 적용해서 사용하자.

```java
public static Operation inverse(Operation op) {
	switch (op) {
		case PLUS: return Operation.MINUS;
		case MINUS: return Operation.PLUS;
		case TIMES: return Operation.DIVIDE;
		case DIVIDE: return Operation.TIMES;
	
		default: throw new AssertionError("알 수 없는 연산: " + op); )
	}
}
```

<br>

### 적용 사례

- 필요한 원소를 컴파일타임에 다 알 수 있는 상수 집합이라면 항상 열거 타입을 사용하자.
- 열거 타입에 정의된 상수 개수가 영원히 고정 불변일 필요는 없다.

<br>

```
📢 핵심 정리
열거 타입은 확실히 정수 상수보다 뛰어나다. 더 읽기 쉽고 안전하고 강력하다. 대다수 열거 타입이 명시적 생성자나 메서드 없이 쓰이지만, 각 상수를 특정 데이터와 연결짓거나 상수마다 다르게 동작하게 할 때는 필요하다. 드물게는 하나의 메서드가 상수별로 다르게 동작해야 할 때도 있다. 이런 열거 타입에서는 switch 문 대신 상수별 메서드 구현을 사용하자. 열거 타입 상수 일부가 같은 동작을 공유한다면 전략 열거 타입 패턴을 사용하자.
```