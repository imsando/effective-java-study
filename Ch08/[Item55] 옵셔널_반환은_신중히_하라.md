# 옵셔널 반환은 신중히 하라

## ⛳️ 목표

- `Optional` 을 사용할 때 주의사항에 대해 알아보자.

<br>

## 📄 핵심 요약

### 옵셔널(Optional)이란?

- **null** 대신 사용 가능한 컨테이너 객체로, 값이 있거나 없음을 표현한다.
- 값이 있을 경우 해당 값을, 없을 경우 '빈 값'을 안전하게 처리할 수 있도록 설계되었다.
- 최대 1개의 값을 가질 수 있는 **불변** 컬렉션이다.

---

### 1. **옵셔널 사용의 장점**

#### 1️⃣ null 반환과 예외의 대안
- 예외는 진짜 예외 상황에서만 사용해야 한다.
- null 반환은 별도의 null 체크 코드가 필요하며, 실수로 놓칠 경우 **NullPointerException** 발생 위험이 있다.
- 옵셔널은 값이 없을 경우 빈 상태를 안전하게 표현하여 오류 가능성을 줄인다.

#### 2️⃣ 호출자에게 의도를 명확히 전달
- 메서드 반환 타입에 `Optional<T>`를 사용하면 반환값이 없을 수도 있음을 API 사용자에게 명확히 알린다.
- 호출자는 반환값이 없을 경우 기본값 설정, 예외 처리 등 적절한 대처를 할 수 있다.

<br>
<br>

### 2. **옵셔널 활용 방법**

#### 1️⃣ 기본값 설정
```java
String lastWordInLexicon = max(words).orElse("N/A");
```

#### 2️⃣ 예외 던지기
```java
Toy myToy = max(toys).orElseThrow(TemperTantrumException::new);
```

#### 3️⃣ 값이 항상 존재한다고 가정
```java
Element lastNobleGas = max(elements.NOBLE_GASES).get();
```

#### 4️⃣ 값이 필요할 때만 생성
```java
String expensiveDefault = max(words).orElseGet(() -> calculateExpensiveDefault());
```

#### 5️⃣ 옵셔널 필터링 및 변환
```java
ph.parent()
  .map(h -> String.valueOf(h.pid()))
  .orElse("N/A");
```

<br>
<br>

### 3. **스트림과의 조합**

#### 1️⃣ 옵셔널 스트림 변환 (Java 8)
- `Optional`의 값을 꺼내 스트림에 담는 방법:
```java
streamOfOptionals
  .filter(Optional::isPresent)
  .map(Optional::get);
```

#### 2️⃣ 옵셔널 스트림 변환 (Java 9)
- `Optional.stream()`을 사용하여 간단히 변환:
```java
streamOfOptionals
  .flatMap(Optional::stream);
```

<br>
<br>

### 4. **옵셔널 반환 기준**

#### 1️⃣ 옵셔널을 반환해야 하는 경우
- 반환값이 없을 수 있으며, 클라이언트가 이를 명시적으로 처리해야 하는 경우.

#### 2️⃣ 옵셔널을 반환하지 말아야 하는 경우
- 컬렉션, 배열, 스트림 등 컨테이너 타입은 옵셔널로 감싸지 않는다.
    - 예 : 빈 `List`를 반환하는 것이 `Optional<List>`를 반환하는 것보다 낫다.

#### 3️⃣ 성능 민감한 상황
- 옵셔널은 객체 할당과 초기화 비용이 추가로 발생한다.
- 성능이 중요한 메서드에서는 null 반환이나 예외 처리 사용을 고려할 수 있다.

<br>
<br>

### 5. **옵셔널 사용 시 주의점**

#### 1️⃣ 필드에 옵셔널 사용 자제
- 옵셔널은 메서드 반환 타입으로만 사용하는 것이 일반적이다.
- 필수 필드와 선택적 필드를 분리하는 것이 더 적합할 수 있다.

#### 2️⃣ 맵, 컬렉션의 키/값으로 사용 금지
- 키/값이 없는 상황과 빈 옵셔널로 구분해야 하므로 혼란을 줄 수 있다.

<br>
<br>

## 💡 핵심 정리

1. 옵셔널은 반환값이 없을 가능성을 명확히 전달하며 오류를 줄인다.
2. 성능이 중요한 경우, 옵셔널 대신 null 반환이나 예외를 던질 수도 있다.
3. 컨테이너 타입과 필드에는 옵셔널 사용을 지양한다.