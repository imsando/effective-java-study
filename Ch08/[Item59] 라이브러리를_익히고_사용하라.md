# 라이브러리를 익히고 사용하라

## ⛳️ 목표

- 표준 라이브러리의 장점과 이를 활용하는 방법을 이해한다.

<br>

## 📄 핵심 요약

---

### **라이브러리 사용의 이점**

1. **품질 보장**
    - 알고리즘 전문가가 설계 및 검증한 코드로 신뢰성이 높다.
    - 표준 라이브러리는 수백만 명의 사용자가 검증하여 오류가 거의 없다.

2. **시간 절약**
    - 이미 검증된 구현을 활용하여 핵심 업무에 집중 가능하다.
    - 기능 구현에 소요되는 시간을 줄여 개발 속도를 높인다.

3. **성능 최적화**
    - 표준 라이브러리는 성능 개선이 지속적으로 이루어진다.
    - 사용자 기반이 크기 때문에 성능 최적화에 더 많은 자원이 투자된다.

4. **기능 확장**
    - 릴리스마다 새로운 기능이 추가되어 개발자 요구를 충족한다.
    - 최신 기능을 손쉽게 이용할 수 있다.

5. **유지보수성**
    - 코드가 많은 개발자에게 익숙하여 읽기 쉽고 유지보수하기 쉽다.
    - 재사용성이 높아 코드 품질 향상.

---

### **라이브러리 사용 권장 사례**

1. **무작위 숫자 생성**
    - 직접 구현한 코드에는 편향성과 결함이 존재할 수 있다.
    - 표준 라이브러리의 `Random.nextInt(int)` 또는 `ThreadLocalRandom`을 사용한다.

    ```java
    import java.util.concurrent.ThreadLocalRandom;

    public class RandomExample {
        public static void main(String[] args) {
            int randomNumber = ThreadLocalRandom.current().nextInt(0, 100);
            System.out.println(randomNumber);
        }
    }
    ```

2. **대규모 데이터 처리**
    - 컬렉션 프레임워크와 스트림 API 활용한다.
    - 병렬 처리와 고성능 작업 수행한다.

    ```java
    import java.util.List;
    import java.util.stream.Collectors;
    import java.util.stream.IntStream;

    public class StreamExample {
        public static void main(String[] args) {
            List<Integer> numbers = IntStream.range(0, 100).boxed().collect(Collectors.toList());
            List<Integer> evenNumbers = numbers.stream()
                                               .filter(n -> n % 2 == 0)
                                               .collect(Collectors.toList());
            System.out.println(evenNumbers);
        }
    }
    ```

3. **URL 데이터를 효율적으로 읽기**
    - Java 9부터 도입된 `InputStream.transferTo()`를 활용하여 간단하게 구현한다.

    ```java
    import java.io.IOException;
    import java.io.InputStream;
    import java.net.URL;

    public class UrlReader {
        public static void main(String[] args) throws IOException {
            try (InputStream in = new URL("https://example.com").openStream()) {
                in.transferTo(System.out);
            }
        }
    }
    ```

---

### **사용하지 않아야 할 경우**

1. **표준 라이브러리의 한계**
    - 전문적인 고급 기능이 필요한 경우 표준 라이브러리가 충분하지 않을 수 있다.
    - 예 : 통계 라이브러리, 특정 도메인 전문 기능 등.

2. **대안 활용**
    - 표준 라이브러리가 부족하다면 고품질 서드파티 라이브러리를 고려한다.
    - 예 : 구아바(Guava), 아파치 커먼즈(Apache Commons) 등등

---

### 💡 **핵심 정리**

1. **바퀴를 다시 발명하지 말라**
    - 표준 라이브러리 또는 검증된 서드파티 라이브러리를 적극 활용하자.

2. **라이브러리를 주기적으로 학습하라**
    - 주요 업데이트를 정기적으로 확인하고, 새로운 기능을 학습한다.

3. **필요하면 구현하라**
    - 표준 및 서드파티 라이브러리로도 해결할 수 없을 때만 직접 구현한다.

4. **코드 품질과 효율성 향상**
    - 라이브러리를 사용하면 개발 효율성과 유지보수성을 크게 높일 수 있다.