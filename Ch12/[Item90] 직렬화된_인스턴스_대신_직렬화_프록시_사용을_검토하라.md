## ⛳️ 목표
- 직렬화 프록시 패턴을 활용하여 직렬화의 보안 문제를 방지하는 방법을 알아보자.

<br>  

## 📄 핵심 요약

#### **직렬화(Serialization)란?**
- 객체의 상태를 바이트 스트림으로 변환하여 저장하거나 네트워크를 통해 전송할 수 있도록 하는 과정이다.
- `Serializable` 인터페이스를 구현하여 객체를 직렬화할 수 있다.

#### **직렬화의 문제점**
- **1. 생성자 이외의 방식으로 인스턴스를 생성할 수 있음**
    - 직렬화된 데이터를 역직렬화하면 생성자를 거치지 않고 객체가 만들어진다.
    - 이로 인해 **불변식(invariant)이 깨질 위험**이 있다.
- **2. 보안 문제 발생 가능**
    - 공격자가 조작된 직렬화 데이터를 이용하여 내부 필드를 탈취할 수 있다.
    - 가짜 바이트 스트림 공격을 통해 시스템을 위협할 수도 있다.

---
<br>

### 1. **직렬화 프록시 패턴(Serialization Proxy Pattern)**
- 직렬화 시, 기존 클래스 대신 **프록시 객체를 사용하여 직렬화하는 기법**
- **프록시 객체를 통해 역직렬화 후, 정상적인 생성자를 거쳐 인스턴스를 복원**

#### 1️⃣ **직렬화 프록시 설계 방법**
- 바깥 클래스의 논리적 상태를 저장할 **`private static` 중첩 클래스를 생성** 한다.
- **이 중첩 클래스가 바깥 클래스의 직렬화 프록시 역할**을 한다.
- 프록시는 **`Serializable`을 구현**하며, 바깥 클래스의 데이터를 복사하는 생성자를 가진다.

```java
private static class SerializationProxy implements Serializable {
    private final Date start;
    private final Date end;

    SerializationProxy(Period p) {
        this.start = p.start;
        this.end = p.end;
    }

    private static final long serialVersionUID = 234098243823485285L;
}
```

#### 2️⃣ **writeReplace 메서드 추가**
- 직렬화 시 프록시 객체를 대신 반환하도록 한다.
- 이로 인해 바깥 클래스의 직렬화된 인스턴스를 직접 생성할 수 없게 된다.

```java
private Object writeReplace() {
    return new SerializationProxy(this);
}
```

#### 3️⃣ **readObject 메서드를 방어적으로 추가**
- 바깥 클래스에서 역직렬화 공격을 차단하기 위해 사용한다.
- 프록시가 없으면 역직렬화를 막는다.

```java
private void readObject(ObjectInputStream stream) throws InvalidObjectException {
    throw new InvalidObjectException("프록시가 필요합니다.");
}
```

#### 4️⃣ **readResolve 메서드 추가**
- 역직렬화 시, 프록시 객체를 원래 객체로 변환한다.

```java
private Object readResolve() {
    return new Period(start, end); // 생성자를 사용하여 복원
}
```

<br>

### 2. **직렬화 프록시 패턴의 장점**
✅ **불변성 보장**
- 직렬화된 데이터가 생성자를 거쳐야만 인스턴스로 변환됨 → 불변성이 유지됨

✅ **가짜 바이트 스트림 공격 방어**
- 직접적인 직렬화된 객체 생성을 방지하여 공격을 원천 차단

✅ **내부 필드 탈취 공격 방어**
- 프록시 객체만 저장하므로, 불필요한 필드 노출을 방지

✅ **클래스 변경에도 유연**
- 기존 객체 대신 프록시 객체를 직렬화하기 때문에, 클래스의 내부 구조가 변경되어도 역직렬화 가능

<br>

### 3. **직렬화 프록시 패턴의 한계**
⚠️ **확장 가능한 클래스에는 적용할 수 없음**
- `final`이 아닌 클래스라면 하위 클래스에서 상속받아 사용하기 어렵다.

⚠️ **순환 참조가 있는 객체에는 적용할 수 없음**
- 역직렬화 시, 아직 생성되지 않은 객체를 참조할 수 있어 `ClassCastException` 발생할 수 있다.

⚠️ **성능 저하 가능성**
- `readObject()`에서 직접 필드 값을 복사하는 방법보다 약 14% 정도 느릴 수 있다.

---
<br>

### 💡 **핵심 정리**
- 제3자가 확장할 수 없는 클래스라면 직렬화 프록시 패턴을 적극적으로 사용하자.
- 불변성을 유지하면서도 보안 위협을 방지하는 가장 안전한 직렬화 방법이다.