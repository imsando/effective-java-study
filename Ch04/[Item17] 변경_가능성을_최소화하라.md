## 변경 가능성을 최소화하라

## ⛳️ 목표

불변 클래스가 가진 이점을 이해하고 설계 규칙을 알아보자

<br>

## 📄 핵심 요약

- 클래스 설계 시 변경 가능성을 최소화하여 스레드 안정성을 확보해야 한다.
- ***불변 클래스는***
    - 인스턴스를 재활용하여 메모리를 효율적으로 사용할 수 있다.
    - 내부 상태가 변화하지 않기 때문에 이로 인한 에러 발생을 줄일 수 있다.

---

## 불변 클래스를 만드는 5가지 규칙

- 객체의 상태를 변경하는 메서드(변경자) 제공하지 않기
- 클래스를 확장할 수 없게 만들기
- 모든 필드를 final로 선언하기
    - final 필드가 객체 생성 시점에 고정되므로 다중 스레드 환경에서 안전성을 확보할 수 있다. (`가시성` 문제 해결)
- 모든 필드를 private으로 선언하기
- 자신 외에는 내부의 가변 컴포넌트에 접근할 수 없도록 하기

    ```java
    public class Example {
        private List<String> items = new ArrayList<>();
    
        // 내부 메서드에서만 접근
        public void addItem(String item) {
            items.add(item);
        }
    }
    ```

<br>

## 불변 클래스 - 장점

### 장점 1. 단순성

가변 객체에 비해 설계 및 구현이 쉬우며 오류가 생길 여지가 적어 개발자가 다른 노력을 들이지 않아도 된다.

### 장점 2. 스레드 안전

여러 스레드가 동시에 사용해도 절대 훼손되지 않기 때문에 따로 동기화할 필요가 없다.

### 장점 3. 인스턴스 재활용

상수를 사용하거나, 자주 사용되는 인스턴스를 캐싱하여 같은 인스턴스를 중복 생성하지 않게 해주는 정적 팩터리를 제공할 수 있다.

### 장점 4. 공유 가능

자유롭게 공유할 수 있으며 불변 객체끼리 내부 데이터를 공유할 수 있다

```java
String s1 = "Hello";
String s2 = "Hello"; // s1과 s2는 동일한 객체를 참조
```

### 장점 5. 실패 원자성

내부 상태를 바꿀 수 없기 때문에 메서드에서 예외가 발생한 후에도 객체가 메서드 호출 전과 같은 상태를 유지할 수 있다.

<br>

## 불변 클래스 - 단점

값이 다르면 반드시 독립된 객체로 만들어야 한다. 해당 문제를 해결하기 위한 방법은 다음과 같다.

- `다단계 연산(multiple operation)`들을 예측하여 기본 기능으로 제공한다.
    - 여러 개의 개별 연산을 수행할 때마다 새로운 객체를 생성하지 않고 일괄적으로 처리하여 최종 객체를 생성한다.

    ```java
    public class Point {
        private final int x;
        private final int y;
    
        public Point(int x, int y) {
            this.x = x;
            this.y = y;
        }
    
        public Point move(int deltaX, int deltaY) {
            return new Point(this.x + deltaX, this.y + deltaY);
        }
    
        public Point scale(int factor) {
            return new Point(this.x * factor, this.y * factor);
        }
    
        public Point moveAndScale(int deltaX, int deltaY, int factor) {
            return this.move(deltaX, deltaY).scale(factor);
        }
    }
    ```

- `가변 동반 클래스`를 제공한다.
    - `e.g.` **String.class** → **StringBuilder.class**

<br>

## 정리

- 꼭 필요한 경우가 아니라면 불변 클래스를 만들자.
- 불변으로 만들 수 없는 클래스라도 변경할 수 있는 부분을 최소한으로 줄이자.
    - 다른 합당한 이유가 없다면 모든 필드는 `private final`이어야 한다.
- 생성자는 불변식 설정이 모두 완료된, 초기화가 완벽히 끝난 상태의 객체를 생성해야 한다.