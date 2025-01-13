# 정확한 계산이 필요하다면 float와 double을 피하라

## ⛳️ 목표

- 정확한 계산을 해야할 때 부동소수점을 피해야하는 이유에 대해 알아보자.

---

## 📄 핵심 요약

### **부동소수점 연산의 한계**

1. **float와 double의 설계 목적**
   - 과학 및 공학 계산용으로 설계되었다.
   - 넓은 범위의 값을 빠르게 계산하기 위해 이진 부동소수점 방식을 채택하곤 했다.
   - 정밀한 근사값은 제공하지만, **정확한 값**이 필요한 계산에는 부적합하다.

2. **부동소수점의 문제 사례**
   - 부정확한 값 반환
     ```java
     System.out.println(1.03 - 0.42); // 0.6100000000000001 출력
     System.out.println(1.00 - 9 * 0.10); // 0.09999999999999998 출력
     ```

   - 반복 연산 문제
     ```java
     double funds = 1.00;
     for (double price = 0.10; funds >= price; price += 0.10) {
         funds -= price;
     }
     System.out.println(funds); // 0.3999999999999999 출력
     ```

---

### **해결 방법**

1. **BigDecimal 활용**
   - 정확한 소수 계산이 가능하다.
   - 문자열 생성자를 사용해 부정확한 값을 방지할 수 있다.
     ```java
     BigDecimal funds = new BigDecimal("1.00");
     BigDecimal price = new BigDecimal("0.10");
     funds = funds.subtract(price);
     ```

   - 장점
     - 정확한 결과를 제공한다.
   - 단점
     - 기본 타입보다 쓰기 불편하고, 계산하는 시간으로 인해 성능이 느린다.

2. **int나 long 사용**
   - 정수를 사용해 소수점을 직접 관리
     ```java
     int funds = 100; // 센트 단위
     int price = 10;
     funds -= price;
     ```

   - 장점
     - 성능이 우수하고, 코드가 간단하다.
   - 단점
     - 다룰 수 있는 값의 크기에 제한이 있다.

---

### **코드 예제**

#### 1️⃣ BigDecimal을 사용한 정확한 계산
```java
import java.math.BigDecimal;

public class AccurateCalculation {
    public static void main(String[] args) {
        final BigDecimal TEN_CENTS = new BigDecimal("0.10");
        int itemsBought = 0;
        BigDecimal funds = new BigDecimal("1.00");

        for (BigDecimal price = TEN_CENTS; funds.compareTo(price) >= 0; price = price.add(TEN_CENTS)) {
            funds = funds.subtract(price);
            itemsBought++;
        }

        System.out.println(itemsBought + "개 구입");
        System.out.println("잔돈(달러): " + funds);
    }
}
```

#### 2️⃣ int를 사용한 성능 최적화
```java
public class AccurateCalculationWithInt {
    public static void main(String[] args) {
        int itemsBought = 0;
        int funds = 100; // 센트 단위

        for (int price = 10; funds >= price; price += 10) {
            funds -= price;
            itemsBought++;
        }

        System.out.println(itemsBought + "개 구입");
        System.out.println("잔돈(센트): " + funds);
    }
}
```

---

## 💡 핵심 정리

1. **float와 double은 근사치를 제공**하므로 금융 계산 등 **정확한 계산**에는 사용하지 말자.
2. **BigDecimal**을 사용하면 정확한 계산이 가능하지만 성능이 느리며 사용이 번거롭다.
3. **int나 long**을 사용해 소수점을 직접 관리하면 성능이 우수하며, 간단한 계산에 적합하다.
4. 상황에 따라 적절한 방식을 선택하라
    - **정확성이 최우선**인 경우 : BigDecimal
    - **성능이 중요**하거나 계산 범위가 제한적인 경우 : int 또는 long
