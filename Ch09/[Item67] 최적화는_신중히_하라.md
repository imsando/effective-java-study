# 최적화는 신중히 하라

## ⛳️ 목표

- 최적화의 위험성에 대해 알아보자.

<br>

## 📄 핵심 요약

---

### **최적화의 어두운 진실**

1. **과도한 최적화의 문제점**
    - 잘못된 최적화는 프로그램의 가독성과 유지보수성을 심각하게 저하시킬 수 있다.
    - 성능 향상을 기대하며 구조를 망가뜨릴 위험이 크다.
    - 책에서 언급된 격언 :
        - *"최적화는 만악의 근원이다."* — 도널드 크누스
        - *"성급한 최적화는 하지 마라."* — M. A. 잭슨

2. **나쁜 결과**
    - 수정이 어려운 프로그램을 만들어 성능 문제를 해결하기 어렵게 만든다.
    - 빠르지도, 제대로 동작하지도 않는 소프트웨어가 탄생한다.

<br>

### **설계와 최적화의 균형**

1. **설계 단계에서 성능을 염두에 두라**
    - 아키텍처의 결함은 성능을 제한하며, 완성 후 수정하기 어렵다.
    - API, 네트워크 프로토콜, 데이터 포맷 등 변경하기 어려운 요소에서 성능 문제를 예방해야 한다.

2. **API 설계에서 성능을 고려하라**
    - 불변 객체를 활용해 불필요한 복사를 방지해야 한다.
    - 상속 대신 컴포지션을 사용해 성능 제약을 피해야 한다.
    - 구체적인 구현 타입 대신 인터페이스를 사용해 유연성을 확보해야 한다.

<br>

### **최적화 시점과 방법**

1. **최적화는 마지막 단계에서**
    - 좋은 설계를 먼저 하고, 명백한 성능 문제가 나타난 이후 최적화를 시도해야 한다.
    - 최적화와 측정
        - *"측정 없이 최적화하지 마라."* — 성능 문제는 종종 예상치 못한 곳에서 발생한다.
        - 90%의 실행 시간은 10%의 코드에서 소비된다.

2. **도구 활용**
    - 프로파일링 도구로 병목 지점을 식별하자.

<br>

### **최적화 시 주의사항**

1. **알고리즘을 먼저 점검하라**
    - 비효율적인 알고리즘을 사용하는 경우, 저수준 최적화는 무의미하다.
    - 효율적인 알고리즘으로 교체하면 성능 문제를 근본적으로 해결할 수 있다.

2. **성능 모델을 이해하라**
    - 자바의 성능 모델은 플랫폼마다 다를 수 있다.
    - 다양한 환경에서 성능을 측정하고 타협점을 찾는 것이 중요하다.

---

### 💡 **핵심 정리**

- 좋은 설계를 먼저 하고, 명백한 성능 문제가 있을 때 최적화를 고려하라.
- 최적화 전에 반드시 성능을 측정하라.
- 알고리즘의 복잡도를 점검하고 프로파일링 도구를 활용해 병목 지점을 식별하라.
- 시스템 설계 시, 특히 API, 네트워크 프로토콜, 데이터 포맷에 성능 문제를 예방하라.