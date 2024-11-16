## 이왕이면 제네릭 타입으로 만들라

## ⛳️ 목표

비제네릭 타입을 제네릭 타입으로 변환할 때의 이점을 알아보자

<br>

## 📄 핵심 요약

- 기존 비제네릭 타입은 호출 시마다 형변환을 해야 한다.
- 타입 매개변수를 선언하여 제네릭 타입으로 변환하면 타입 안정성을 보장할 수 있다.

---

## 비제네릭 타입 사용 예제

클라이언트가 이 코드를 사용하게 되면 스택에서 꺼낸 객체를 **형변환**해야 하는데 이때 **런타임 오류**가 날 위험이 있다. `타입 안정성`**이 필요하다.**

```java
public class Stack {
    private List elements = new ArrayList();

    public void push(Object e) {
        elements.add(e);
    }

    public Object pop() {
        if (elements.isEmpty()) {
            throw new EmptyStackException();
        }
        return elements.remove(elements.size() - 1);
    }
}

```

<br>

## 제네릭 타입으로 변환 예제

```java
// AS-IS: Stack { ... }
public class Stack<E> { 

		// AS-IS: private List elements = new ArrayList();
    private List<E> elements = new ArrayList<>(); 

		// AS-IS: push(Object e)
    public void push(E e) {
        elements.add(e);
    }

		// AS-IS: Object pop()
    public E pop() {
        if (elements.isEmpty()) {
            throw new EmptyStackException();
        }
        return elements.remove(elements.size() - 1);
    }
}
```

- 클래스 선언에 `타입 매개변수`를 추가한다. 이때 타입 이름은 보통 `E`를 사용한다.
- 코드에 쓰인 Object를 타입 매개변수로 바꾼다. 이 코드에서는 List와 메서드 시그니처에 타입 매개변수를 적용했다.

<br>

### 배열의 경우

배열을 사용하는 코드를 제네릭으로 만드려고 할 때 사용할 수 있는 두 가지 방법을 소개한다.

**[1] 첫 번째 방법**

```java

// AS-IS
private Object[] elements;

public Stack() {
    elements = new Object[DEFAULT_CAPACITY];
}

// TO-BE
private E[] elements;

@SuppressWarnings("unchecked")
public Stack() {
    elements = (E[]) new Object[DEFAULT_CAPACITY];
}

```

- Object 배열을 생성한 다음 타입 매개변수를 사용해 제네릭 배열로 형변환한다.
- 형변환으로 인해 발생할 수 있는 경고는 `@SuppressWarnings(”unchecked”)` 를 사용해 제외시킨다.

<br>

**[2] 두 번째 방법**

```java
// AS-IS
private Object[] elements;

public Stack() {
    elements = new Object[DEFAULT_CAPACITY];
}

public Object pop() {
    if (elements.isEmpty()) {
        throw new EmptyStackException();
    }
    
    Object result = elements[--size];
    elements[size] = null;
    
    return result;
}

// TO-BE
private Object[] elements;

public Stack() {
    elements = new Object[DEFAULT_CAPACITY];
}

public E pop() {
    if (elements.isEmpty()) {
        throw new EmptyStackException();
    }
    
    @SuppressWarnings("unchecked")
    E result = (E) elements[--size];
    elements[size] = null;
    
    return result;
}
```

- 배열의 타입은 Object 배열로 유지한다.
- E는 실체화 불가 타입으로 컴파일러가 런타임에 이뤄지는 형변환이 안전한지 증명할 수 없다.
- 형변환으로 인해 발생할 수 있는 경고는 `@SuppressWarnings(”unchecked”)` 를 사용해 제외시킨다.

<br>

**[3] 비교**

|  | 첫 번째 방법                                                                                       | 두 번째 방법 |
| --- |-----------------------------------------------------------------------------------------------| --- |
| 장점 | - 가독성이 좋다. <br> - 배열의 타입을 `E[]`로 선언하여 오직 E 타입 인스턴스만 받음을 명시한다. <br> - 형변환은 배열 생성 시 한 번만 하면 된다. | -`힙 오염(heap pollution)`이 발생하지 않는다. |
  | 단점 | 배열의 런타임 타입이 컴파일타임 타입과 달라 `힙 오염(heap pollution)`이 발생한다.                                        | 형변환은 배열에서 원소를 읽을 때마다 해줘야 한다. |