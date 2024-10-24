# public 클래스에서는 접근자 메서드를 사용하라

## ⛳️ 목표

- 인스턴스 필드를 노출하지 않도록 클래스 설계를 개선하는 방법에 대해 알아보자

<br>

## 📄 핵심 요약

### 잘못된 설계 [1] : 인스턴스 필드를 직접 노출하는 클래스

```java
class Point {
    public double x;
    public double y;
}
```

- 이와 같은 클래스는 **캡슐화**를 전혀 제공하지 못한다
- 필드가 `public`으로 선언되어 외부에서 직접 접근할 수 있기 때문이다
    > - (1) **내부 표현을 변경**할 수 없으며 **불변식을 보장**할 수도 없다
    > - (2) 외부에서 필드에 접근할 때 **부수적인 작업**(ex. 값 검증, 변경 알림 등)을 할 수 없다

<br>

### 잘못된 설계 [2] : `public` 클래스에서 불변 필드를 직접 노출

```java
public final class Time {
    public final int hour;
    public final int minute;

    public Time(int hour, int minute) {
        if (hour < 0 || hour >= 24)
            throw new IllegalArgumentException("Invalid hour: " + hour);
        if (minute < 0 || minute >= 60)
            throw new IllegalArgumentException("Invalid minute: " + minute);
        this.hour = hour;
        this.minute = minute;
    }
}
```

- 이 클래스는 `hour`와 `minute` 필드를 `public`으로 직접 노출하고 있다
- 필드가 **불변**이므로 값을 변경할 수 없는 점은 안전하다
    > - (1) 그러나 **내부 표현을 바꿀 수 없으며** 필드 접근 시 **부수 작업**을 할 수 없다
    > - (2) 따라서 불변 필드라고 해도 `public`으로 직접 노출하는 것은 좋지 않다

<br>

### 개선된 설계 방법[1] : 접근자와 변경자를 통한 캡슐화

```java
class Point {
    private double x;
    private double y;

    public Point(double x, double y) {
        this.x = x;
        this.y = y;
    }

    public double getX() {
        return x;
    }

    public double getY() {
        return y;
    }

    public void setX(double x) {
        this.x = x;
    }

    public void setY(double y) {
        this.y = y;
    }
}
```
- 필드를 `private`으로 선언하여 **정보 은닉**을 적용한 코드이다
- **접근자(getter)** 와 **변경자(setter)** 메서드를 통해 필드에 접근하도록 설계함으로써 필드 값을 변경할 때 부가 작업을 추가할 수 있다
    - 예) 값 검증,  로그 기록 등
- **내부 구현을 변경** 할 때도 API를 수정하지 않아도 된다

<br>

### 개선된 설계 방법[2] : 불필요한 캡슐화 피하기 (`package-private` 및 `private` 중첩 클래스)

- **package-private** 또는 **private 중첩 클래스**의 경우 필드를 직접 노출해도 괜찮은 경우가 있다
    - 패키지 바깥에서 사용되지 않는 클래스라면 접근자를 사용하기보다 **필드를 직접 노출**하는 것이 간결할 수 있다
- **public 클래스** 는 필드를 직접 노출하면 **API 변경의 제약** 이 생기므로 필드를 **캡슐화**하는 것이 필수적이다

<br>

### 결론

- **public 클래스**에서는 절대로 **가변 필드를 직접 노출**하지 말아야 한다
- **불변 필드**라고 하더라도 직접 노출하면 유연성을 잃게 되므로 가능하면 **접근자를 사용**하자
- **package-private** 또는 **private 중첩 클래스**에서는 상황에 따라 필드를 직접 노출하는 것이 효율적일 수 있다

> 💡 **핵심 포인트**: 클래스의 필드에 직접 접근하지 않도록 캡슐화하고 API는 유연하게 설계하여 변경이 용이하도록 하자