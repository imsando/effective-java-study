# 커스텀 직렬화 형태를 고려해보라

## ⛳️ 목표

자바의 기본 직렬화 형태와 커스텀 직렬화 형태를 알아보자

<br>

## 📄 핵심 요약

- 자바의 기본 직렬화 형태는 객체를 직렬화한 결과가 해당 객체의 논리적 표현에 부합할 때 사용하자.
- 부합하지 않을 때는 커스텀 직렬화 형태를 고려하자.

---

> 먼저 고민해보고 괜찮다고 판단될 때만 기본 직렬화 형태를 사용하라.
기본 직렬화 형태는 유연성, 성능, 정확성 측면에서 신중히 고민한 후 합당할 때만 사용해야 한다.

<br>

## 기본 직렬화 형태에 적합한 후보

객체의 물리적 표현과 논리적 내용이 같다면 기본 직렬화 형태라도 무방하다.

```java
public class Name implements Serializable {
	/**
	* 성. null이 아니어야 함.
	* @serial
	*/
	private final String lastName;
	/**
	* 이름. null이 아니어야 함.
	* @serial
	*/
	private final String firstName;
	/**
	* 중간이름. 중간이름이 없다면 null.
	* @serial
	*/
	private final String middleName;
	// 생략
}
```

- 성명은 논리적으로 이름, 성, 중간 이름이라는 3개의 문자열로 구성된다.
- 앞 코드의 인스턴스 필드들은 이 논리적 구성요소를 정확히 반영했다.
- 불변식 보장과 보안을 위해 `readObject` 메서드를 제공해야 할 때가 많은데, 이 코드에서는 해당 메서드로 lastName과 firstName 필드가 null이 아님을 보장해야 한다.
- 공개 API에 속하기 때문에 문서화를 해야 한다.

<br>

## 기본 직렬화 형태에 적합하지 않은 후보

```java
public final class StringList implements Serializable {
	private int size = 0;
	private Entry head = null;
	
	private static class Entry implements Serializable {
		String data;
		Entry next;
		Entry previous;
	}
	// 생략
}
```

- 이 클래스는 논리적으로 일련의 문자열을 표현하는데, 물리적으로는 문자열들을 이중 연결 리스트로 연결했다.
- 객체의 물리적 표현과 논리적 표현의 차이가 클 때 기본 직렬화 형태를 사용하면 네 가지 면에서 문제가 생긴다.

<br>

**[1] 공개 API가 현재의 내부 표현 방식에 영구히 묶인다.**

- 내부 표현 방식을 바꾸더라도 여전히 `LinkedList` 로 표현된 입력도 처리할 수 있어야 한다.
- 더는 이 자료구조 방식을 사용하지 않더라도 관련 코드를 절대 제거할 수 없다.

**[2] 너무 많은 공간을 차지할 수 있다.**

- `LinkedList` 의 모든 엔트리와 연결 정보까지 기록하게 되는데, 이 정보는 내부 구현에 해당하니 직렬화 형태에 포함할 가치가 없다.
- 직렬화 형태가 이처럼 너무 커지면 디스크에 저장하거나 네트워크로 전송하는 속도가 느려진다.

**[3] 시간이 너무 많이 걸릴 수 있다.**

- 직렬화 로직은 객체 그래프의 위상(topology)에 관한 정보가 없으니 그래프를 직접 순회해볼 수 밖에 없다.

**[4] 스택 오버플로를 일으킬 수 있다.**

- 기본 직렬화 과정은 객체 그래프를 재귀 순회하는데, 중간 정도 크기의 객체 그래프에서도 자칫 스택 오버플로를 일으킬 수 있다.

<br>

## 커스텀 직렬화 형태

위의 코드를 합리적인 직렬화 형태로 바꿔보자.

```java
public final class StringList implements Serializable {
	private transient int size = 0;
	private transient Entry head = null;
	
	// 이제는 직렬화되지 않는다.
	private static class Entry implements Serializable {
		String data;
		Entry next;
		Entry previous;
	}
	
	// 지정한 문자열을 이 리스트에 추가한다.
	public final void add(String s) { ... }
	
	/**
	* 이 {@code StringList} 인스턴스를 직렬화한다.
	*
	* @serialData 이 리스트의 크기(포함된 문자열의 개수)를 기록한 후
	* ({@code int}), 이어서 모든 원소를(각각은 {@code String})
	* 순서대로 기록한다.
	*/
	private void writeObject(ObjectOutputStream s) throws IOException {
		s.defaultWriteObject();
		s.writeInt(size);
		
		// 모든 원소를 올바른 순서로 기록한다.
		for (Entry e = head; e != null; e = e.next) 
			s.writeObject(e.data);
	}
	
	private void readObject(ObjectInputStream s) throws IOException, ClassNotFoundException {
		s.defaultReadObject();
		int numElements = s.readInt();
		
		// 모든 원소를 읽어 이 리스트에 삽입한다.
		for (int i = 0; i < numElements; i++) 
			add((String) s.readObject());
	}
	
	// 생략
}
```

- 물리적인 상세 표현은 배제하고 논리적인 구성만 담는다.
- `writeObject`, `readObject`가 직렬화 형태를 처리한다.
- `transient`가 선언되었더라도 `defaultWriteObject`, `defaultReadObject`가 호출되어야 하는데, 향후 릴리스에서 transient가 아닌 인스턴스 필드가 추가되더라도 상호 호환되기 위해서이다.
    - 해당 객체의 논리적 상태와 무관한 필드라고 확신할 때만 transient 한정자를 생략해야 한다.
    - 기본 직렬화를 사용한다면 transient 필드들은 역직렬화될 때 기본값으로 초기화된다.

<br>

> **참고. 해시테이블**  
해시테이블은 물리적으로는 키-값 엔트리들을 담은 해시 버킷을 차례로 나열한 형태다. 어떤 엔트리를 어떤 버킷에 담을지는 키에서 구한 해시코드가 결정하는데, 계산 방식은 구현에 따라 혹은 계산할 때마다 달라진다. 따라서 해시테이블에 기본 직렬화를 사용하면 심각한 버그로 이어질 수 있따.


## 그외 알아둬야 할 것들

**[1] synchronized 사용 클래스**

- 객체의 전체 상태를 읽는 메서드에 적용해야 하는 동기화 메커니즘을 직렬화에도 적용해야 한다.

<br>

**[2] 직렬 버전 UID**

- 어떤 직렬화 형태를 택하든 직렬화 가능 클래스에는 직렬 버전 UID를 명시적으로 부여하자.
    - 명시하지 않으면 런타임에 이 값을 생성하느라 복잡한 연산을 수행해야 한다.
- 구버전으로 직렬화된 인스턴스들과의 호환성을 끊으려는 경우를 제외하고는 직렬 버전 UID를 절대 수정하지 말자.