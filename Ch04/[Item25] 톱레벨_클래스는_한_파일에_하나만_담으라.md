## 톱레벨 클래스는 한 파일에 하나만 담으라

## ⛳️ 목표

한 파일에 하나의 Top Level Class를 둬야 하는 이유를 알아보자

<br>

## 📄 핵심 요약

- `Top Level Class`란 중첩 클래스가 아닌 클래스를 말한다.
- 한 파일에 여러 개의 Top Level Class가 위치하면 의도하지 않은 동작이나 버그가 발생할 수 있다.
- 한 파일에 한 개의 Top Level Class를 두어 명확성과 안정성을 유지해야 한다.

---

## 톱레벨 클래스란

중첩 클래스가 아닌 클래스를 `탑 레벨 클래스(Top Level Class)`라고 한다.

> *top level class* is a class that is not a nested class.
> *nested class* is any class whose declaration occurs within the body of another class or interface.
> 
> 출처 : https://docs.oracle.com/javase/specs/jls/se8/html/jls-8.html

<br>

## Bad Case

**[두 클래스가 한 파일에 정의된 경우]**

***Utensil.java***

```java
// Utensil.java
class Utensil {
	static final String NAME = "pan";
}

class Dessert {
	static final String NAME = "cake";
}

```

***Dessert.java***

```java
class Utensil {
	static final String NAME = "pot";
}

class Dessert {
	static final String NAME = "pie";
}
```

***Main.java***

```java

public class Main {
	public static void main(String[] args) {
		System.out.println(Utensil.NAME + Dessert.NAME);
	}
}
```

<br>

**컴파일 명령** 

```java
javac Main.java Dessert.java
```

`동작 과정`

1. Main.java를 컴파일한다.
    1. 이때 Utensil 참조를 만나 [Utensil.java](http://Utensil.java) 파일을 살핀다.
    2. [Utensil.java](http://Utensil.java) 파일 내부의 Utensil과 Dessert를 모두 찾아낸다.
2. Dessert.java를 처리하려고 할 때 같은 클래스의 정의가 이미 있음을 알게 된다.


> 만약, 위의 순서가 아닌 다른 순서로 컴파일한다면 동작이 달라질 수도 있어 원하는 기능을 구현하기 위해서는 해결해야 할 문제이다.

<br>

## Good Case

- Top Level 클래스들이 서로 다른 소스 파일에 위치한다.
- 한 파일에 여러 Top Level 클래스들을 담기 위해 static 클래스를 활용한다.

<br>

***예제***

```java
public class Test {
	public static void main(String[] args) {
		System.out.println(Utensil.NAME + Dessert.NAME);
	}
	
	private static class Utensil {
		static final String NAME = "pan";
	}
	
	private static class Dessert {
		static final String NAME = "cake";
	}
}
```

<br>

> **핵심 정리**  
*소스 파일 하나에는 반드시 톱레벨 클래스(혹은 톱레벨 인터페이스)를 하나만 담자*. 이 규칙만 따른다면 컴파일러가 한 클래스에 대한 정의를 여러 개 만들어내는 일은 사라진다. 소스 파일을 어떤 순서로 컴파일하든 바이너리 파일이나 프로그램의 동작이 달라지는 일은 결코 일어나지 않을 것이다.
