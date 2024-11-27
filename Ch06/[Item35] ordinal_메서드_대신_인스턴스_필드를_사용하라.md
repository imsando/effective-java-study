# ordinal 메서드와 열거 타입의 설계에 대하여

## ⛳️ 목표

- `ordinal` 메서드의 사용을 피해야하는 이유에 대해 알아보자.

<br>

## 📄 핵심 요약

### **ordinal이란?**

- **`ordinal` 메서드**는 열거 타입(Enum)의 상수가 **정의된 순서**를 반환한다.
- 이 메서드는 내부 구현에서 **EnumSet, EnumMap**과 같은 범용 자료구조를 지원하기 위한 용도로 설계되었다.

```java
public enum Day {
    MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY, SUNDAY;
    
    public static void main(String[] args) {
        System.out.println(Day.MONDAY.ordinal()); // 출력: 0
        System.out.println(Day.WEDNESDAY.ordinal()); // 출력: 2
    }
}
```

<br>

---

### **1. ordinal 메서드의 오용**

- **문제점** : `ordinal`을 열거 상수와 연결된 값으로 사용하는 경우 발생하는 유지보수 문제가 있다.

#### 잘못된 구현 예시
```java
public enum Ensemble {
    SOLO, DUET, TRIO, QUARTET, QUINTET, SEXTET, SEPTET, OCTET, NONET, DECTET;

    public int numberOfMusicians() {
        return ordinal() + 1; // 상수 순서로 음악가 수 계산
    }
}
```

#### 주요 문제

1. **상수 선언 순서 의존성**
    - 상수 순서를 변경하면 `numberOfMusicians`가 잘못된 값을 반환한다.
    - 예 : `SOLO`를 마지막으로 이동하면 10을 반환하게 된다.
2. **중복된 값 추가 불가**
    - 예 : 8명이 연주하는 **DOUBLE_QUARTET** 추가가 불가능하다.
3. **값을 비울 수 없음**
    - 예 : 12명의 **TRIPLE_QUARTET** 추가 시 11명짜리 더미 상수가 필요하다.

<br>

---

### **2. 올바른 설계 : 필드에 값을 저장**

- **해결책** : 열거 타입 상수에 연결된 값을 `ordinal`이 아닌 **인스턴스 필드**에 저장한다.

#### 개선된 구현 예시
```java
public enum Ensemble {
    SOLO(1), DUET(2), TRIO(3), QUARTET(4), QUINTET(5),
    SEXTET(6), SEPTET(7), OCTET(8), DOUBLE_QUARTET(8),
    NONET(9), DECTET(10), TRIPLE_QUARTET(12);

    private final int numberOfMusicians;

    Ensemble(int size) {
        this.numberOfMusicians = size; // 연주자 수를 필드에 저장
    }

    public int numberOfMusicians() {
        return numberOfMusicians;
    }
}
```

<br>

---

### **3. ordinal 메서드 사용 규칙**

- `ordinal`은 **내부적 용도**로만 사용해야 한다.
- **API 문서 권장사항** : 대부분의 프로그래머는 `ordinal`을 사용할 일이 없다.

#### 권장 사용 사례
- **EnumSet, EnumMap**과 같은 **열거 기반 자료구조**에 활용한다.
- 상수의 **순서**가 필요한 상황에서만 사용한다.

```java
EnumSet<Ensemble> smallGroups = EnumSet.range(Ensemble.SOLO, Ensemble.QUARTET);
System.out.println(smallGroups); // 출력: [SOLO, DUET, TRIO, QUARTET]
```

<br>

---

### **4. 추가 설계 권장사항**

- 열거 타입에 **부가 정보를 추가**하여 더 풍부한 표현을 지원한다.

#### 확장된 구현 예시
```java
public enum Ensemble {
    SOLO(1, "Solo Performance"), DUET(2, "Two Performers"), 
    TRIO(3, "Three Performers"), QUARTET(4, "Four Performers");

    private final int numberOfMusicians;
    private final String description;

    Ensemble(int size, String description) {
        this.numberOfMusicians = size;
        this.description = description;
    }

    public int numberOfMusicians() {
        return numberOfMusicians;
    }

    public String getDescription() {
        return description;
    }
}
```

#### 사용 예시
```java
public static void main(String[] args) {
    System.out.println(Ensemble.TRIO.getDescription()); // 출력: Three Performers
}
```

<br>

---

> 💡 **핵심 정리**
>
> 1. `ordinal` 메서드는 상수의 순서에 의존하는 동작을 만들면 안 된다.
> 2. 상수와 연결된 값은 **인스턴스 필드**를 사용하여 관리한다.
> 3. `ordinal`은 내부적으로만 사용하며, 범용 자료구조와 같은 특별한 용도에서만 활용한다.
> 4. 유지보수와 확장성을 고려해 열거 타입 설계를 진행해야한다.