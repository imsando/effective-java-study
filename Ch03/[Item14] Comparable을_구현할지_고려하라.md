# Comparable을 구현할지 고려하라

## ⛳️ 목표

`Comparable` 구현을 통한 안전한 값 비교에 대해 이해하자

<br>

## 📄 핵심 요약

**`*Comparable` 인터페이스 구현을 통해***

- 객체의 동치성은 물론, 순서까지 비교할 수 있다.
- 오버플로우 발생 위험없이 상대적 크기 만을 비교할 수 있다.
- 자바 7 이후부터 도입된 정적 메서드로 값을 안전하고 효율적으로 비교할 수 있다.

---

## Comparable.compareTo()

- 단순 동치성과 순서 비교가 가능하다.
- 제네릭 구조를 가지기 때문에 구체적인 타입을 넣어 사용할 수 있다.

    ```
    public interface Comparable<T> {
        int compareTo(T o);
    }
    ```

- 검색(이진 검색), 극단값(최대값, 최소값) 계산, 자동 정렬되는 컬렉션 관리가 가능하다.

<br>

## compareTo() 일반 규약

규약을 지키지 못하면 비교를 활용하는 클래스를 사용하지 못한다. (`e.g.` Collections, Arrays)

> **이 객체와 주어진 객체의 순서를 비교**한다. 이 객체가 주어진 객체보다 작으면 음의 정수를, 같으면 0을, 크면 양의 정수를 반환한다. 이 객체와 비교할 수 없는 타입의 객체가 주어지면 ***ClassCastException***을 던진다.
>

<br>

1️⃣ **반사성** : 두 객체 참조의 순서를 바꿔 비교해도 예상한 결과가 나와야 한다.

```java
x.compareTo(y)
y.compareTo(x)
```

<br>

2️⃣ **추이성** : 첫 번째가 두 번째보다 크고 두 번째가 세 번째보다 크면, 첫 번째는 세 번째보다 커야 한다.

```java
x.compareTo(y) > 0 && y.compareTo(z) > 0
x.compareTo(z) > 0
```

<br>

3️⃣ **대칭성** : 크기가 같은 객체들끼리는 어떤 객체와 비교하더라도 항상 같아야 한다.

```java
x.compareTo(y) == 0
x.compareTo(z) == y.compareTo(z)
```

<br>

4️⃣ compareTo 메서드로 수행한 동치성 테스트의 결과가 equals 와 같아야 한다.

```java
(x.compareTo(y) == 0) == (x.equals(y))
```

<br>

## equals()와의 비교

- `equals()`는 객체의 동일성을 비교하기 위해 사용되며, 두 객체의 동치성 여부를 true 또는 false로 반환한다.
- `compareTo()`는 객체 간의 순서를 비교할 때 사용되며, 같으면 0, 크거나 작으면 양수 또는 음수를 반환한다.

***예제 코드***

이름과 나이 필드를 가진 `Person` 객체가 있다고 했을 때 `equals()`와 `compareTo()` 메서드의 코드를 비교해서 보자.

```java
// compareTo: 나이를 기준으로 비교
@Override
public int compareTo(Person other) {
    return Integer.compare(this.age, other.age);  // 나이 비교
}

// equals: 이름과 나이가 모두 같아야 동일하다고 판단
@Override
public boolean equals(Object obj) {
    if (this == obj) return true;
    if (obj == null || getClass() != obj.getClass()) return false;
    Person person = (Person) obj;
    return age == person.age && name.equals(person.name);  // 이름과 나이 비교
}
```

***호출하는 코드***

```java
public static void main(String[] args) {
    Person person1 = new Person("Alice", 30);
    Person person2 = new Person("Bob", 30);
    Person person3 = new Person("Alice", 30);
    Person person4 = new Person("Charlie", 25);

    // equals() 비교
    System.out.println("person1 equals person2: " + person1.equals(person2));  // false (이름이 다름)
    System.out.println("person1 equals person3: " + person1.equals(person3));  // true (이름과 나이가 같음)

    // compareTo() 비교 (나이만 기준)
    System.out.println("person1 compareTo person2: " + person1.compareTo(person2));  // 0 (나이가 같음)
    System.out.println("person1 compareTo person4: " + person1.compareTo(person4));  // 양수 (person1 나이가 더 큼)
}
```

***결과***

```java
person1 equals person2: false
person1 equals person3: true

person1 compareTo person2: 0
person1 compareTo person4: 1
```

<br>

## 자바 버전에 따른 비교

### 자바 7 이전

정수 기본 타입 필드를 비교할 때 관계 연산자인 <, > 를 사용하는 것을, 실수 기본 타입 필드를 비교할 때 `Double.compare`, `Float.compare`를 사용하는 것을 권고했다.

<br>

### 자바 7 이후

박싱된 기본 타입 클래스들에 새로 추가된 정적 메서드인 `compare()`를 사용할 수 있다.

```java
public static void main(String[] args) {
    Integer a = 10;
    Integer b = 20;

    // Integer.compare() 사용
    int result = Integer.compare(a, b);
    
    // 생략
}
```

<br>

### 자바 8 이후

Comparator 인터페이스가 일련의 Comparator 생성 메서드로 메서드 연쇄 방식을 구현할 수 있게 되었다.

***자바 7 이전***

`compare()` 정적 메서드를 통해 분기문 내부에서 추가적으로 구현하는 코드이다.

```java
public int compareTo(PhoneNumber pn) {
	int result = Short.compare(areaCode, pn.areaCode);
	if (result == 0) {
		result = Short.compare(prefix, pn.prefix);
		if (result == 0) {
			result = Short.compare(lineNum, pn.lineNum);
		}
	}
	return result;
}
```

***자바 8 이후***

메서드 연쇄 방식으로 구현한 코드로 간결하지만 약간의 성능 저하가 뒤따른다.

```java
// wkqk

private static final Comparator<PhoneNumber> COMPARATOR =
	comparingInt((PhoneNumber pn) -> pn.areaCode)
		.thenComparingInt(pn -> pn.areaCode)
		.thenComparingInt(pn -> pn.lineNum)

public int compareTo(PhoneNumber pn) {
	return COMPARATOR.comapre(this, pn);
}
```

<br>

## 해시코드 비교

해시코드를 비교할 때는 단순한 뺄셈 연산은 사용하면 값이 범위를 벗어나 오버플로우가 발생할 수 있다. `Integer.compare()`를 활용해서 상대적인 차이만 구하는 것이 안전하다.

```java
int hashCode1 = object1.hashCode();
int hashCode2 = object2.hashCode();

// 안전한 비교 방식
int result = Integer.compare(hashCode1, hashCode2);
```