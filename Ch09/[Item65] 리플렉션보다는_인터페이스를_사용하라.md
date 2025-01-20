# 리플렉션보다는 인터페이스를 사용하라

## ⛳️ 목표

리플렉션의 기능과 한계점을 알아보고, 효율적으로 사용하는 방법을 이해하자.

<br>

## 📄 핵심 요약

- 리플렉션을 사용해야 한다면, 인스턴스 생성까지만 사용하자. 그 이후의 작업들(메서드 호출 등)은 성능의 저하를 일으킬 수 있다.
- 인스턴스를 생성한 이후에는 인터페이스나 상위 클래스를 참조하여 사용해야 한다.

---

## 리플렉션의 기능

- 리플렉션 기능(java.lang.reflect)을 이용하면 프로그램에서 임의의 클래스에 접근할 수 있다.
- Class 객체가 주어지면 그 클래스의 생성자, 메서드, 필드에 해당하는 인스턴스를 통해 클래스의 멤버 이름, 필드 타입, 메서드 시그니처 등의 정보를 가져올 수 있다.
- 해당 클래스의 인스턴스를 생성하거나, 메서드를 호출하거나, 필드에 접근할 수 있다.

<br>

## 리플렉션의 단점

- 컴파일타임 타입 검사가 주는 이점을 누릴 수 없다.
- 리플렉션을 이용하면 코드가 지저분해지고 장황해진다.
- 성능이 떨어진다.

**즉, 리플렉션은 아주 제한된 형태로만 사용해야 그 단점을 피하고 이점만 취할 수 있다.**

<br>

## 리플렉션 사용법 - 인터페이스 참조

리플렉션은 인스턴스 생성에만 쓰고, 이렇게 만든 인스턴스는 인터페이스나 상위 클래스로 참조해 사용하자.

```java
public static void main(String[] agrs) {
	// 클래스 이름을 Class 객체로 변환
	Class<? extends Set<String>> cl = null;
	try {
		cl = (Class<? extends Set<String>>) // 비검사 형변환!
			Class.forName(args[0]);
	} catch (ClassNotFoundException e) {
		fatalError("클래스를 찾을 수 없습니다.");
	}
	
	// 생성자를 얻는다.
	Constructor<? extends Set<String>> cons = null;
	try {
		cons = cl.getDeclaredConstructor();
	} catch (NoSuchMethodException e) {
		fatalError("매개변수 없는 생성자를 찾을 수 없습니다.");
	}
	
	// 집합의 인스턴스를 만든다.
	Set<String> s = null;
	try {
		s = cons.newInstance();
	} catch (IllegalAccessException e) {
		fatalError("생성자에 접근할 수 없습니다.");
	} catch (InstantiationException e) {
		fatalError("클래스를 인스턴스화할 수 없습니다.");
	} catch (InvocationTargetException e) {
		fatalError("생성자가 예외를 던졌습니다: " + e.getCause());
	} catch (ClassCastException e) {
		fatalError("Set을 구현하지 않은 클래스입니다.");
	}
	
	// 생성한 집합을 사용한다.
	s.addAll(Arrays.asList(args).subList(1, args.legnth));
}
```

**예제를 통해 보는 리플렉션의 단점**

- 런타임에 총 여섯 가지나 되는 예외를 던질 수 있다. 리플렉션 없이 생성했다면 컴파일타임에 해결할 수 있는 예외들이다.
- 클래스 이름만으로 인스턴스를 생성해내기 위해 긴 코드를 작성해야 했다.

<br>

## 정리

- 리플렉션은 런타임에 존재하지 않을 수도 있는 다른 클래스, 메서드, 필드와의 의존성을 관리할 때 적합하다. (e.g. 버전이 여러 개 존재하는 외부 패키지를 다룰 때)
- 사용할 때는 되도록 객체 생성에만 사용하고, 생성한 객체를 이용할 때는 적절한 인터페이스나 컴파일타임에 알 수 있는 상위 클래스로 형변환해 사용해야 한다.