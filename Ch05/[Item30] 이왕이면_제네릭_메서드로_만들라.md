## 이왕이면 제네릭 메서드로 만들라

## ⛳️ 목표

비제네릭 타입을 제네릭 타입으로 변환할 때의 이점을 알아보자

<br>

## 📄 핵심 요약

- 메서드에 타입 매개변수를 선언하여 제네릭 타입으로 변환하면 타입 안정성을 보장할 수 있다.
- 정적 팩터리를 활용해 불변 객체를 여러 타입으로 활용할 수 있다.
- 재귀적 타입 한정을 통해 자기 자신과 비교 가능한 객체를 안전하게 처리할 수 있다.

---

## 비제네릭 타입 사용 예제

형변환 경고가 발생하므로 타입 안전하게 만들어야 한다.

```java
public static Set union(Set s1, Set s2) {
	Set result = new HashSet(s1);
	result.addAll(s2);
	return result;
}
```
<br>

## 제네릭 타입 사용 예제

경고 없이 컴파일되며, 타입 안전하고, 쓰기 쉽게 변경해보자.

```java
public static <E> Set<E> uniton(Set<E> s1, Set<E> s2) {
	Set<E> result = new HashSet<>(s1);
	result.addAll(s2);
	return result;
}
```

- 메서드 선언에서의 세 집합의 원소 타입을 타입 매개변수로 명시한다.
- 메서드 안에서 타입 매개변수를 사용하도록 한다.

<br>

## 여러 타입 사용 예제

- 불변 객체를 여러 타입으로 활용할 수 있게 만들 수 있다.
- 요청한 타입 매개변수에 맞게 그 객체의 타입을 바꿔주는 `정적 팩터리`를 만들어야 한다. 이 패턴을 `제네릭 싱글턴 팩터리`라고 한다.

***Collections.reverseOrder()***

```java
@SuppressWarnings("unchecked")
public static <T> Comparator<T> reverseOrder() {
    return (Comparator<T>) ReverseComparator.REVERSE_ORDER;
}
```

- `ReverseComparator`를 `Comparator<T>`로 형변환하면 비검사 형변환 경고가 발생한다.

<br>

***ReverseComparator.class***

```java
private static class ReverseComparator
  implements Comparator<Comparable<Object>>, Serializable {

  @java.io.Serial
  private static final long serialVersionUID = 7207038068494060240L;

  static final ReverseComparator REVERSE_ORDER
      = new ReverseComparator();

  public int compare(Comparable<Object> c1, Comparable<Object> c2) {
      return c2.compareTo(c1);
  }

  @java.io.Serial
  private Object readResolve() { return Collections.reverseOrder(); }

  @Override
  public Comparator<Comparable<Object>> reversed() {
      return Comparator.naturalOrder();
  }
}
```

- `ReverseComparator`의 내부 구현을 보면 이 객체가 `Comparator<T>`로 안전하게 사용될 수 있다는 것을 알 수 있다.
- `@SuppressWarnings`를 추가해 경고를 제외시킨다.

<br>

## 재귀적 타입 한정

`재귀적 타입 한정(recursive type bound)`은 자기 자신이 들어간 표현식을 사용하여 타입 매개변수의 허용 범위를 한정할 수 있는 개념이다.

```java
public interface Comparable<T> {
	int compareTo(T o);
}
```

- 타입 매개변수 `T`는 `Comparable<T>`를 구현한 타입이 비교할 수 있는 원소의 타입을 정의한다.
- 자기 자신과 비교 가능한 객체를 안전하게 처리할 수 있다.
- 정렬, 검색, 최솟값이나 최솟값을 구하는 작업에서 특정 타입 제한을 두므로 오류를 방지할 수 있다.