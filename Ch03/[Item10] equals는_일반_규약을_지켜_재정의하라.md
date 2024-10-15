# equals는 일반 규약을 지켜 재정의하라

## ⛳️ 목표

`equals()` 메서드를 올바르게 재정의하는 방법을 알아보자

## 📄 핵심 요약

1️⃣ 논리적 동치성을 비교해야 할 때 **Object 명세**를 지켜 `equals()` 메서드를 재정의 해야 한다.

- **반사성(reflexivity)** : `x.equals(x)`는 항상 `true`여야 한다.
- **대칭성(symmetry)** : `x.equals(y)`가 `true`이면 `y.equals(x)`도 반드시 `true`여야 한다.
- **추이성(transitivity)** : `x.equals(y) == true`이고 `y.equals(z) == true`이면, `x.equals(z) == true`여야 한다.
- **일관성(consistency)** : `x.equals(y)`의 결과는 외부 요인에 의해 변하지 않아야 한다.
- **null 아님** : `x.equals(null)`는 항상 `false`여야 한다.

2️⃣ 올바른 타입으로 형변환하여 핵심 필드들이 일치하는지 확인해야 한다.

3️⃣ `equals`를 재정의 할 땐 `hashCode`도 반드시 재정의 하자. (아이템 11)

---

## 재정의 하지 않는 경우

`equals()` 메서드를 아예 재정의하지 않으면 그 클래스의 인스턴스는 오직 자기 자신과만 같게 된다. **다음에서 열거한 상황 중 하나에 해당한다면 재정의하지 않는 것이 최선이다.**

- 각 인스턴스가 본질적으로 고유하거나 논리적 동치성을 검사할 일이 없다.
- 상위 클래스에서 재정의한 `equals()`가 하위 클래스에도 사용 가능하다.
- 클래스가 `private`이거나 `package-private`이고 `equals()` 메서드를 호출할 일이 없다.

<br>

## 재정의 해야 하는 경우

`객체 식별성(object identity; 두 객체가 물리적으로 같은가)`이 아니라 `논리적 동치성(object equality)`을 확인해야 하는데, 상위 클래스의 `equals()`가 논리적 동치성을 비교하도록 재정의되지 않았을 때이다.

<br>

### 재정의 시 따라야 하는 Object 명세

> equals 메서드는 `동치관계(equivalence relation)`을 구현하며, 다음을 만족한다.
- **반사성(reflexivity)**
- **대칭성(symmetry)**
- **추이성(transitivity)**
- **일관성(consistency)**
- **null 아님**

<br>

1️⃣ **반사성(reflexivity)**

null이 아닌 모든 참조 값 x에 대해, `x.equals(x)`는 `true`이다. 즉, **객체는 자기 자신과 같아야 한다는 뜻**이다.

<br>

2️⃣ **대칭성(symmetry)**

null이 아닌 모든 참조 값 x, y에 대해 `x.equals(y)`가 `true`면 `y.equals(x)`도 `true`다. 즉, **두 객체는 서로에 대한 동치 여부에 똑같이 답해야 한다는 뜻이다.**

***❌ 대칭성이 지켜지지 않는 예제***

```java
public final class CaseInsensitiveString {
	
	private final String str;
	
	public CaseInsensitiveString(String str) {
		this.str = Objects.requireNonNull(str);
	}
	
	@Override
	public boolean equals(Object o) {
		if (o instanceof CaseInsensitiveString) {
			return str.equalsIgnoreCase((CaseInsensitiveString) o).s);
		}
		if (o instanceof String) { // 한 방향으로만 작동
			return s.equals((String) o);
		}
		return false;
	}
}
```

<br>

`cis.equals(str)`은 `true`를 반환하지만 `s.equals(cis)`는 `false`를 반환하여 대칭성을 위반한다. 그렇다면 코드를 어떻게 변경해야 할까?

```java
CaseInsensitiveString cis = new CaseInsensitiveString("Polish");
String str = "polish";
```

***⭕️ 대칭성이 지켜진 예제***

```java
@Override
public boolean equals(Object o) {
	return o instanceof CaseInsensitiveString &&
		((CaseInsensitiveString) o).s.equalsIgnoreCase(str);
}
```

- 이전 코드와는 다르게 `표준형(canocical form)`을 `String`이 아닌 `CaseInsensitiveString`의 필드값으로 비교하였다.

<br>

3️⃣ **추이성(transitivity)**

null이 아닌 모든 참조 값 x, y, z에 대해 `x.equals(y)`가 `true`이고 `y.equals(z)`가 `true`면, `x.equals(z)`도 `true`다. 즉, **첫 번째 객체와 두 번째 객체가 같고, 두 번째 객체와 세 번째 객체가 같으면 첫 번째 객체와 세 번째 객체도 같아야 한다는 뜻**이다.

<br>

***❌ 추이성이 지켜지지 않은 예제***

```java
public class Point {
	
	private final int x;
	private final int y;
	
	public Point(int x, int y) {
		this.x = x;
		this.y = y;
	}
	
	@Override
	public boolean equals(Object o) {
		if (!(o instanceof Point)) {
			return false;
		}
		Point p = (Point) o;
		return p.x == x && p.y == y;
	}
}

public class ColorPoint extends Point {
    private final String color;

    public ColorPoint(int x, int y, String color) {
        super(x, y);
        this.color = color;
    }

    @Override
    public boolean equals(Object o) {
        if (!(o instanceof ColorPoint)) {
            return false;
        }
        ColorPoint cp = (ColorPoint) o;
        return super.equals(o) && cp.color.equals(color);
    }
}
```

- `Point`는 좌표 값을 비교한다.
- `ColorPoint`는 `Point`를 상속하며 색깔까지 비교한다.

<br>

상위 클래스와 하위 클래스의 `equals()`가 일치하지 않기 때문에 p1과 p2, p1과 p3는 좌표 값만 비교하여 `true`가 반환되니 추이성이 깨지게 된다.

`리스코프 치환 원칙(LSP, Liskov substitution principle)`에 따라 상위 타입의 모든 메서드가 하위 타입에서도 똑같이 잘 작동해야 한다는 점을 떠올리며 코드를 수정하자.

```java
Point p1 = new Point(1, 2);
Point p2 = new ColorPoint(1, 2, "red");
Point p3 = new ColorPoint(1, 2, "blue");

System.out.println(p1.equals(p2));  // true, 좌표만 비교
System.out.println(p2.equals(p3));  // false, 색깔이 다르므로
System.out.println(p1.equals(p3));  // true, 좌표만 비교
```

***⭕️ 추이성이 지켜진 예제***

```java
public class ColorPoint {
    private final Point point;
    private final String color;

    public ColorPoint(int x, int y, String color) {
        this.point = new Point(x, y);  // Point 객체를 참조
        this.color = color;
    }

    @Override
    public boolean equals(Object o) {
        if (!(o instanceof ColorPoint)) {
            return false;
        }
        ColorPoint cp = (ColorPoint) o;
        return point.equals(cp.point) && color.equals(cp.color);  // Point의 equals 사용
    }
}

```

- `ColorPoint`에서 `Composition` 방식으로 `Point`을 참조한다.
- `Point`의 `equals()`에서 좌표를 비교하고, `color` 비교는 추가한다.

<br>

🏷️ **참고**

> `리스코프 치환 원칙(LSP, Liskov substitution principle)`은 상위 타입의 객체를 하위 타입의 객체로 치환해도 동작에 문제가 없어야 한다는 원칙이다.
>

<br>

4️⃣ **일관성(consistency)**

null이 아닌 모든 참조 값 x,y에 대해 `x.equals(y)`를 반복해서 호출하면 항상 `true`를 반환하거나 항상 `false`를 반환한다. 즉, **두 객체가 같다면 가변 객체가 아닌 이상 영원히 같아야 한다는 뜻**이다.

- `equals()` 메서드에서 사용하는 자원이 변경될 수 있는 값이 되면 안 된다. (e.g. 변경 가능성이 있는 호스트의 IP)

<br>

5️⃣ **null 아님**

null이 아닌 모든 참조 값 x에 대해, `x.equals(null)`은 `false`이다. 즉, **모든 객체가 null이 아니어야 한다는 뜻**이다. 그렇지만 코드 내부에 명시적으로 null을 검증하지 않고 형변환에 앞서 `instanceof` 연산자로 매개변수가 올바른 타입인지 검사하며 묵시적으로 null 검사를 된다.

<br>

### **양질의 equals 메서드 구현 방법**

- `==` 연산자를 사용해 입력이 자기 자신의 참조인지 확인한다.
    - float과 double 필드는 특수한 값(Float.NaN, -0.0f, 특수한 부동소수 값)들을 다뤄야 하기 때문에 `compare()` 메서드로 비교한다. `equals()` 메서드는 오토 박싱이 수반될 수 있어 성능상 좋지 않다.
    - null도 정상 값으로 취급하는 경우는 `Objects.equals(Object, Object)`로 비교해 NPE 발생을 예방해야 한다.
- `instanceof` 연산자로 입력이 올바른 타입인지 확인한다.
- 입력을 올바른 타입으로 형변환한다.
- 입력 객체와 자기 자신의 대응되는 핵심 필드들이 모두 일치하는지 하나씩 검사한다.
    - 다를 가능성이 더 크거나 비교하는 비용이 싼 필드를 먼저 비교하면 성능에 좋다.
    - 동기화용 락(lock) 필드 같이 객체의 논리적 상태와 관련 없는 필드는 비교하면 안 된다.

<br>

### **주의사항**

- `equals`를 재정의할 땐 `hashCode`도 반드시 재정의 하자. (아이템 11)
- 너무 복잡하게 해결하려 들지 말자. 필드들의 동치성만 검사해도 `equals` 규약을 어렵지 않게 지킬 수 있다. 일반적으로 별칭은 비교하지 않는 게 좋다.
- `Object` 외의 타입을 매개 변수로 받는 `equals` 메서드는 선언하지 말자.

    ```java
    // 입력 타입은 반드시 Object!
    public boolean equals(MyClass o) { ... }
    ```

<br>


🏷️ **참고**

> 직접 `equals()` 메서드를 작성하기 보다는 IDE에 맡기거나 `AutoValue` 프레임워크를 활용하자. 또한 재정의 해야 할 때는 클래스의 핵심 필드 모두를 빠짐없이 포함 시키자.
>