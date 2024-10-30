# 상속을 고려하여 설계와 문서화해라

## ⛳️ 목표

- 상속을 사용할 때 설계와 문서화를 고려해야하는 이유에 대해서 알아보자.

<br>

## 📄 핵심 요약

***상속이란?***

- 기존 클래스의 속성과 메서드를 새로운 클래스에 물려주어 재사용성을 높이는 기법이다.
- 주로 `extends` 키워드를 사용해서 부모 클래스를 상속받는다.

***좋은 문서화? (상속을 고려한 문서화?)***

- 상위 클래스의 메서드나 필드를 어떻게 사용해야 하는지 특히 하위 클래스에서 확장하거나 오버라이딩할 때 주의할 점을 명확히 기술하는 것을 의미한다.
- 하위 클래스가 예상치 못한 결과나 오작동을 피할 수 있게 해준다.
- 주로 JavaDoc을 활용해 클래스나 메서드에 주석으로 문서화한다.

<br>

### 1. 재정의 가능 메서드의 명시적 설명
- 상속용 클래스를 설계할 때는 public이나 protected로 선언된 final이 아닌 모든 메서드가 재정의될 수 있다. 
- 하위 클래스가 예상하지 못한 동작을 피하기 위해 상위 클래스가 내부적으로 재정의 가능한 메서드를 어떻게 사용하는지 설명해야 한다.

예시코드
```java
/**
 * 도형을 그립니다. 이 메서드는 도형을 화면에 그리는 작업을 수행하기 전에 준비 작업을 합니다.
 * 하위 클래스에서 `drawShape()`를 재정의하여 개별 도형의 그리기 동작을 구현할 수 있습니다.
 * 
 * <p><b>재정의 가능 메서드 설명:</b></p>
 * 이 메서드는 `drawShape()`를 내부적으로 호출합니다. 
 * 하위 클래스에서 `drawShape()`를 재정의할 경우, 이 메서드를 호출하는
 * 다른 상위 메서드(`render` 등)에도 영향을 미칠 수 있습니다.
 */
public void draw() {
    prepareForDrawing();  // 준비 작업 수행
    drawShape();          // 실제 도형을 그리는 재정의 가능 메서드 호출
}

/**
 * 개별 도형을 화면에 그립니다. `draw()` 메서드에서 호출됩니다.
 * 하위 클래스는 이 메서드를 재정의하여 도형별 그리기 로직을 구현할 수 있습니다.
 */
protected void drawShape() {
    // 기본 그리기 로직 (상위 클래스에서 기본 구현 제공)
}
```
<br>

### 2. @implSpec 태그 활용하기
- Java에서는 @implSpec 태그를 활용해 메서드의 내부 구현 방식을 문서화할 수 있다. 
- 이 태그는 메서드가 어떻게 구현되는지 설명하여 하위 클래스가 상속할 때 내부 동작을 이해하도록 도와준다. 
- 예를 들어 java.util.AbstractCollection 클래스의 remove 메서드는 @implSpec 태그를 통해 반복자를 사용하여 원소를 제거하는 방식으로 구현되었음을 명확히 나타낸다.

```java
/**
 * 요소 제거를 지원하는 추상 컬렉션입니다.
 */
public abstract class MyAbstractCollection<E> extends AbstractCollection<E> {

    /**
     * 컬렉션에 존재하는 경우 지정된 요소의 단일 인스턴스를 제거합니다.
     *
     * @implSpec 이 구현은 반복자를 사용하여 컬렉션을 순회합니다.
     *           지정된 요소가 발견되면, 해당 요소는 반복자의 remove 메서드를 통해 제거됩니다.
     *           이를 통해 컬렉션이 안전하고 예측 가능한 방식으로 수정됩니다.
     *
     * @param element 이 컬렉션에서 제거할 요소 (존재하는 경우)
     * @return 이 호출로 인해 요소가 제거된 경우 {@code true}
     */
    @Override
    public boolean remove(Object element) {
        Iterator<E> iterator = iterator();
        while (iterator.hasNext()) {
            if (iterator.next().equals(element)) {
                iterator.remove();
                return true;
            }
        }
        return false;
    }
}
```
<br>

### 3. 상속 가능 메서드의 제약사항 설계
- 상속을 고려한 클래스의 생성자에서는 재정의 가능 메서드를 호출하지 않는 것이 중요하다.
- 상위 클래스의 생성자가 실행될 때 하위 클래스의 필드는 아직 초기화되지 않았기 때문에 재정의된 메서드를 호출할 경우 NullPointerException 등의 문제가 발생할 수 있다.
- 이를 방지하기 위해 상속용 클래스의 생성자에서는 가급적 private, final, 또는 static 메서드만 호출하도록 설계해야 한다.

나쁜 예시코드
```java
public class Parent {

    public Parent() {
        initialize(); // 상속이 가능한 메서드 호출은 위험
    }

    /**
     * 하위 클래스에서 재정의할 수 있는 메서드
     */
    protected void initialize() {
        // 기본 초기화 코드
    }
}

public class Child extends Parent {

    private String childData;

    public Child() {
        super();
        this.childData = "Child Specific Data";
    }

    @Override
    protected void initialize() {
        // childData는 아직 초기화되지 않았기 때문에 NullPointerException 발생 가능
        System.out.println(childData.toUpperCase());
    }
}
```

좋은 예시코드
```java
public class Parent {

    public Parent() {
        initInternal();
    }

    private void initInternal() {
        // 안전한 초기화 코드
    }

    protected void initialize() {
        // 재정의 가능하지만 생성자에서는 호출하지 않음
    }
}
```
<br>

### 4. 하위 클래스 작성 및 테스트
- 상속 설계가 제대로 이루어졌는지 확인하기 위해 하위 클래스를 작성하고 테스트하는 것이 중요하다.
- 하위 클래스와의 상호작용에서 발생할 수 있는 오류를 사전에 발견할 수 있다.

<br>

> 💡 **핵심 포인트**
> - 상속용 클래스를 설계할 때는 하위 클래스가 예기치 않게 동작하지 않도록 신중하게 접근해야 하며 문서화는 그 필수 요소다.
> - 클래스의 사용법과 제약을 명확하게 설명함으로써 다른 개발자가 의도대로 상속할 수 있도록 돕고 유지보수성을 높일 수 있다.