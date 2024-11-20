# 와일드카드 타입을 사용하라

## ⛳️ 목표

- 자바의 와일드카드 타입을 이해하고 사용해야하는 이유에 대해서 알아보자

<br>

## 📄 핵심 요약

### **와일드카드 타입이란?**

- **제네릭 타입의 유연성**을 확보하기 위해 사용하는 특별한 매개변수화 타입이다
- 제네릭 타입은 기본적으로 불공변(Invariant)이므로 `List<Type1>`과 `List<Type2>`는 아무 관계도 없다
    - **예시** : `List<String>`은 `List<Object>`의 하위 타입이 아니다
- 위와 같은 문제를 와일드카드(`?`)를 사용하여 이를 보완할 수 있다

<br>

### **PECS 원칙**

- 와일드카드 타입은 **생산자(Producer)** 와 **소비자(Consumer)** 의 역할에 따라 다르게 사용한다
- **PECS 공식**
    - **Producer(생산자)** : `<? extends T>` 사용
        - 데이터를 제공하므로 읽기만 가능하다
    - **Consumer(소비자)** : `<? super T>` 사용
        - 데이터를 소비하므로 쓰기만 가능하다

<br>

---

### **1. Producer (생산자)**

#### **예시상황 : 데이터를 읽어오는 경우**

- `<? extends T>`를 사용해 유연성을 확보한다
- 생산자 역할인 입력 매개변수에서 적합하다

#### 예시 : `pushAll` 메서드 수정

```java
public void pushAll(Iterable<? extends E> src) {
    for (E e : src)
        push(e);
}
```

#### 동작 방식

- `Iterable<? extends E>`는 `E`의 하위 타입만 받을 수 있다
- **예시코드**
  ```java
  Stack<Number> stack = new Stack<>();
  Iterable<Integer> integers = List.of(1, 2, 3);
  stack.pushAll(integers); // Number의 하위 타입인 Integer가 가능
  ```

<br>

---

### **2. Consumer (소비자)**

#### **예시상황 : 데이터를 저장하거나 소비하는 경우**

- `<? super T>`를 사용해 유연성을 확보한다
- 소비자 역할인 입력 매개변수에서 적합하다

#### 예시 : `popAll` 메서드 수정

```java
public void popAll(Collection<? super E> dst) {
    while (!isEmpty())
        dst.add(pop());
}
```

#### 동작 방식

- `Collection<? super E>`는 `E`의 상위 타입만 받을 수 있다
- **예시코드**
  ```java
  Stack<Number> stack = new Stack<>();
  Collection<Object> objects = new ArrayList<>();
  stack.popAll(objects); // Number의 상위 타입인 Object가 가능
  ```

<br>

---

### **3. PECS 원칙 적용 사례**

#### **생산자-소비자 동시 역할**
- 입력 매개변수가 **생산자와 소비자 역할을 동시에** 하면 와일드카드를 사용할 수 없다
- 정확한 타입을 지정해주어야 한다
- **예시코드**
  ```java
  public static <E> void swap(List<E> list, int i, int j) {
      E temp = list.get(i);
      list.set(i, list.get(j));
      list.set(j, temp);
  }
  ```

<br>

#### **복잡한 상황 해결 : 헬퍼 메서드 사용**
- 와일드카드와 실제 타입 혼합 시 헬퍼 메서드로 문제를 해결할 수 있다
- **예시코드 : Swap 메서드**
  ```java
  public static void swap(List<?> list, int i, int j) {
      swapHelper(list, i, j);
  }

  private static <E> void swapHelper(List<E> list, int i, int j) {
      list.set(i, list.set(j, list.get(i)));
  }
  ```

<br>

---

### **4. 추가 사례**

#### **Chooser 클래스**

- `<? extends T>`를 사용해 생성자 유연성을 추가한다

```java
public class Chooser<T> {
    private final List<T> choiceList;

    public Chooser(Collection<? extends T> choices) {
        choiceList = new ArrayList<>(choices);
    }

    public T choose() {
        Random rnd = ThreadLocalRandom.current();
        return choiceList.get(rnd.nextInt(choiceList.size()));
    }
}
```

<br>

#### **union 메서드**

- `<? extends T>`로 입력 매개변수 유연성을 확보한다

```java
public static <E> Set<E> union(Set<? extends E> s1, Set<? extends E> s2) {
    Set<E> result = new HashSet<>(s1);
    result.addAll(s2);
    return result;
}
```

---

> 💡 **핵심 정리**
>
> 1. **와일드카드 타입으로 API 유연성 극대화**
>    - 생산자는 `<? extends T>`를 사용한다
>    - 소비자는 `<? super T>`를 사용한다
> 2. **제네릭 타입이 불공변임을 인지**하고 와일드카드로 해결한다