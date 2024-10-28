## 상속보다는 컴포지션을 사용하라

## ⛳️ 목표

상속의 문제점을 해결하기 위한 컴포지션과 전달 사용 방식에 대해 알아보자

<br>

## 📄 핵심 요약

- 상속은 상위 클래스와 하위 클래스가 IS-A 관계일 때 사용해야 한다.
- 상속은 캡슐화를 깨뜨리며 그로 인한 메서드 재정의로 코드의 오작동을 불러일으킬 수 있다.
- 컴포지션과 전달을 사용하면 기존 클래스의 영향에서 벗어나 원하는 기능을 추가해 활용할 수 있다.

---

## 상속의 문제점

메서드 호출과 달리 상속은 캡슐화를 깨뜨린다. 상위 클래스가 어떻게 구현되느냐에 따라 하위 클래스의 동작에 이상이 생길 수 있다.

<br>

### [1] 하위 클래스가 상위 클래스의 구현을 모를 때

`HashSet`을 상속받아 원소가 더해진 결과값들을 계산하려고 한다.

```java
public class InstrumentedHashSet<E> extends HashSet<E> {
	
	private int addCount = 0;
	
	public InstrumentedHashSet(){}
	
	public InstrumentedHashSet(int initCap, float loadFactor) {
		super(initCap, loadFactor);
	}
	
	@Override public boolean add(E e) {
		addCount++;
		return super.add(e);
	}
	
	@Override public boolean addAll(Collection<? extends E> c) {
		addCount += c.size();
		return super.addAll(c);
	}
	
	public int getAddCount() {
		return addCount;
	}
}
```

`InstrumentedHashSet`의 `addAll`은 addCount에 List.size()인 3을 더한 후 `HashSet`의 `addAll` 을 호출했다.

```java
// 호출
InstrumentedHashSet<String> s = new InstrumentedHashSet<>();
s.addAll(List.of("틱", "탁탁", "펑"));

// 반환 결과
기대 : 3
실제 : 6
```

<br>


`HashSet`의 `addAll`은 각 원소를 `add` 메서드를 호출해 추가한다.

<img width="595" alt="HashSet" src="https://github.com/user-attachments/assets/7b622b10-763c-415c-9193-ec21d8fdc04c">

이때 불리는 `add`가 `InstrumentedHashSet`에서 재정의한 메서드이다. 따라서, 의도하지 않은 방향대로 코드가 동작해 값이 중복해서 더해지는 문제점이 발생하는 것이다.

<br>

### [2] 상위 클래스의 구현 방식이 달라질 때

`Vector`와 `Stack`의 상속 관계에서 Stack은 Vector의 구현 방식에 따라 Thread Safe하게 되어 성능이 저하되는 단점을 가지게 되었다.

이때, Vector 클래스 내부 기능이 바뀌게 되거나 Thread Safe하지 않게 변한다면 Stack 클래스는 어떻게 될까? Stack 자체에서는 아무런 변경 사항이 없더라도 상위 클래스의 변경 사항에 따라 작동하게 되게 된다.

<br>

## 컴포지션

상속의 문제점은 컴포지션으로 극복할 수 있다. 기존 클래스를 확장하는 대신 새로운 클래스를 만들고 private 필드로 기존 클래스의 인스턴스를 참조하게 하자.

- `전달(forwarding)` : 새 클래스의 인스턴스 메서드(전달 메서드)들은 기존 클래스의 대응하는 메서드를 호출해 그 결과를 반환한다.

<br>

### [1] 장점

새로운 클래스는 기존 클래스의 내부 구현 방식의 영향에서 벗어나며, 심지어 기존 클래스에 새로운 메서드가 추가되더라도 전혀 영향을 받지 않는다.

**<예제>**

위에 상속을 사용해서 문제가 발생했던 코드를 컴포지션을 사용해서 변경해 본다.

***Wrapper class***

`데코레이터 패턴`과 같이 상속을 사용해 구체 클래스들을 만들지 않아도 기능을 동적으로 확장할 수 있게 한다.

```java
public class InstrumentedSet<E> extends ForwardingSet<E> {
	
	private int addCount = 0;
	
	public InstrumentedHashSet(Set<E> s){
		super(s);
	}
	
	@Override public boolean add(E e) {
		addCount++;
		return super.add(e);
	}
	
	@Override public boolean addAll(Collection<? extends E> c) {
		addCount += c.size();
		return super.addAll(c);
	}
	
	public int getAddCount() {
		return addCount;
	}
}
```

<Br>

***Forwarding class***

임의의 Set에 계측 기능을 추가해 새로운 Set으로 만든다.

```java
public class ForwardingSet<E> implements Set<E> {
    
    private final Set<E> s;   

    public ForwardingSet(Set<E> s) { this.s = s; }

    public void clear()               { s.clear();            }
    public boolean contains(Object o) { return s.contains(o); }
    public boolean isEmpty()          { return s.isEmpty();   }
    public int size()                 { return s.size();      }
    public Iterator<E> iterator()     { return s.iterator();  }
    public boolean add(E e)           { return s.add(e);      }
    public boolean remove(Object o)   { return s.remove(o);   }
    
    public boolean containsAll(Collection<?> c) { return s.containsAll(c); }
    public boolean addAll(Collection<? extends E> c) { return s.addAll(c);      }
    public boolean removeAll(Collection<?> c) { return s.removeAll(c);   }
    public boolean retainAll(Collection<?> c) { return s.retainAll(c);   }
    
    public Object[] toArray()          { return s.toArray();  }
    public <T> T[] toArray(T[] a)      { return s.toArray(a); }
    
    @Override public boolean equals(Object o) { return s.equals(o);  }
    @Override public int hashCode()    { return s.hashCode(); }
    @Override public String toString() { return s.toString(); }
}
```

<br>

### [2] 단점

콜백 프레임워크에서 내부 객체는 자신을 감싸고 있는 래퍼의 존재를 몰라 대신 자신(this)의 참조를 넘기고, 콜백 때는 래퍼가 아닌 내부 객체를 호출하게 되는 `SELF` 문제가 발생한다.

<br>

## 상속과 컴포지션 적용

상속은 `is-a` 관계일 때만 사용한다. 그렇지 않다면 모두 컴포지션으로 구현해야 한다.

### 컴포지션 대신 상속을 사용했을 때의 문제점

- 내부 구현을 불필요하게 노출하여 API가 내부 구현에 묶이고 클래스의 성능도 제한된다.
- 클라이언트가 내부에 직접 접근할 수 있어 상위 클래스를 직접 수정한다면 하위 클래스에 악영향이 갈 수 있다.
- ***e.g.*** HashTable과 Properties, Vector와 Stack

### 체크 리스트

- 확장하려는 클래스의 API에 아무런 결함이 없는가?
- 결함이 있다면, 이 결함이 상속을 받아 구현하려는 클래스의 API에 아무런 결함이 없는가?
