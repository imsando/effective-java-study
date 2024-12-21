# 메서드 시그니처를 신중히 설계하라

## ⛳️ 목표

- 메서드 시그니처 설계를 할 때 주의해야 할 사항을 알아보자.

<br>

## 📄 핵심 요약

### **메서드 시그니처 구성**
1. **메서드 이름**
```java
void print();  // 메서드 이름: print
```
2. **매개변수 목록**
```java
void print(String message);  // 매개변수: (String message)
```

---

### 1. **편의 메서드 설계**
- 필요한 만큼만 제공 : 과도한 편의 메서드는 클래스 복잡성을 높이고 유지보수를 어렵게 만든다.
- 약칭 메서드는 자주 사용되는 경우에만 제공한다.
- 확신이 서지 않으면 편의 메서드를 추가하지 않는 것을 권장한다.

<br>

### 2. **매개변수 목록 설계**

#### 1️⃣ 매개변수 수 제한
- 4개 이하로 유지 : 매개변수가 많을수록 사용자가 기억하기 어렵다.
- 같은 타입의 매개변수가 연달아 나올 경우, 순서 실수로 인한 오류 가능성이 높아진다.

#### 2️⃣ 긴 매개변수 목록 해결 방법
⓵ 메서드 분리
>    - 메서드를 여러 개로 쪼개, 각 메서드가 매개변수 부분집합만 처리한다.
>    - 예 : `List.subList()`와 `indexOf()` 메서드로 부분리스트의 원소 검색 구현한다.
>   ```java
>   List<String> list = List.of("A", "B", "C", "D");
>   int index = list.subList(1, 3).indexOf("C");
>   ```

② 도우미 클래스 사용
>    - 연관된 매개변수들을 묶어 클래스로 정의한다.
>    - 예 : 카드게임에서 숫자(`rank`)와 무늬(`suit`)를 묶는 클래스
>   ```java
>   public class Card {
>       private final Rank rank;
>       private final Suit suit;
>
>       public Card(Rank rank, Suit suit) {
>           this.rank = rank;
>           this.suit = suit;
>       }
>   }
>   ```

⓷ 빌더 패턴 활용
>    - 매개변수가 많거나 일부가 선택적인 경우에 유용하다.
>    - 객체 생성 방식을 메서드 호출로 확장한다.
>   ```java
>   Thermometer thermometer = new Thermometer.Builder()
>           .scale(TemperatureScale.CELSIUS)
>           .precision(2)
>           .build();
>   ```

<br>

### 3. **매개변수 타입 설계**

#### 1️⃣ 클래스 대신 인터페이스 사용
- 매개변수 타입으로 구현 클래스 대신 상위 인터페이스를 사용한다.
>    - `HashMap` 대신 `Map`을 사용해 다양한 구현체 지원한다.
>  ```java
>  void process(Map<String, String> data);
>  ```

#### 2️⃣ boolean 대신 열거 타입 사용
- boolean 매개변수의 의미 명확화
>    - `TemperatureScale.CELSIUS`와 같은 열거 타입으로 가독성을 높인다.
>  ```java
>  Thermometer.newInstance(TemperatureScale.CELSIUS);
>  ```

<br>

### 4. **직교성 높이기**

#### 1️⃣ 직교성의 정의
- 기능을 독립적으로 분리하여 기능이 겹치지 않도록 설계한다.
- 예: `List`의 `subList`와 `indexOf`를 조합해 목적 달성하듯

#### 2️⃣ 직교성의 이점
- 코드 간결화
    - 기본 기능을 조합해 복잡한 작업을 수행한다.
    - 중복 제거로 유지보수성과 테스트 용이성이 증가한다.
- 사용자 경험 개선
    - 직교적인 설계는 API를 배우고 사용하기 쉽게 만든다.

---

### 5. **구체적인 설계 예제**

#### 1️⃣ 부분리스트 인덱스 검색
- 직교성을 고려한 API
  ```java
  List<String> subList = list.subList(0, 3);
  int index = subList.indexOf("target");
  ```

#### 2️⃣ 도우미 클래스로 매개변수 묶기
- 카드게임 설계 예
  ```java
  public class Card {
      private final Rank rank;
      private final Suit suit;

      public Card(Rank rank, Suit suit) {
          this.rank = rank;
          this.suit = suit;
      }
  }
  ```

---

### 6. **열거 타입의 활용**

#### 1️⃣ 불리언 대신 열거 타입
- 열거 타입으로 코드 가독성과 확장성 확보
  ```java
  public enum TemperatureScale {
      FAHRENHEIT, CELSIUS, KELVIN;
  }
  ```

#### 2️⃣ 코드 예제
- 열거 타입 활용
  ```java
  Thermometer thermometer = Thermometer.newInstance(TemperatureScale.CELSIUS);
  ```

---

> 💡 **핵심 정리**
>
> 1. 메서드 시그니처는 간결하고 이해하기 쉽게 설계하라.
> 2. 매개변수는 적을수록 좋으며, 필요하다면 도우미 클래스나 빌더 패턴을 활용하라.
> 3. 인터페이스와 열거 타입을 적극 활용해 유연성과 가독성을 높여라.
> 4. 직교성을 높인 설계는 유지보수성과 확장성을 크게 개선한다.