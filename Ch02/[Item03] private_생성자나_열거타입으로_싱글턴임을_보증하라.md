# 싱글턴

## ⛳️ 목표

- **싱글턴 패턴**을 구현할 때, **private 생성자**와 **열거 타입**을 사용하여 인스턴스 생성을 제한하고, **싱글턴 특성을 안전하게 보장**하는 방법을 이해하자

<br>

## 📄 핵심 요약

***싱글턴***

- 인스턴스를 오직 하나만 생성할 수 있는 클래스를 말한다.
- 공유자원을 일관성 있게 관리할 수 있다.

***

### 1. 구현방식[1] : public static 멤버가 final 필드인 방식

싱글턴 인스턴스를 클래스가 로딩되는 시점에 즉시 초기화(eager initialization)하는 방법이다.

- 클래스 로딩 시점에 인스턴스가 생성되므로 별도의 동기화가 필요하지 않기 때문에 스레드 안정성을 보장한다.
- 한 번 생성된 인스턴스는 변경 불가능하며 항상 동일한 객체가 반환된다.
- 코드가 짧고 직관적이다.

  ```java
      public class Singleton {
      // 클래스가 로딩될 때 인스턴스를 생성 (static과 final로 선언)
      public static final Singleton instance = new Singleton();
    
          // private 생성자로 외부에서 객체 생성을 방지
          private Singleton() {
              // 생성자는 외부에서 호출할 수 없습니다.
          }
    
          // 인스턴스를 반환하는 메서드
          public static Singleton getInstance() {
              return instance;  // 항상 같은 인스턴스를 반환
          }
    
          // 예시로 사용할 메서드
          public void showMessage() {
              System.out.println("싱글턴 예제 코드입니다. ");
          }
      }
  ```

### 2. 구현방식[2] : 정적 팩터리 메서드 사용

반환하는 메서드를 제공함으로써 외부에서 직접 객체를 생성하지 못하게 하는 방법이다.

- 유연성 : 팩터리 메서드를 통해 객체를 반환하므로 객체 생성 로직을 변경할 필요가 있으면 코드 내부에서만 수정이 가능하다.
- 지연 초기화 가능 : 인스턴스가 실제로 필요할 때 생성될 수 있도록 지연 초기화를 구현할 수 있다.
- 제네릭 지원 : 다양한 타입에 대해 싱글턴을 구현할 수 있다.

  ```java
    public class Singleton {
    // private static 멤버로 인스턴스를 선언하지만 아직 초기화하지 않음 (지연 초기화)
    private static Singleton instance;
   
        // private 생성자로 외부에서 객체 생성을 방지
        private Singleton() {
            // 생성자는 외부에서 호출할 수 없습니다.
        }
  
        // public static 팩터리 메서드를 통해 인스턴스를 반환
        public static Singleton getInstance() {
            // 인스턴스가 아직 생성되지 않았다면 새로 생성 (지연 초기화)
            if (instance == null) {
                instance = new Singleton();
            }
            return instance;  // 이미 생성된 인스턴스를 반환
        }
  
        // 예시로 사용할 메서드
        public void showMessage() {
            System.out.println("Hello, I am a Singleton using a static factory method!");
        }
    }
  ```

### 3. 구현방식 비교

공통점

- public static 접근: 두 방식 모두 인스턴스를 외부에서 접근할 수 있도록 public static 멤버로 제공한다.
- 인스턴스 재사용: 두 방식 모두 동일한 인스턴스를 반환하여, 매번 새 객체를 생성하지 않고 재사용한다.

차이점

| **특징**      | **public static 멤버가 final 필드인 방식**                          | **정적 팩터리 메서드 사용하는 방식**             |
|-------------|------------------------------------------------------|------------------------------------|
| **초기화 시점**  | 클래스가 로딩될 때 즉시 초기화 (eager initialization) | 필요할 때 초기화 (lazy initialization) 가능 |
| **인스턴스 생성** | 한 번만 생성되며, 더 이상 변경할 수 없음                    | 필요에 따라 인스턴스를 생성할 수 있음              |
| **유연성**     | 생성 로직이 고정되어 있으며, 변경하기 어렵다                   | 객체 생성 로직을 쉽게 변경할 수 있다              |
| **메서드 형태**  | 단순히 인스턴스를 반환하는 `static final` 필드                 | 인스턴스를 반환하는 `static` 메서드            |
| **동기화**     | 기본적으로 스레드 안전                                     | 동기화 처리가 필요할 수 있음                   |

정리



### 4. 구현방식[3] : 열거타입 선언

앞서 방식들보다 더 간결하고, 추가노력 없이 직렬화 할 수 있는 방법이다.

- 직렬화 상황이나 리플렉션 공격에서도 제 2의 인스턴스가 생기는 일을 막는다.
- 원소가 하나뿐인 열거 타입이 싱글턴을 만드는 가장 좋은 방법이다.

  ```java
    public enum Elvis {
        INSTANCE;
      
        public void leaveTheBuilding() {
            System.out.println("Elvis is leaving the building!");
        }
    }
  ```
