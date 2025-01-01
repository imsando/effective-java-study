# 적시에 방어적 복사본을 만들라

## ⛳️ 목표

클라이언트에 의해 불변식이 깨지지 않게 하기 위한 방법을 알아보자

<br>

## 📄 핵심 요약

- 가변인 내부 객체를 클라이언트에 반환할 때 안심할 수 없다면 방어적 복사를 해야 한다.
- 복사 비용이 크거나 성능 저하가 우려될 때는 클라이언트의 책임을 문서에 명시하고 수정을 허용하게 둘 수 있다.

---

## 불변식이 깨지는 경우

자바는 안전한 언어이나 클라이언트로 인해 불변식이 깨질 수 있다.

<br>

### Case 1::생성자를 통한 수정

아래와 같이 Period 클래스가 있을 때 불변식이 깨지는 클라이언트 호출 코드를 살펴보자.

**[예제 코드]**

***Period.class***

```java
public final class Period {
	private final Date start;
	private final Date end;
	
	public Period(Date start, Date end) {
		if (start.compareTo(end) > 0)	
			throw new IllegalArgumentException(start + "가" + end + "보다 늦다.");
		this.start = start;
		this.end = end;
	}
	
	public Date start() {
		return start;
	}
	
	public Date end() {
		return end;
	}
		
}
```

***클라이언트 호출 코드***

```java
Date start = new Date();
Date end = new Date();
Period p = new Period(start, end);
end.setYear(78); // p의 내부 수정
```

<br>

**[해결 방법 1]**

- 자바 8 이후에는 Date 대신 불변인 Instant를 사용하면 된다.(혹은 LocalDateTime이나 ZoneDateTime을 사용해도 된다).
- Date처럼 가변인 값 타입은 새로운 코드를 작성할 때는 사용하지 말자.

<br>

**[해결 방법 2]**

생성자에서 받은 가변 매개변수 각각을 방어적으로 복사(defensive copy)하자.

```java
public Period(Date start, Date end) {
	this.start = new Date(start.getTime());
	this.end = new Date(end.getTime());
	
	if (this.start.compareTo(this.end) > 0)
		throw IllegalArgumentException(this.start + "가" + this.end + "보다 늦다.");
}
```

- 매개변수의 유효성을 검사하기 전에 방어적 복사본을 만들고, 이 복사본으로 유효성을 검사한다.
- 원복 객체의 유효성을 검사한 후 복사본을 만드는 찰나에 다른 스레드가 원본 객체를 수정할 위험을 방지하기 위해서이다.
    - `검사시점/사용시점(time-of-check/time-of-use) 공격` 혹은 `TOCTOU 공격`이라고 한다.
- 매개변수가 제3자에 의해 확장될 수 있는 타입이라면 방어적 복사본을 만들 때 clone을 사용하면 안 된다.

<br>

### Case 2::접근자 메서드를 통한 수정

접근자 메서드가 내부의 가변 정보를 직접 드러내기 때문에 여전히 수정이 가능하다.

```java
Date start = new Date();
Date end = new Date();
Period p = new Period(start, end);
p.end().setYear(78); 
```

<br>

**[해결 방법]**

접근자가 가변 필드의 방어적 복사본을 반환하게 한다.

```java
public Date start() {
	return new Date(start.getTime());
}

public Date end() {
	return new Date(end.getTime());
}
```

- 생성자에 이어 접근자 메서드까지 수정하면 Period 자신 말고는 가변 필드에 접근할 방법이 없어지게 된다.
- 생성자와 달리 접근자 메서드에서는 방어적 복사에 clone을 사용해도 된다. Period가 가지고 있는 Date 객체는 [Java.util.Date](http://Java.util.Date)임이 확실하기 때문이다.

<br>

## 정리

- 클래스가 불변이든 가변이든 가변인 내부 객체를 클라이언트에 반환할 때 안심할 수 없다면 원본을 노출하지 말고 방어적 복사본을 반환해야 한다.
- 방어적 복사에는 성능 저하가 따르고, 또 항상 쓸 수 있는 것이 아니기에 해당 구성요소를 수정했을 때의 책임이 클라이언트에 있음을 문서에 명시하는 방법을 선택할 수도 있다.