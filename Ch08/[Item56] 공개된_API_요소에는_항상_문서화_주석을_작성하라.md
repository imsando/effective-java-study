# 공개된 API 요소에는 항상 문서화 주석을 작성하라

## ⛳️ 목표

- `Javadoc`을 사용하여 API 문서를 작성하는 방법에 대해 알아보자.

<br>

## 📄 핵심 요약

### 1. Javadoc 의 개요

#### 1️⃣ Javadoc 이란?
- 자바에서 제공하는 **API 문서 자동 생성 도구**이다.
- 소스코드에서 **문서화 주석(doc comment)** 을 추출하여 HTML 문서를 생성한다.

#### 2️⃣ 문서화 주석의 중요성
- 모든 공개된 클래스, 인터페이스, 메서드, 필드에 주석을 작성해야 한다.
- 문서화 주석이 없는 API 는 선언만 나열되며, 사용자 혼란을 초래할 가능성이 크다.

<br>
<br>

### 2. Javadoc 작성 원칙

#### 1️⃣ 공개된 모든 API 에 문서화 주석 작성
- **클래스, 인터페이스, 메서드, 필드** 등 모든 공개 API 요소는 주석이 필요하다.
- **기본 생성자**는 문서화할 수 없으므로, 반드시 명시적 생성자를 제공하는 것이 좋다.

#### 2️⃣ **what**을 설명하고 **how**은 생략
- 문서화 주석은 **해당 API 가 무엇을 하는지**를 설명해야 한다.
- 메서드의 동작 원리보다 **계약(contract)** 을 설명하는 것이 중요하다.

#### 3️⃣ 메서드의 계약 명확히 하기
- **전제조건(Precondition)** 과 **사후조건(Postcondition)** 을 모두 문서화해야 한다.
- **부작용(Side Effect)** 이 있다면 명확히 기술해야 한다.

<br>
<br>

### 3. Javadoc 태그 사용법

#### 1️⃣ 주요 태그
> - **`@param`** : 메서드의 매개변수를 설명
> - **`@return`** : 메서드 반환값을 설명
> - **`@throws`** : 발생할 수 있는 예외와 그 조건을 설명
> - **`@see`** : 관련 API 로의 링크를 제공
> - **`@since`** : API 가 추가된 버전을 명시

#### 2️⃣ 태그 활용 예시
```java
/**
 * 지정된 두 정수를 더한다.
 *
 * @param a 더할 첫 번째 정수
 * @param b 더할 두 번째 정수
 * @return 두 정수의 합
 * @throws IllegalArgumentException 두 정수 중 하나가 음수일 경우
 */
public int add(int a, int b) {
    if (a < 0 || b < 0) {
        throw new IllegalArgumentException("음수는 허용되지 않습니다.");
    }
    return a + b;
}
```

#### 3️⃣ 설명 작성 관례
- `@param`과 `@return` 설명은 **명사구**로 작성한다.
- `@throws` 설명은 **if**로 시작하는 조건문으로 작성한다.

<br>
<br>

### 4. HTML 태그와 메타문자

#### 1️⃣ 주요 HTML 태그
- **`{@code}`**: 코드 폰트를 적용한다.
  ```java
  * {@code index < 0}일 경우 예외가 발생한다.
  ```
- **`{@literal}`**: HTML 메타문자를 표시한다.
  ```java
  * {@literal a < b}는 true다.
  ```
- **`<pre>`**: 줄바꿈과 들여쓰기를 유지한 코드 블록을 생성한다.
  ```java
  * <pre>
  * for (int i = 0; i < 10; i++) {
  *     System.out.println(i);
  * }
  * </pre>
  ```

#### 2️⃣ 작성 시 유의사항
- 문서에서 HTML 메타문자(`<`, `>`, `&`)는 **`{@literal}`** 로 처리한다.
- 다중 줄 코드에는 `<pre>{@code ...}</pre>`를 활용한다.

<br>
<br>

### 5. 문서화 주석의 구조

#### 1️⃣ 요약 설명 (Summary Description)
- 주석의 첫 번째 문장은 요약 설명으로 간주되며, 문서의 헤더에 표시된다.
- 요약 설명은 각 API 요소에 대해 **고유**해야 한다.
- **메서드와 생성자**: 동사구로 작성한다.
  ```java
  // 예: Constructs an empty list with the specified initial capacity.
  ```
- **클래스와 필드**: 명사구로 작성.
  ```java
  // 예: An instantaneous point on the time-line.
  ```

#### 2️⃣ 상세 설명 (Detailed Description)
- 요약 설명 뒤에 상세한 계약과 사용 방법을 기술한다.
- 필요 시 HTML 태그와 코드 예제를 포함한다.

<br>
<br>

### 6. 상속과 Javadoc

#### 1️⃣ 문서화 주석 상속
- **`{@inheritDoc}`** 태그를 사용하면 상위 타입의 문서화 주석을 재사용할 수 있다.
  ```java
  /**
   * {@inheritDoc}
   */
  @Override
  public boolean equals(Object obj) {
      // 구현 내용
  }
  ```

#### 2️⃣ `@implSpec` 태그
- 메서드의 **구현 요구사항(Implementation Requirements)** 을 명시한다.
  ```java
  /**
   * {@implSpec}
   * 이 메서드는 모든 속성을 비교한다.
   */
  public boolean equals(Object obj) {
      // 구현 내용
  }
  ```

<br>
<br>

### 7. 패키지 및 모듈 문서화

#### 1️⃣ 패키지 문서화
- **`package-info.java`** 파일에 패키지 설명을 작성한다.
- 패키지의 역할과 포함된 클래스 개요를 기술한다.

#### 2️⃣ 모듈 문서화
- **`module-info.java`** 파일에 모듈의 역할과 의존성을 설명한다.

<br>
<br>

### 8. 자주 누락되는 정보

#### 1️⃣ 스레드 안전성
- 클래스가 스레드 안전한지 여부를 반드시 명시한다.
  ```java
  // 예: 이 클래스는 Immutable하며 스레드 안전합니다.
  ```

#### 2️⃣ 직렬화 가능성
- 직렬화할 수 있는 클래스는 직렬화 형태를 API 에 포함한다.
  ```java
  // 예: 직렬화는 기본 직렬화 형태를 따릅니다.
  ```

<br>
<br>

### 9. Javadoc 검사와 유지보수

#### 1️⃣ 문서화 품질 검사
- 명령줄에서 **`-Xdoclint`** 옵션으로 문법 오류를 점검할 수 있다.
- Checkstyle 플러그인을 사용하여 IDE에서 실시간으로 검사 가능하다.

#### 2️⃣ 문서 완성도 확인
- Javadoc 으로 생성된 HTML 문서를 직접 읽어보고 검토한다.
- 작성된 문서가 명확하고, 누락된 정보가 없는지 확인한다.

---

## 💡 핵심 정리

1. 공개된 모든 API 요소에 문서화 주석을 작성하라.
2. Javadoc 태그와 HTML 태그를 적절히 활용하여 읽기 쉬운 문서를 작성하라. 
3. Javadoc 문서를 직접 읽어보고 개선점을 발견하라.  