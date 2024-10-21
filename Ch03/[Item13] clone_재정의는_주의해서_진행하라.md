# clone 재정의는 주의해서 진행하라

<br>

## ⛳️ 목표

`clone()` 메서드의 문제점을 이해하고 객체 복사를 안전하게 구현하는 방법을 알아보자.

<br>

## 📄 핵심 요약

- `clone()` 메서드를 사용할 때는 얕은 복사의 문제를 주의하자.
- `clone()` 메서드를 사용하기 보다는 복사 생성자나 복사 팩터리를 사용하자.
- 배열의 경우 `clone()` 메서드를 최적으로 사용할 수 있으니 활용하자.

---

## Cloneable 인터페이스

`Cloneable`은 복제해도 되는 클래스임을 명시하는 용도의 믹스인 인터페이스이다.

***Cloneable***

```java
public interface Cloneable {
}
```

***Object.class***

```java
@IntrinsicCandidate
protected native Object clone() throws CloneNotSupportedException;
```

<br>

### 역할

- `Object`의 `protected` 메서드인 `clone()`의 **동작 방식**을 결정한다.
- Cloneable을 구현한 클래스의 인스턴스에서 clone()을 호출하면 그 객체의 필드들을 하나하나 복사한 객체를 반환하며, 그렇지 않은 클래스의 인스턴스에서 호출하면 `CloneNotSupportedException`을 던진다.

<br>

### 문제점

- clone 메서드가 Cloneable이 아닌 Object에 `protected`로 선언되어있어 Cloneable을 구현하는 것만으로는 외부에서 clone 메서드를 호출할 수 없다.
- `얕은 복사(Shallow Copy)`를 지원하기에 원본 객체와 복제된 객체가 내부 데이터를 공유 했을 때 문제가 발생할 수 있다.

<br>

🏷️ **참고**

> **Mixin 인터페이스**  
특정 기능이나 동작을 다른 클래스와 결합할 수 있도록 하는 인터페이스이며 이를 통해 다양한 클래스에서 공통된 기능을 재사용할 수 있다.
>

<br>

> **배열을 복사하는 여러 가지 방법**
> - `얇은 복사(Shallow Copy)` : 복사된 배열이나 원본 배열이 변경될 때 서로 간의 값이 같이 변경된다.
> - `깊은 복사(Deep Copy)` : 복사된 배열이나 원본 배열이 변경될 때 각자의 값만 바뀐다.
>

<br>

## 재정의 시 따라야 하는 Object 명세

> 이 객체의 복사본을 생성해 반환한다. 어떤 객체 x에 대해 다음 식은 참이다.  
> **- x.clone() ≠ x**  
> **- x.clone().getClass == x.getClass()**
> 
> *다음 식부터는 필수가 아니나 관례상 혹은 일반적으로 참이다.*  
> **- x.clone().equals(x)**  
> **- x.clone().getClass() == x.getClass()**
>
- 마지막 식은 clone() 메서드가 반환하는 객체를 `super.clone()`을 호출해 얻은 것이라면 참일 것이다.
⚠️ clone() 메서드에서 `super.clone()`이 아닌 생성자를 호출해 얻은 인스턴스를 반환해도 컴파일 하는 데는 문제가 없다. 그렇기에 클래스 B가 클래스 A를 상속한다고 했을 때, A가 자신의 생성자를 사용해 반환했을 때 문제가 없지만 B가 `super.clone()`을 호출하면 A 타입의 객체가 반환되는 문제가 생긴다.

<br>

### 구현

**[가변 객체를 참조하지 않는 클래스용]**

```java
public class PhoneNumber implements Cloneable {
	// 생략
	@Override
	public PhoneNumber clone() {
		try {
			return (PhoneNumber) super.clone();
		} catch (CloneNotSupportedException e) {
			throw new AssertionError(); 
		}
	}
}
```

1️⃣ Cloneable 구현하도록 추가하고 clone() 메서드를 재정의한다.

- 접근 제한자는 public으로, 반환 타입은 본인 자신으로 변경한다.

2️⃣ `clone()` 메서드 내부에서 `super.clone()`을 호출한다. 이렇게 얻은 객체가 원본의 완벽한 복제본이다.

3️⃣ 클라이언트가 형변환하지 않도록 `super.clone()`에서 얻은 객체를 반환하기 전에 해당 클래스로 형변환한다.

<br>

🏷️ **참고**

> **super.clone 호출에서 try-catch 블록을 사용해야 하는 이유**  
Object의 clone 메서드가 `검사 예외(Checked Exception)`인 `CloneNotSupportedException`을 던지도록 선언되었기 때문이다.
>

<br>

**[가변 객체를 참조하는 클래스용]**

아래 코드에서 Cloneable을 구현해 `super.clone()` 을 호출하면 원본 스택과 복제된 스택 간에 객체 배열 `elements`가 공유되는 문제가 생긴다. 어느 한쪽에서 데이터를 변경하면 다른 쪽에도 영향을 끼치게 된다.

```java
public class Stack {

	private Object[] elements;
	private int size = 0;
	private static final int DEFAULT_INITIAL_CAPACITY = 16;
	
	public Stack() {
		this.elements = new Object[DEFAULT_INITIAL_CAPACITY];
	}
	
	// 생략
	
}
```

- **참조 타입**을 사용해 배열을 구현하고 있기 때문에 `얕은 복사(Shallow Copy)`가 일어나기 때문이다.
- clone은 원본 객체에 아무런 해를 끼치지 않는 동시에 복제된 객체의 **불변식**을 보장해야 한다.

<br>

⭕️ ***해결 방법 - 깊은 복사(Deep Copy)***

객체 내부에 숨어있는 모든 가변 객체를 복사하고 복제본이 가진 객체 참조 모두가 복사된 객체들을 가리키도록 변경해야 한다.

```java
@Override
public Stack clone() {
	try {
		Stack result = (Stack) super.clone();
		result.elements = elements.clone();
		return result;
	} catch(CloneNotSupportedException e) {
		throw new AssertionError();
	}
}
```

- 배열의 clone은 런타임 타입과 컴파일타임 타입 모두 원본 배열과 똑같은 배열을 반환하므로 Object[]로 형변환할 필요가 없다.
- elements 필드가 final이었다면 새로운 값을 할당할 수 없으므로 사용할 수 없다. 그래서 복제할 수 있는 클래스를 만들기 위해 일부 필드에서 final 한정자를 제거해야 할 수도 있다.

<br>

## 권장 방법

clone()을 재정의하기보다는 `복사 생성자(변환 생성자, conversion constructor)`나 `복사 팩터리(변환 팩터리, conversion factory)`를 사용한다.

```java
// 복사 생성자
public Yum(Yum yum) { ... };

// 복사 팩터리
public static Yum newInstance(Yum yum) { ... };
```

- **예외 처리 간소화** : 불필요한 `checked exception`을 처리하지 않아도 된다.
- **상속 관련 문제 회피** : 상속 구조와 상관없이 각 클래스에서 필요한 복사 로직을 명시하기 때문에 문제를 피할 수 있다.
- **깊은 복사와 얕은 복사 사용** : 내부적으로 어떤 필드에 어떤 복사 방식을 사용할지 직접 구현할 수 있다.
- **불변 객체 복사** : 복사할 필요 없이 기존 객체를 그대로 반환할 수 있다.
- **다형성** : 복사하려는 객체의 타입에 따라 적절한 하위 타입의 인스턴스를 반환할 수 있다.
- **명세 강제 회피** : 잘못 설계된 Cloneable 인터페이스의 Object 명세를 따를 필요 없이 필요에 따라 복제 방식을 정의할 수 있다.

