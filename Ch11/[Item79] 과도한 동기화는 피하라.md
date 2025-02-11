## ⛳️ 목표

- 과도한 동기화의 문제점을 방지하는 방법에 대해 알아보자.

<br>

## 📄 핵심 요약

#### **동기화(Synchronization)란?**
- 여러 스레드가 동시에 공유 자원에 접근할 때 발생할 수 있는 문제를 방지하기 위해, 특정 코드 블록이나 메서드를 한 번에 하나의 스레드만 실행하도록 제어하는 기술이다.
- `synchronized` 키워드나 동시성 컬렉션을 활용해 구현할 수 있다.

#### **교착상태(Deadlock)란?**
- 둘 이상의 스레드가 서로 상대방이 점유한 자원의 해제를 기다리면서 무한 대기 상태에 빠지는 현상이다.
- 동기화 블록을 잘못 사용하면 발생할 수 있으며, 프로그램이 응답하지 않는 원인이 된다.

---
<br>

### 1. **과도한 동기화가 초래하는 문제**
#### 1️⃣ **성능 저하**
- 불필요한 동기화는 스레드 경쟁을 유발하고, 프로그램의 실행 속도를 저하시킨다.
- 특히 멀티코어 환경에서는 동기화 비용이 더 커질 수 있다.

#### 2️⃣ **교착상태(Deadlock)의 위험**
- 동기화된 블록 안에서 외부 메서드를 호출하면, 해당 메서드가 다시 동기화를 요구하는 경우 교착상태에 빠질 수 있다.
- 예 : `notifyElementAdded` 메서드에서 `SetObserver`의 `added` 메서드를 호출할 때, `removeObserver`를 동시에 실행하면 발생할 수 있다.

#### 3️⃣ **예측할 수 없는 동작 (Unexpected Behavior)**
- 동기화된 블록 안에서 외부 메서드를 호출하면 예외가 발생하거나 데이터가 손상될 위험이 있다.
- 예 : `ConcurrentModificationException` 발생 가능성이 높아진다.

<br>

### 2. **잘못된 동기화 방식의 예제**
#### ❌ 동기화된 블록 내부에서 외부 메서드 호출 (잘못된 코드)
```java
private void notifyElementAdded(E element) {
    synchronized (observers) {
        for (SetObserver<E> observer : observers) {
            observer.added(this, element); // 동기화 블록 내에서 외부 메서드 호출
        }
    }
}
```
- 위 코드에서는 `observer.added` 메서드가 실행될 때 내부적으로 `removeObserver`를 호출하면 `ConcurrentModificationException`이 발생할 수 있다.

<br>

### 3. **과도한 동기화를 피하는 방법**
#### ✅ 동기화 블록 밖에서 외부 메서드 호출 (개선된 코드)
```java
private void notifyElementAdded(E element) {
    List<SetObserver<E>> snapshot;
    synchronized (observers) {
        snapshot = new ArrayList<>(observers); // 복사본 생성
    }
    for (SetObserver<E> observer : snapshot) {
        observer.added(this, element); // 동기화 블록 밖에서 호출
    }
}
```
- `observers` 리스트의 복사본을 만들어 동기화 없이 안전하게 순회할 수 있도록 개선했다.
- 이를 **열린 호출 (Open Call)** 기법이라고 하며, 동기화 영역을 최소화하는 대표적인 방법이다.

#### ✅ CopyOnWriteArrayList 활용 (더 나은 해결책)
```java
private final List<SetObserver<E>> observers = new CopyOnWriteArrayList<>();

public void addObserver(SetObserver<E> observer) {
    observers.add(observer);
}

public boolean removeObserver(SetObserver<E> observer) {
    return observers.remove(observer);
}

private void notifyElementAdded(E element) {
    for (SetObserver<E> observer : observers) {
        observer.added(this, element); // 동기화 없이 안전한 순회 가능
    }
}
```
- `CopyOnWriteArrayList`는 내부 배열을 수정할 때마다 새로운 복사본을 생성하므로, 순회할 때 락을 사용할 필요가 없다.
- 수정이 자주 발생하지 않는 관찰자 패턴에서는 최적의 선택이다.

<br>

### 4. **동기화를 최소화하는 전략**
#### 1️⃣ **락의 범위를 최소화하라**
- 공유 데이터에 대한 접근이 꼭 필요한 부분에서만 동기화를 적용한다.
- 불필요한 코드까지 동기화하면 성능이 저하된다.

#### 2️⃣ **동시성 컬렉션을 활용하라**
- `CopyOnWriteArrayList`, `ConcurrentHashMap` 등 자바의 동시성 컬렉션을 사용하면 명시적인 동기화 없이도 안전한 동작을 보장할 수 있다.

#### 3️⃣ **락 분할(Lock Splitting)과 락 스트라이핑(Lock Striping)을 고려하라**
- 하나의 락이 여러 개의 독립적인 데이터를 보호하는 경우, 각각 별도의 락을 사용하면 성능이 향상될 수 있다.
- `ConcurrentHashMap`은 락 스트라이핑 기법을 활용해 높은 동시성을 제공한다.

#### 4️⃣ **불변 객체(Immutable Object)를 적극 활용하라**
- 동기화가 필요 없는 불변 객체를 사용하면 동시성 문제 자체를 원천적으로 방지할 수 있다.

<br>

### 💡 **핵심 정리**
- **동기화는 최소한으로 적용하되, 필요한 경우에는 적절한 기법을 활용하라.**
- **동기화된 블록 내부에서 외부 메서드를 호출하지 말고, 열린 호출(Open Call) 패턴을 사용하라.**
- **`CopyOnWriteArrayList` 같은 동시성 컬렉션을 활용해 성능과 안정성을 높여라.**
- **교착상태를 피하기 위해 락의 순서를 고려하고, 불변 객체 사용을 권장한다.**

