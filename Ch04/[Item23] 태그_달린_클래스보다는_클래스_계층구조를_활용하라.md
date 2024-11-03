# 태그 클래스보다는 계층구조를 이용하라

## ⛳️ 목표

- 태그 클래스보다 계층구조를 이용하는 것이 좋은 이유에 대해서 알아보자

<br>

## 📄 핵심 요약

***태그 클래스란?***

- 태그 필드로 어떤 동작을 할지 결정하지만 그 자체로는 아무런 상태를 가지지 않는다
- 주로 특정 용도로 객체를 구별하거나 식별하기 위한 _태그_ 역할을 수행합니다

예시코드
```java
// Figure 클래스: 태그 필드를 사용하는 방식 (비효율적)
public class Figure {
    enum Shape { RECTANGLE, CIRCLE };

    // 현재 도형의 타입을 나타내는 태그 필드
    final Shape shape;

    // RECTANGLE 타입일 때 사용되는 필드
    double length;
    double width;

    // CIRCLE 타입일 때 사용되는 필드
    double radius;

    // 사각형 생성자
    public Figure(double length, double width) {
        shape = Shape.RECTANGLE;
        this.length = length;
        this.width = width;
    }

    // 원 생성자
    public Figure(double radius) {
        shape = Shape.CIRCLE;
        this.radius = radius;
    }

    public double area() {
        switch (shape) {
            case RECTANGLE:
                return length * width;
            case CIRCLE:
                return Math.PI * (radius * radius);
            default:
                throw new AssertionError(shape);
        }
    }
}
```

***계층구조란?***      

- 상위 클래스(부모 클래스)가 기본적인 속성과 행동을 정의하고 하위 클래스(자식 클래스)는 상위 클래스의 특성을 상속받아 구체적인 특성이나 기능을 확장하는 구조이다

예시코드
```java
// 상위 클래스 - Shape
abstract class Shape {
    abstract double area(); // 모든 도형의 면적을 계산할 메서드
}

// 하위 클래스 - Circle
class Circle extends Shape {
    private final double radius;

    Circle(double radius) {
        this.radius = radius;
    }

    @Override
    double area() {
        return Math.PI * radius * radius;
    }
}

// 하위 클래스 - Rectangle
class Rectangle extends Shape {
    private final double width;
    private final double height;

    Rectangle(double width, double height) {
        this.width = width;
        this.height = height;
    }

    @Override
    double area() {
        return width * height;
    }
}

// 하위 클래스 - Triangle
class Triangle extends Shape {
    private final double base;
    private final double height;

    Triangle(double base, double height) {
        this.base = base;
        this.height = height;
    }

    @Override
    double area() {
        return 0.5 * base * height;
    }
}
```

### 1. 태그 클래스의 단점

> - **⓵ 비효율적** : 태그 필드로 동작을 결정하므로 메모리 공간을 낭비하고 코드를 복잡하게 만든다
> - **② 가독성 저하** : 태그 필드로 동작을 결정하므로 코드를 이해하기 어렵다
> - **⓷ 유지보수성 저하** : 새로운 도형이 추가될 때마다 태그 필드와 switch 문을 추가해야 하므로 유지보수성이 떨어진다
> - **④ 타입 안정성이 떨어짐** : 태그 필드로 동작을 결정하므로 컴파일러가 타입을 검증하지 못한다

정리 : 태그 클래스는 장황하여 오류내기 쉬워 비효율적이다

### 2. 계층구조의 장점

> - **⓵ 비효율성 해결** : 계층구조를 사용하면 태그 필드를 사용하지 않아도 되므로 메모리 공간을 절약할 수 있다
> - **② 가독성 향상** : 계층구조를 사용하면 코드를 이해하기 쉽다
> - **⓷ 유지보수성 향상** : 새로운 도형이 추가될 때 계층구조를 확장하기만 하면 되므로 유지보수성이 높아진다
> - **④ 타입 안정성 보장** : 계층구조를 사용하면 컴파일러가 타입을 검증할 수 있어 안정성이 보장된다

정리 : 계층구조는 코드를 간결하게 만들어주고 오류를 줄여준다

> 💡 **핵심 포인트**
> 실제로 태그 클래스를 사용하는 경우는 적다
> 클래스를 작성할 때 태그 필드가 등장한다면 계층구조로 리팩터링하는 것을 고려해보자