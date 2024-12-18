# 스트림에서는 부작용 없는 함수를 사용하라

## ⛳️ 목표

스트림 패러다임을 이해하며 올바른 스트림 사용을 위한 Collector를 알아보자

<br>

## 📄 핵심 요약

- 스트림 계산은 일련의 변환 단계로 이루어져야 하는데 이때 순수함수를 사용한다.
- Collector을 사용하면 스트림의 원소들을 쉽게 컬렉션으로 모을 수 있다.

---

## 스트림 패러다임

- 스트림이 제공하는 표현력, 속도, 병렬성을 얻으려면 API는 물론 패러다임까지 함께 받아들여야 한다.
- 스트림 패러다임의 핵심은 계산을 일련의 `변환(transformation)`으로 재구성하는 부분이다.
    - 이때 각 변환 단계는 가능한 한 이전 단계의 결과를 받아 처리하는 `순수 함수`여야 한다.
    - 순수 함수란 **오직 입력만이 결과에 영향을 주는 함수**를 말한다.

<br>

### Bad case::스트림 패러다임을 이해하지 못한 경우

```java
Map<String, Long> freq = new HashMap<>();
try (Stream<String> words = new Scanner(file).tokens() {
	words.forEach(word -> {
		freq.merge(word.toLowerCase(), 1L, Long::sum);
	})
}
```

- 스트림 코드를 가장한 반복적 코드다.
- 모든 작업이 종단 연산인 `forEach`에서 일어나는데 이때 외부 상태를 수정한다.
    - 스트림이 수행한 연산 결과를 보여주는 것 그 이상의 일을 수한다.
    - `forEach` 연산은 스트림 계산 결과를 보고할 때만 사용하고, 계산하는 데는 쓰지 말자!

<br>

### Good case::스트림 패러다임을 이해한 경우

```java
Map<String, Long> freq;
try (Stream<String> words = new Snanner(file).tokens()) {
	freq = words
		.collect(groupingBy(String::toLowerCase, counting()));
}
```

- 스트림 API를 제대로 사용하였으며 코드가 짧고 명확하다.
- `Scanner`의 스트림 메서드인 `tokens`를 사용해 스트림을 얻는다. 자바 9부터 지원한다.
- 이 코드에서는 `수집기(collector)`를 사용했다.

<br>

## Collector

`collector`가 생성하는 객체는 일반적으로 컬렉션이며, 스트림의 원소를 손쉽게 컬렉션으로 모을 수 있다.

- `e.g.` toList(), toSet(), toCollection(collectionFactory)

<br>

### [1] toList()

```java
List<String> topTen = freq.keySet().stream()
	.sorted(comparing(freq::get).reversed())
	.limit(10)
	.collect(toList());
```

- comparing 메서드는 키 추출 함수를 받는 비교자 생성 메서드드다.
- 비교자(comparing)을 역순으로 정렬하기 위해 sorted 내부에 reversed가 사용됐다.

<br>

### [2] toMap(keyMapper, valueMapper)

스트림의 각 원소가 고유한 키에 매핑되어 있을 때 적합하다.

```java
private static final Map<String, Operation> stringToEnum =
	Stream.of(values()).collect(
		toMap(Object::toString, e -> e));
```

<br>

**[2-1] 병합 함수 `BinaryOperator<U>` 제공**

- U는 해당 맵의 값 타입이다.
- 예로 병합 함수가 곱셈이라면 키가 같은 모든  값을 곱한 결과를 얻는다.

***예제***

```java
Map<Artist, Album> topHits = albums.collect(
	toMap(Album::artist, a -> a, maxBy(comparing(Album::sales))));
```

- 비교자로는 BinaryOperator에서 정적 임포트한 `maxBy`를 사용했다.
    - Comparator<T>를 입력받아 BinaryOperator<T>를 돌려준다.
- 앨범 스트림을 맵으로 바꾸는데, 이 맵은 각 음악가와 그 음악가의 베스트 앨범을 key, value로 갖는다.

<br>

### **[3] groupingBy**

입력으로 분류 함수(classfier)를 받고 출력으로는 원소들을 카테고리별로 모아 놓은 맵을 담은 수집기를 반환한다.

```java
words.collect(groupingBy(word -> alphabetize(word)))
```

- 알파벳화 결과가 같은 단어들의 리스트로 매핑하는 맵을 생성했다.

<br>

**[3-1] 다운스트림 수집기**

- 리스트 외의 값을 갖는 맵을 생성하게 하려면 분류 함수와 함께 `다운스트림(downstream)` 수집기도 명시해야 한다.
- 해당 카테고리의 모든 원소를 담은 스트림으로부터 값을 생성하는 역할을 한다.

```java
Map<String, Long> freq = words
	.collect(groupingBy(String::toLowerCase, counting()));
```

- 각 카테고리(키)를 해당 카테고리에 속하는 원소의 개수(값)와 매핑한 맵을 얻는다.

<br>

### [4] joining

CharSequence 인스턴스의 스트림에만 적용할 수 있다.

- 매개변수가 없는 joining은 단순히 원소들을 연결하는 수집기를 반환한다.
- 구분문자를 매개변수로 받는다면 구분 문자와 함께 원소들을 연결한다.

<br>

```
[핵심 정리]
스트림 파이프라인 프로그래밍의 핵심은 부작용 없는 함수 객체에 있다. 스트림뿐 아니라 스트림 관련 객체에 건네지는 모든 함수 객체가 부작용이 없어야 한다. 종단 연산 중 forEach는 스트림이 수행한 계산 결과를 보고할 때만 이용해야 한다. 계산 자체에는 이용하지 말자. 스트림을 올바로 사용하려면 수집기를 잘 알아둬야 한다. 가장 중요한 수집기 팩터리는 toList, toSet, toMap, groupingBy, joining이다.
```
