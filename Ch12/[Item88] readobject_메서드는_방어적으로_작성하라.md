## ⛳️ 목표
- 역직렬화 과정에서 불변성이 깨지는 문제를 방지하는 방법을 알아보자.

---

## 📄 핵심 요약

### **직렬화(Serialization)란?**
- 객체를 **바이트 스트림**으로 변환하여 저장하거나 전송하는 기술이다.
- `ObjectOutputStream`을 사용하여 객체를 파일이나 네트워크로 전송할 수 있다.
- 반대로, `ObjectInputStream`을 사용하여 객체를 **역직렬화(Deserialization)** 할 수 있다.

### **readObject란?**
- readObject(ObjectInputStream s)는 객체를 **역직렬화(Deserialization)** 할 때 호출되는 메서드이다.

---
<br>

### **1. readObject의 위험성**
- `readObject`는 사실상 **"바이트 스트림을 받는 또 다른 생성자"** 역할을 한다.
- 따라서 기존 생성자에서 수행하는 **불변성 유지 및 방어적 복사**가 반드시 필요하다.
- 그렇지 않으면 **악의적인 직렬화 데이터를 이용한 공격**이 가능해진다.

<br>

### **2. readObject의 보안 취약점과 해결 방법**

#### **❌ 잘못된 readObject 구현 (취약점 존재)**
```java
private void readObject(ObjectInputStream s) throws IOException, ClassNotFoundException {
    s.defaultReadObject();  // 기본 역직렬화 수행
}
```
- 기본 역직렬화만 수행하면 **불변성이 깨질 위험**이 있다.
- 공격자가 `ObjectInputStream`을 조작하여 내부 필드 값을 변경할 수 있다.

#### **✅ 방어적 복사를 적용한 안전한 readObject**
```java
private void readObject(ObjectInputStream s) throws IOException, ClassNotFoundException {
    s.defaultReadObject();  // 기본 역직렬화 수행
    
    // 방어적 복사 수행
    start = new Date(start.getTime());
    end = new Date(end.getTime());

    // 불변식 검사 (시작 시간이 종료 시간보다 늦으면 예외 발생)
    if (start.compareTo(end) > 0) {
        throw new InvalidObjectException("시작 시간이 종료 시간보다 늦을 수 없습니다.");
    }
}
```
✔ **해결 방법 요약:**
1. `defaultReadObject()` 호출 후, **가변 객체(Date)를 방어적으로 복사**한다.
2. **불변식 검사를 수행하여** 유효하지 않은 객체 생성 시 예외를 던진다.

<br>

### **3. readObject 작성 시 주의할 점**
1️⃣ **private 필드의 가변 객체를 방어적으로 복사하라.**
- 예 : `Date`, `ArrayList` 등 내부 상태가 변할 수 있는 객체는 **복사본을 만들어 저장**해야 한다.

2️⃣ **불변식을 검사하여 유효하지 않은 객체 생성 시 예외를 던져라.**
- `InvalidObjectException`을 사용해 객체 생성 자체를 막을 수 있다.

3️⃣ **readObject 내부에서 절대 재정의 가능한 메서드를 호출하지 말라.**
- `readObject` 실행 중 하위 클래스의 오버라이딩 메서드가 호출되면 예측할 수 없는 동작이 발생할 수 있다.

---

## 💡 **핵심 정리**
- `readObject`는 **새로운 생성자**와 같으므로, **불변성을 유지할 수 있도록 방어적으로 작성해야 한다.**
- 가변 객체는 반드시 **방어적 복사(Defensive Copy)** 를 수행해야 한다.
- `InvalidObjectException`을 활용해 **불변식 위반을 사전에 차단**하라.
- 재정의 가능한 메서드를 호출하지 않도록 주의하라.