# 비트필드와 EnumSet에 대하여

## ⛳️ 목표

- **비트 필드** 대신 **EnumSet**을 사용해야 하는 이유에 대해 알아보자.

<br>

## 📄 핵심 요약

### **비트 필드란?**

- **비트 필드**는 상수를 정수 값으로 정의하고, 비트 연산을 통해 집합 연산을 수행하는 패턴이다.
- 예) 굵은 글씨체(BOLD)와 기울임 글씨체(ITALIC)를 함께 사용하려면, 두 상수의 비트를 OR 연산으로 결합한다.

```java
public class Text {
    public static final int STYLE_BOLD = 1 << 0;       // 1
    public static final int STYLE_ITALIC = 1 << 1;    // 2
    public static final int STYLE_UNDERLINE = 1 << 2; // 4
    public static final int STYLE_STRIKETHROUGH = 1 << 3; // 8

    // 스타일 적용 메서드
    public void applyStyles(int styles) {
        // 비트 필드로 스타일 처리
    }
}

// 사용 예시
text.applyStyles(STYLE_BOLD | STYLE_ITALIC);
```

<br>

---

### **1. 비트 필드의 단점**

1. **읽기 어렵다**
    - 예 : 비트 필드 값 `3`(STYLE_BOLD | STYLE_ITALIC)을 보면 어떤 스타일이 적용되었는지 해석하기 어렵다.

2. **확장성 문제**
    - 상수를 추가하려면 미리 예측 가능한 비트 수를 사용해야 한다.
    - 예 : 32비트 정수(`int`)로는 최대 32개의 상수만 표현 가능하다.

3. **반복 작업의 어려움**
    - 비트 필드에서 각각의 원소를 순회하려면 직접 비트 연산을 해야 한다.

4. **타입 안전성 부족**
    - 비트 필드는 정수형 상수로 표현되므로, 잘못된 값이 사용될 가능성이 있다.

<br>

---

### **2. EnumSet : 현대적 대안**

- **EnumSet**은 열거 타입 상수로 이루어진 집합을 효율적으로 다루기 위한 클래스다.
- 내부적으로 **비트 벡터**로 구현되었지만, 더 읽기 쉽고 안전하며, 열거 타입의 모든 장점을 제공한다.

#### 개선된 구현 예시
```java
public class Text {
    public enum Style { BOLD, ITALIC, UNDERLINE, STRIKETHROUGH }

    // 스타일 집합 적용
    public void applyStyles(Set<Style> styles) {
        // EnumSet을 사용하는 코드
    }
}

// 사용 예시
text.applyStyles(EnumSet.of(Text.Style.BOLD, Text.Style.ITALIC));
```

---

### **3. EnumSet의 장점**

1. **가독성**
    - 집합을 `EnumSet.of()`로 표현하므로, 어떤 상수가 포함되어 있는지 명확히 알 수 있다.
    - 예) `EnumSet.of(Style.BOLD, Style.ITALIC)`는 `STYLE_BOLD | STYLE_ITALIC`보다 직관적이다.

2. **효율성**
    - 내부적으로 **비트 벡터**로 동작해 비트 필드와 유사한 성능을 제공한다.
    - 최대 64개의 상수까지 Long 하나로 처리 가능하다.

3. **타입 안전성**
    - `Set<Style>`로 타입을 제한하므로, 잘못된 타입을 사용할 수 없다.

4. **확장성**
    - 비트 필드처럼 비트 수를 미리 계산할 필요 없이, 열거 타입을 확장할 수 있다.

<br>

---

### **4. EnumSet과 인터페이스 활용**

- 메서드에서 EnumSet 대신 **Set 인터페이스**로 받으면, 다른 `Set` 구현체도 전달 가능하다.
- 이 설계는 **유연성**을 높인다.

```java
public void applyStyles(Set<Style> styles) {
    // EnumSet이 아닌 다른 Set 구현체도 허용
}
```

---

### **5. 불변 EnumSet 만들기**

- EnumSet은 기본적으로 변경 가능하다.
- 불변 집합을 원할 경우, `Collections.unmodifiableSet`로 감싸서 사용한다.

```java
Set<Text.Style> immutableStyles = Collections.unmodifiableSet(EnumSet.of(Text.Style.BOLD, Text.Style.ITALIC));
```

<br>

---

### **6. EnumSet의 단점**

- (자바 9 기준) **불변 EnumSet**을 직접적으로 지원하지 않는다.
    - 다만, 향후 릴리스에서 개선될 가능성이 있다.

<br>

---

> ## 💡 핵심 정리
>
> 1. **비트 필드의 단점**
> - 읽기 어려움, 확장성 부족, 타입 안전성 부족.
> 2. **EnumSet의 장점**
> - **가독성**과 **타입 안전성**을 보장하며, 비트 필드 수준의 성능 제공.
> 3. **설계 팁**
> - EnumSet을 사용하되, 인터페이스(Set)로 매개변수를 받는 유연한 설계를 하자.
> 4. **결론**
> - 집합으로 열거 타입을 사용해야 한다면, 비트 필드 대신 EnumSet을 활용하자!