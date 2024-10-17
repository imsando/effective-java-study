# toString을 항상 재정의하라

## ⛳️ 목표

- `toString` 재정의 했을 때 장점에 대해 알아보자

<br>

## 📄 핵심 요약

***toString 재정의***

- `Object` 클래스에서 기본 제공되는 `toString` 메서드이다
- 클래스 이름과 해시코드를 16진수로 반환한다
- 예시: `PhoneNumber@adbbd`와 같은 출력은 간결하지만 유익하지 않다

```java
PhoneNumber phoneNumber = new PhoneNumber();
System.out.println(phoneNumber);  // PhoneNumber@adbbd
```

- 위처럼 기본 출력보다는 실제 정보를 담은 `707-867-5309` 로 출력되는 것이 의미 있기 때문에 재정의가 필요하다

<br>

### 1. toString의 중요성

`toString` 메서드를 재정의하면 디버깅과 로깅이 쉬워진다

> - **① 디버깅과 로깅**: 오류 메시지나 로그에서 객체의 유용한 정보를 제공하여 문제 해결에 도움이 된다.
> - **② 문자열 연결 시 유용**: `toString`은 객체를 문자열로 자동 변환하므로 `+` 연산이나 `printf` 등에서도 유용하게 사용된다.

<br>

### 2. toString의 좋은 사용예시

`toString`은 그 객체가 담고 있는 주요 정보를 포함하는 것이 좋다
예를 들어, 전화번호 객체라면 실제 전화번호를 반환하는 것이 유익하다.

> - **⓵ 주요 정보 제공하기**: 전화번호, 좌표값, 날짜 등 명확한 정보를 반환.
> - **② 상태가 복잡할 경우 요약 정보 제공하기**: 큰 객체일 경우 요약된 정보로 표현.

```java
//⓵ 주요 정보 제공하기
@Override
public String toString() {
        return countryCode + "-" + areaCode + "-" + number;  // 전화번호의 주요 정보를 반환
        }

//② 상태가 복잡할 경우 요약 정보 제공하기
@Override
public String toString() {
        return "Order ID: " + orderId + ", Product: " + product + ", Price: $" + price;  // 요약된 정보 제공
        }
```

<br>

### 3. 포맷 문서화

- `toString` 메서드의 반환 포맷을 명시적으로 문서화할지 결정해야 한다
- 예를 들어 전화번호나 좌표 클래스처럼 표준적인 형태의 값을 반환하는 경우 포맷을 문서화하는 것이 좋다

```java
@Override
public String toString() {
    return String.format("%03d-%03d-%04d", areaCode, prefix, lineNumber);
}
```

- 만약 포맷을 명시하면 해당 포맷에 대한 의존성이 생기므로 이후 포맷 변경이 어렵다

🏷️ **NOTE**

> 포맷을 명시하지 않으면 향후 릴리스에서 더 많은 정보를 추가하거나 포맷을 개선할 수 있는 유연성을 얻게 된다

<br>

### 4. toString과 상호작용할 수 있는 API 제공하자

- `toString`에서 반환된 정보를 파싱하지 않고도 주요 정보를 얻을 수 있도록 접근자를 제공하는 것이 중요하다
- 이를 통해 `toString`의 반환값을 직접 사용하지 않고도 원하는 데이터를 안전하게 얻을 수 있다

```java
public int getAreaCode() {
    return areaCode;
}

public int getPrefix() {
    return prefix;
}
```

<br>

### 5. 재정의가 불필요한 경우

- 대부분의 열거 타입(enum)은 이미 `toString`이 적절히 구현되어 있어 별도로 재정의할 필요가 없다
- 정적 유틸리티 클래스나 자동 생성된 값 클래스는 `toString`을 자동으로 생성할 수 있으며 필요 시 이를 재정의하는 것이 좋다

<br>

### 6. 정리

- **모든 구체 클래스에서 `Object`의 `toString`을 재정의하자**
- 유용한 정보를 명확하고 간결하게 제공하여 클래스의 사용성을 높이고 시스템 디버깅을 쉽게 하자
