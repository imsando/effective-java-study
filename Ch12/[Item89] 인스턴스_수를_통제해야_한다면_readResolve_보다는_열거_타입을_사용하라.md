## ⛳️ 목표

- 인스턴스 수를 통제해야 할 때, `readResolve`보다 열거 타입을 사용하는 것이 왜 더 좋은지 알아보자.

<br>

## 📄 핵심 요약

#### **직렬화된 싱글턴의 문제점**
- `Serializable`을 구현한 싱글턴 클래스는 역직렬화 시 새로운 인스턴스가 생성될 위험이 있다.
- `readResolve()`를 사용하면 기존 인스턴스를 반환할 수 있지만, 완전한 해결책은 아니다.

#### **readResolve()란?**
- 역직렬화 시 새로운 객체 대신 기존의 싱글턴 인스턴스를 반환하는 메서드이다.
- 하지만 역직렬화 과정에서 일시적으로 새로운 객체가 생성되는 문제점이 있다.
- 모든 참조 타입 필드를 `transient`로 선언해야만 안전성을 확보할 수 있다.

#### **readResolve()를 우회하는 공격 기법**
- `readResolve()`가 실행되기 전에 싱글턴 객체의 참조를 가로채는 `ElvisStealer` 같은 공격이 가능하다.
- 직렬화 스트림을 조작하면 `readResolve()` 실행 이전에 생성된 객체를 활용할 수 있다.
- 따라서 완벽한 방어책이 되지 못하며, 보안 취약점이 존재한다.

#### **열거 타입(Enum) 기반 싱글턴의 장점**
- 자바가 제공하는 `Enum`은 기본적으로 직렬화가 안전하다.
- 리플렉션 공격에도 강하며, 추가적인 코드 없이 싱글턴을 보장할 수 있다.
- 따라서 싱글턴을 구현할 때는 `Enum`을 사용하는 것이 가장 안전한 방법이다.

---
<br>

### 1. **직렬화 가능한 싱글턴 클래스의 문제점**

#### **기본적인 싱글턴 구현 (문제 발생 가능)**
```java
public class Elvis implements Serializable {
    public static final Elvis INSTANCE = new Elvis();
    private Elvis() {}
    private String[] favoriteSongs = {"Hound Dog", "Heartbreak Hotel"};
    
    public void printFavorites() {
        System.out.println(Arrays.toString(favoriteSongs));
    }
    
    private Object readResolve() {
        return INSTANCE;
    }
}
```
- `readResolve()`를 사용했지만 역직렬화 중 새로운 객체가 생성되는 문제는 여전히 존재한다.

---

### 2. **ElvisStealer 공격 기법**

#### **싱글턴을 훔치는 도둑 클래스**
```java
public class ElvisStealer implements Serializable {
    static Elvis impersonator;
    private Elvis payload;
    
    private Object readResolve() {
        impersonator = payload;
        return new String[]{"A Fool Such as I"};
    }
}
```
- `ElvisStealer`를 이용하면, `readResolve()` 실행 이전의 인스턴스 참조를 가로챌 수 있다.

<br>

### 3. **안전한 해결책: `Enum` 싱글턴**

#### **열거 타입 기반 싱글턴 (가장 안전한 방법)**
```java
public enum Elvis {
    INSTANCE;
    
    private String[] favoriteSongs = {"Hound Dog", "Heartbreak Hotel"};
    
    public void printFavorites() {
        System.out.println(Arrays.toString(favoriteSongs));
    }
}
```
- `Enum`을 사용하면 역직렬화 공격과 리플렉션 공격을 모두 차단할 수 있다.
- 자바가 `Enum`을 직렬화할 때 인스턴스가 하나만 유지되도록 보장한다.

---

### 💡 **핵심 정리**
- 싱글턴을 구현할 때 `readResolve()`를 사용하는 것은 완벽한 해결책이 아니다.
- 직렬화된 객체가 `readResolve()` 이전에 임시로 생성될 수 있어, 공격 가능성이 존재한다.
- 가능하면 열거 타입(`Enum`)을 사용하여 싱글턴을 구현하는 것이 가장 안전하다.

