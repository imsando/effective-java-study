# 타입 안정 이종 컨테이너를 고려하라

## ⛳️ 목표

타입 안정성을 유지하면서 서로 다른 타입의 객체를 저장하고 검색하기 위한 방법을 알아보자

<br>

## 📄 핵심 요약

- `타입 안전 이종 컨테이너`로 타입 안정성을 유지하며 이종 객체를 저장 및 검색할 수 있다.
- `한정적 타입 토큰`과 `asSubclass()`를 활용하여 특정 타입 계층 구조 내에서 안전한 처리가 가능하다.

---

## 타입 안전 이종 컨테이너 패턴  


: **type safe heterogeneous container pattern**

> 제네릭 타입 시스템이 값의 타입이 키와 같음을 보장하게 해주는 설계 방식
> 1. 컨테이너 대신 키를 매개변수화한다.
> 2. 컨테이너에 값을 넣거나 뺄 때 매개변수화한 키를 함께 제공한다.
>

<br>

***[1] API 예제 코드***

타입별로 즐겨 찾는 인스턴스를 저장하고 검색할 수 있는 클래스이다.

```java
public class Favorites {
	public <T> void putFavorite(Class<T> type, T instance);
	public <T> T getFavorite(Class<T> type);
}
```

- 각 타입의 Class 객체를 매개변수화한 키 역할로 사용한다.
- 이 방식이 동작하는 이유는 class의 클래스가 제네릭이기 때문이다.
    - **e.g.** `String.class`의 타입 : `Class<String>`
- `타입 토큰(type token)` : 컴파일 타입 정보와 런타임 타입 정보를 알아내기 위해 메서드들이 주고받는 class 리터럴

<br>

***[2] 클라이언트 예제 코드***

```java
public static void main(String[] args) {
	Favorites favorites = new Favorites();
	
	favorites.putFavorite(String.class, "Java");
	favorites.putFavorite(Integer.class, 0xcafebabe);
	favorites.putFavorite(Class.class, Favorites.class);
	
	String favoriteString = favorites.getFavorite(String.class);
	int favoriteInteger = favorites.getFavorite(Integer.class);
	Class<?> favoriteClass = favorites.getFavorite(Class.class);
	
	system.out.printf("%s %x %s%n, favoriteString, favoriteInteger, favoriteClass);
}
```

- String을 요청했는데 다른 타입을 반환하는 일이 없기 때문에 타입 안전하게 사용할 수 있다.
- 일반적인 맵과 달리 여러 가지 타입의 원소를 담을 수 있다.

<br>

***[3] 구현 예제 코드***

```java
public class Favorites {
	private Map<Class<?>, Object> favorites = new HashMap<>();
	
	// 키와 값 정보가 저장되면 타입 링크(type linkage) 버려짐 
	public <T> void putFavorite(Class<T> type, T instance) { 
		favorites.put(Objects.requireNonNull(type), instance);
	}
	
	// Object 타입을 Class 객체가 가리키는 타입으로 동적 형변환
	public <T> T getFavorite(Class<T> type) {
		return type.cast(favorites.get(type);
	}
}
```

- 맵이 아니라 키가 와일드카드 타입이므로 모든 키가 서로 다른 매개변수화 타입일 수 있다.
- 값 타입이 `Object`이므로 모든 값이 키로 명시한 타입임이 보증되진 않으나 개발자가 이것을 이해하고 사용해야 한다.

<br>

**[3-1] 제약 사항**

`문제 1` 악의적인 클라이언트가 Class 객체를 원시 타입으로 넘기면 Favorites의 타입 안정성이 쉽게 깨진다.

`해결 방법` 값 저장 시 instance의 타입이 type으로 명시된 타입과 같은지 확인하는 동적 형변환을 쓴다.

```java
public <T> void putFavorite(Class<T> type, T instance) {
	favorites.put(Objects.requireNonNull(type), type.cast(instance));
}
```

<br>

`문제 2` 실체화 불가 타입에는 사용할 수 없다.

```java
// 사용 가능한 타입
String, String[]

// 사용 불가능한 타입
List<String>, List<Integer>
```

`해결 방법` 완벽히 만족스러운 방법은 없으나 `슈퍼 타입 토큰(super type token)`으로 시도해 볼 수 있다.

```java
Favorites favorites = new Favorites();

List<String> pets = Arrays.asList("개", "고양이", "앵무");

favorites.putFavorite(new TypeRef<List<String>>(){}, pets);
List<String> listofStrings = favorites.getFavorite(new TypeRef<List<String>>(){});
```

<br>

**[3-2] 한정적 타입 토큰과 asSubclass()**

- `한정적 타입 토큰`
  - 한정적 타입 매개변수나 한정적 와일드카드를 사용하여 표현 가능한 타입을 제한하는 것이다.
  - 이때의 형변환은 비검사이므로 컴파일하면 경고가 뜰 수 있다.
- `asSubclass() `
  - Class 클래스에서 제공하는 **형변환을 안전하고 동적으로 수행해주는 인스턴스 메서드**로 위의 단점을 보완할 수 있게 해준다.

<br>

***asSubclass()***
```java
public Class<? extends U> asSubclass(Class<U> clazz)
```

- 런타임에 `ClassCastException`을 방지한다.
- 캐스팅된 타입이 clazz의 서브타입인지 보장한다.

<br>

```
📢 핵심 정리
컬렉션 API로 대표되는 일반적인 제네릭 형태에서는 한 컨테이너가 다룰 수 있는 타입 매개변수의 수가 고정되어 있다. 하지만 컨테이너 자체가 아닌 키를 타입 매개변수로 바꾸면 이런 제약이 없는 타입 안전 이종 컨테이너를 만들 수 있다. 타입 안전 이종 컨테이너는 Class로 키를 쓰며, 이런 식으로 쓰이는 Class 객체를 타입 토큰이라 한다. 또한, 직접 구현한 키 타입도 쓸 수 있다. 예컨대 데이터베이스의 행(컨테이너)를 표현한 DatabaseRow 타입에는 제네릭 타입인 Column<T>를 키로 사용할 수 있다.
```