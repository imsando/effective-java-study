# 프로그램 요소의 접근성은 가능한 한 최소한으로 하라

## ⛳️ 목표

- public API로 설계는 최소화로 하는 방법과 그 이유에 대해서 알아보자

<br>

## 📄 핵심 요약

***API 란?***

- Application Programming Interface의 약자로 프로그램을 개발하고 사용하기 위한 인터페이스를 의미한다
- 클래스, 메서드, 인터페이스 등을 포함한다
- 즉, API는 프로그램이 다른 프로그램과 소통하는 방법을 제공한다]

***Public API 란?***

- 프로그램의 클래스나 메서드, 필드 등에서 외부에서 사용할 수 있는 공개된 인터페이스이다
- 즉, 외부에서 접근 가능한 API를 의미한다

```java
public class User {
    
    private final String name;
    private final int age;

    // Private 필드로 내부 상태는 외부에 노출되지 않음
    private String password;

    // 생성자: 객체 생성 시 필요한 필수 정보를 받음
    public User(String name, int age, String password) {
        this.name = name;
        this.age = age;
        this.password = password;
    }

    // Public API: 필수 정보만 제공
    public String getName() {
        return name;
    }

    public int getAge() {
        return age;
    }

    // Public API: 특정 동작을 캡슐화 (비밀번호 변경 메서드)
    public void changePassword(String newPassword) {
        if (validatePassword(newPassword)) {
            this.password = newPassword;
        } else {
            throw new IllegalArgumentException("Invalid password");
        }
    }

    // Private 메서드: 내부 구현 세부사항을 숨김 (Public API가 아님)
    private boolean validatePassword(String password) {
        // 비밀번호 유효성 검사 로직 (외부에 노출되지 않음)
        return password.length() >= 8;
    }

    // Public API: 객체의 상태를 요약해서 제공 (의도적으로 공개된 메서드)
    @Override
    public String toString() {
        return "User{name='" + name + "', age=" + age + "}";
    }
}
```

***굳이 차이점***

- Public API는 내부에서, Web API는 네트워크를 통해 외부와 상호작용하는 한다

<br>

### 정보은닉(캡슐화)의 장점

- 개발속도 증진 : 컴포넌트 병렬개발 가능하기 때문이다
- 관리비용 감소 : 컴포넌트 파악 속도 빠르고 교체 부담 적기 때문이다
- 성능 최적화  : 컴포넌트 최적화 가능하기 때문이다
- 재사용성 증대 : 컴포넌트 재사용성 높기 때문이다
- 유지보수성 증대 : 컴포넌트 수정 용이하기 때문이다

> 💡 접근제한자를 사용하는 것이 정보은닉의 핵심이다

<br>

### 1. 정보은닉의 원칙

- 모든 클래스와 멤버는 가능한 한 접근성을 좁게 설정해야 한다
- 소프트웨어가 제대로 동작하는 한 가장 낮은 접근 수준을 부여해야 한다

### 2. 접근 제한자의 중요성

- **접근 제한자**는 클래스, 메서드, 필드의 접근 범위를 제한하여 정보 은닉을 가능하게 하는 키워드이다.
- **Java의 주요 접근 제한자**
    - **private**: 해당 클래스 내에서만 접근 가능
    - **package-private (default)**: 같은 패키지 내에서만 접근 가능
    - **protected**: 같은 패키지 내 및 자식 클래스에서 접근 가능
    - **public**: 모든 클래스에서 접근 가능

```java
class Account {

    // private 필드로 정보 은닉
    private double balance;

    // public API로 제공되는 접근 메서드
    public double getBalance() {
        return balance;
    }

    // public API로 노출되는 비즈니스 로직
    public void deposit(double amount) {
        if (amount > 0) {
            balance += amount;
        }
    }

    // private 메서드로 내부 로직 감춤
    private void auditTransaction(double amount) {
        // 거래 감사 로직 (외부에 노출되지 않음)
    }
}
```

### 3. 주의할 점

- package-private 권장 
  > - 같은 패키지의 다른 클래스에서 접근해야 하는 멤버는 private에서 package-private으로만 설정할 것
  > - 권한을 풀어주게 된다면 시스템에서 컴포넌트 분해를 다시 고려해야 하기 때문이다
- Serializable 주의
  > - Serializable을 구현한 클래스에서는 private 필드도 의도치 않게 공개 API가 될 수 있으니 주의할 것
- protected 주의
  > - public 클래스에서 protected로 설정된 멤버는 공개 API로 간주되어 영원히 지원되어야 한다
  > - 이러한 멤버는 가급적 적게 사용하는 것이 좋다

- 예시코드

  ```java
  public class ExampleClass implements Serializable {
      private static final long serialVersionUID = 1L;
  
      // private 멤버 (외부에서 접근 불가)
      private String privateField = "Private Value";
  
      // package-private 멤버 (같은 패키지 내에서 접근 가능)
      String packagePrivateField = "Package-Private Value";
  
      // protected 멤버 (서브클래스에서만 접근 가능)
      protected String protectedField = "Protected Value";
  
      // 공개 API
      public String getPrivateField() {
          return privateField;
      }
  
      public void setPrivateField(String value) {
          this.privateField = value;
      }
  }
  ```

### 4. 접근성을 최소화하는 이유

- **정보 은닉 강화**: 외부에 불필요한 정보를 노출하지 않음으로써 객체의 무결성을 유지하고, 불필요한 의존성을 줄일 수 있다.
- **API 간결성**: 필요한 부분만 공개함으로써 API가 간결해지고 사용하기 쉬워진다.
- **유연성**: 내부 구현을 자유롭게 변경할 수 있고, 외부에 미치는 영향을 최소화할 수 있다.
- **보안성**: 중요한 데이터나 민감한 로직이 외부에 노출되지 않도록 보호할 수 있다.

<br>

### (+) 접근성 최소화의 장점 예시

```java
public class BankAccount {

    // Private 필드로 외부에 노출되지 않음
    private String accountNumber;
    private double balance;

    // Public 생성자: 필수 정보만 외부에서 설정
    public BankAccount(String accountNumber, double initialBalance) {
        this.accountNumber = accountNumber;
        this.balance = initialBalance;
    }

    // Public API: 잔액 조회
    public double getBalance() {
        return balance;
    }

    // Private 메서드로 내부 로직 감춤
    private boolean isBalanceSufficient(double amount) {
        return balance >= amount;
    }

    // Public API: 출금 기능 제공
    public void withdraw(double amount) {
        if (isBalanceSufficient(amount)) {
            balance -= amount;
        } else {
            throw new IllegalArgumentException("잔액 부족");
        }
    }
}
```

- 이 예시에서 `accountNumber`와 `balance`는 `private`로 선언되어 외부에서 직접 접근할 수 없고, `getBalance()`와 같은 필수 정보만 외부에 제공된다
- 내부 로직인 `isBalanceSufficient()`는 외부에 노출되지 않으며, 이를 통해# 프로그램 요소의 접근성은 가능한 한 최소한으로 하라

## ⛳️ 목표

- public API로 설계는 최소화로 하는 방법과 그 이유에 대해서 알아보자

<br>

## 📄 핵심 요약

***API 란?***

- Application Programming Interface의 약자로 프로그램을 개발하고 사용하기 위한 인터페이스를 의미한다
- 클래스, 메서드, 인터페이스 등을 포함한다
- 즉, API는 프로그램이 다른 프로그램과 소통하는 방법을 제공한다]

***Public API 란?***

- 프로그램의 클래스나 메서드, 필드 등에서 외부에서 사용할 수 있는 공개된 인터페이스이다
- 즉, 외부에서 접근 가능한 API를 의미한다

```java
public class User {
    
    private final String name;
    private final int age;

    // Private 필드로 내부 상태는 외부에 노출되지 않음
    private String password;

    // 생성자: 객체 생성 시 필요한 필수 정보를 받음
    public User(String name, int age, String password) {
        this.name = name;
        this.age = age;
        this.password = password;
    }

    // Public API: 필수 정보만 제공
    public String getName() {
        return name;
    }

    public int getAge() {
        return age;
    }

    // Public API: 특정 동작을 캡슐화 (비밀번호 변경 메서드)
    public void changePassword(String newPassword) {
        if (validatePassword(newPassword)) {
            this.password = newPassword;
        } else {
            throw new IllegalArgumentException("Invalid password");
        }
    }

    // Private 메서드: 내부 구현 세부사항을 숨김 (Public API가 아님)
    private boolean validatePassword(String password) {
        // 비밀번호 유효성 검사 로직 (외부에 노출되지 않음)
        return password.length() >= 8;
    }

    // Public API: 객체의 상태를 요약해서 제공 (의도적으로 공개된 메서드)
    @Override
    public String toString() {
        return "User{name='" + name + "', age=" + age + "}";
    }
}
```

***굳이 차이점***

- Public API는 내부에서, Web API는 네트워크를 통해 외부와 상호작용하는 한다

<br>

### 정보은닉(캡슐화)의 장점

- 개발속도 증진 : 컴포넌트 병렬개발 가능하기 때문이다
- 관리비용 감소 : 컴포넌트 파악 속도 빠르고 교체 부담 적기 때문이다
- 성능 최적화  : 컴포넌트 최적화 가능하기 때문이다
- 재사용성 증대 : 컴포넌트 재사용성 높기 때문이다
- 유지보수성 증대 : 컴포넌트 수정 용이하기 때문이다

> 💡 접근제한자를 사용하는 것이 정보은닉의 핵심이다

<br>

### 1. 정보은닉의 원칙

- 모든 클래스와 멤버는 가능한 한 접근성을 좁게 설정해야 한다
- 소프트웨어가 제대로 동작하는 한 가장 낮은 접근 수준을 부여해야 한다

### 2. 접근 제한자의 중요성

- **접근 제한자**는 클래스, 메서드, 필드의 접근 범위를 제한하여 정보 은닉을 가능하게 하는 키워드이다.
- **Java의 주요 접근 제한자**
  - **private**: 해당 클래스 내에서만 접근 가능
  - **package-private (default)**: 같은 패키지 내에서만 접근 가능
  - **protected**: 같은 패키지 내 및 자식 클래스에서 접근 가능
  - **public**: 모든 클래스에서 접근 가능

```java
class Account {

    // private 필드로 정보 은닉
    private double balance;

    // public API로 제공되는 접근 메서드
    public double getBalance() {
        return balance;
    }

    // public API로 노출되는 비즈니스 로직
    public void deposit(double amount) {
        if (amount > 0) {
            balance += amount;
        }
    }

    // private 메서드로 내부 로직 감춤
    private void auditTransaction(double amount) {
        // 거래 감사 로직 (외부에 노출되지 않음)
    }
}
```

### 3. 주의할 점

- package-private 권장
  > - 같은 패키지의 다른 클래스에서 접근해야 하는 멤버는 private에서 package-private으로만 설정할 것
  > - 권한을 풀어주게 된다면 시스템에서 컴포넌트 분해를 다시 고려해야 하기 때문이다
- Serializable 주의
  > - Serializable을 구현한 클래스에서는 private 필드도 의도치 않게 공개 API가 될 수 있으니 주의할 것
- protected 주의
  > - public 클래스에서 protected로 설정된 멤버는 공개 API로 간주되어 영원히 지원되어야 한다
  > - 이러한 멤버는 가급적 적게 사용하는 것이 좋다

- 예시코드

  ```java
  public class ExampleClass implements Serializable {
      private static final long serialVersionUID = 1L;
  
      // private 멤버 (외부에서 접근 불가)
      private String privateField = "Private Value";
  
      // package-private 멤버 (같은 패키지 내에서 접근 가능)
      String packagePrivateField = "Package-Private Value";
  
      // protected 멤버 (서브클래스에서만 접근 가능)
      protected String protectedField = "Protected Value";
  
      // 공개 API
      public String getPrivateField() {
          return privateField;
      }
  
      public void setPrivateField(String value) {
          this.privateField = value;
      }
  }
  ```

### 4. 접근성을 최소화하는 이유

- **정보 은닉 강화**: 외부에 불필요한 정보를 노출하지 않음으로써 객체의 무결성을 유지하고, 불필요한 의존성을 줄일 수 있다.
- **API 간결성**: 필요한 부분만 공개함으로써 API가 간결해지고 사용하기 쉬워진다.
- **유연성**: 내부 구현을 자유롭게 변경할 수 있고, 외부에 미치는 영향을 최소화할 수 있다.
- **보안성**: 중요한 데이터나 민감한 로직이 외부에 노출되지 않도록 보호할 수 있다.

<br>

### (+) 접근성 최소화의 장점 예시

```java
public class BankAccount {

    // Private 필드로 외부에 노출되지 않음
    private String accountNumber;
    private double balance;

    // Public 생성자: 필수 정보만 외부에서 설정
    public BankAccount(String accountNumber, double initialBalance) {
        this.accountNumber = accountNumber;
        this.balance = initialBalance;
    }

    // Public API: 잔액 조회
    public double getBalance() {
        return balance;
    }

    // Private 메서드로 내부 로직 감춤
    private boolean isBalanceSufficient(double amount) {
        return balance >= amount;
    }

    // Public API: 출금 기능 제공
    public void withdraw(double amount) {
        if (isBalanceSufficient(amount)) {
            balance -= amount;
        } else {
            throw new IllegalArgumentException("잔액 부족");
        }
    }
}
```

- 이 예시에서 `accountNumber`와 `balance`는 `private`로 선언되어 외부에서 직접 접근할 수 없고, `getBalance()`와 같은 필수 정보만 외부에 제공된다
- 내부 로직인 `isBalanceSufficient()`는 외부에 노출되지 않으며, 이를 통해 클래스는 자신의 무결성을 보호하고 유연성을 유지할 수 있다

> 💡 **핵심 포인트**: 접근성을 최소화하면 프로그램의 안정성과 유지보수성이 향상된다

### 결론

- 클래스의 필드, 메서드, 생성자의 접근성을 최대한 줄여서 **정보 은닉**을 강화하는 것이 프로그램 설계의 핵심이다
- **Public API는 최소한으로 유지**하고, 외부에 필요한 부분만 공개하며, 내부 구현은 철저히 감추는 것이 바람직하다 클래스는 자신의 무결성을 보호하고 유연성을 유지할 수 있다

> 💡 **핵심 포인트**: 접근성을 최소화하면 프로그램의 안정성과 유지보수성이 향상된다

### 결론

- 클래스의 필드, 메서드, 생성자의 접근성을 최대한 줄여서 **정보 은닉**을 강화하는 것이 프로그램 설계의 핵심이다
- **Public API는 최소한으로 유지**하고, 외부에 필요한 부분만 공개하며, 내부 구현은 철저히 감추는 것이 바람직하다