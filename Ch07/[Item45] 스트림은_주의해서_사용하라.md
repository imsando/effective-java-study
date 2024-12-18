# 스트림은 주의해서 사용하라

## ⛳️ 목표

스트림 API의 특징과 사용할 때의 주의점을 알아보자

<br>

## 📄 핵심 요약

- 스트림 API를 과도하게 사용하면 오히려 가독성과 유지보수성을 해칠 수 있다.
- 리팩토링을 통해 반복문과 스트림 API 중 어떤 것을 선택해야 하는지 판단하거나 적절히 조합하여 사용해야 한다.

---

## 스트림 API

자바 8에 다량의 데이터 처리 작업(순차적 혹은 병렬적)을 돕고자 추가된 기능이다.

### 핵심 개념

- `스트림(stream)`은 데이터 원소의 유한 혹은 무한 시퀀스를 뜻한다.
- `스트림 파이프라인(stream pipeline)`은 이 원소들로 수행하는 연산 단계를 표현하는 개념이다.

<br>

### 스트림 파이프라인

- 소스 스트림에서 시작해 `종단 연산(ternimal operation)`으로 끝나며, 그 사이에 하나 이상의 `중간 연산(intermediate operation)`이 있을 수 있다.
- 각 중간 연산은 스트림을 어떠한 방식으로 `변환(transform)`한다.
- `지연 평가(lazy evaluation)`된다. 즉, 평가는 종단 연산이 호출될 때 이루어지기 때문에 이 특징으로 무한 스트림을 다룰 수 있게 된다. → ***종단 연산을 잊지 말자!***
- 기본적으로 순차적으로 수행되나 병렬적으로 수행하기 위해서는 `parallel` 메서드를 호출한다.

<br>

### 플루언트 API(Fluent API)

메서드 연쇄를 지원하는 `플루언트 API(fluent API)`다. 파이프 라인 하나를 구성하는 모든 호출을 연결하여 단 하나의 표현식으로 완성할 수 있다.

<br>

## 활용 예제

`computeIfAbsent` 메서드는 맵 안에 키가 있는지 찾은 후 있으면 키에 매핑된 값 출력, 없으면 전달된 함수 객체를 키에 적용하여 값을 계산한 결과를 값으로 저장하고 계산된 값을 반환하는 역할을 한다.

```java
public class Anagrams {
	public static void main(String[] args) throws IOException {
		File dictionary = new File(args[0]);
		int minGroupSize = Integer.parseInt(args[1]);
		
		Map<String, Set<String>> groups = new HashMap<>();
		try (Scanner s = new Scanner(dictionary)) {
			while (s.hasNext()) {
				String word = s.next();
				**groups.computeIfAbsent(alphabetize(word), 
					(unused) -> new TreeSet<>()).add(word);**
			}
		}
		
		for (Set<String> group : groups.values())
			if (group.size() >= minGroupSize)
				System.out.println(group.size() + ": " + group);
	}
	
	private static String alphabetize(String s) {
		char[] a = s.toCharArray();
		Arrays.sort(a);
		return new String(a);
	}
}
```

<br>

### Bad case:: 스트림을 과하게 사용한 경우

```java
public class Anagrams {
	public static void main(String[] args) throws IOException {
		Path dictionary = Paths.get(args[0]);
		int minGroupSize = Integer.parseInt(args[1]);
		
		try (Stream<String> words = Files.lines(dictionary)) {
			words.collect(
				groupingBy(word -> word.chars().sorted()
					.collect(StringBuilder::new, (sb, c) ->
						StringBuilder::append).toString()
				)
				.values().stream()
				.filter(group -> group.size() >= minGroupSize)
				.map(group -> group.size() + ": " + group)
				.forEach(System.out::println);
			)
		}
	}
}
```

- 스트림을 과용하면 프로그램이 읽거나 유지보수하기 어려워진다.

<br>

### Good case::스트림을 적절히 사용한 경우

```java
public class Anagrams {
	public static void main(String[] args) throws IOException {
		Path dictionary = Paths.get(args[0]);
		int minGroupSize = Integer.parseInt(args[1]);
		
		try (Stream<String> words = Files.lines(dictionary)) {
			words.collect(groupingBy(word -> alphabetize(word)))
				.values().stream()
				.filter(group -> group.size() >= minGroupSize)
				.forEach(group -> System.out.println(group.size() + ": " + group));
		}
	}
	// alphabetize 생략
}
```

- 첫 스트림의 파이프라인에는 중간 연산이 없으며, 종단 연산에서 모든 단어를 수집해 맵으로 모은다.
- `values()`가 반환한 값으로부터 새로운 스트림을 열고, 필터링을 연산한 후 종단 연산인 forEach에서 리스트를 출력한다.
- 람다 매개변수의 이름을 잘 지어야 파이프라인의 가독성이 유지된다!

<br>

### Question case::char를 다루어야 하는 경우

자바는 기본 타입인 char용 스트림을 지원하지 않는다.

```java
"Hello world!".chars().forEach(x -> System.out.println((char) x));
```

- `chars()`가 반환하는 스트림 원소는 int 이므로 형변환을 해줘야만 한다.
- char 값들을 처리할 때는 스트림을 삼가는 편이 낫다.

<br>

## 스트림을 사용해야 할 때

- 기존 코드는 스트림을 사용하도록 리팩토링하되, 새 코드가 더 나아 보일 때만 반영하자.
- 무조건적인 사용은 코드 가독성과 유지보수 측면에서 손해를 볼 수도 있다.

<br>

### 적용하기 좋은 경우

- 원소들의 시퀀스를 일관되게 변환한다.
- 원소들의 시퀀스를 필터링한다.
- 원소들의 시퀀스를 하나의 연산을 사용해 결합한다(더하기, 연결하기, 최솟값 구하기 등).
- 원소들의 시퀀스를 컬렉션에 모은다(아마도 공통된 속성을 기준으로 묶어가며).
- 원소들의 시퀀스에서 특정 조건을 만족하는 원소를 찾는다.

<br>

### 적용하기 좋지 않은 경우

- 한 데이터가 파이프라인의 여러 단계(stage)를 통과할 때 이 데이터의 각 단계에서의 값들에 동시에 접근해야 할 때

<br>

```
<정리>
반복문과 스트림 중 어떤 것을 골라야 하느냐에 대한 정답은 개인 취향, 프로그래밍 환경의 문제에 따라 다르다는 것이다. 스트림과 함수형 프로그래밍에 익숙하다면 스트림으로 작성했을 때 코드가 명확하게 보이지만 그렇지 않은 경우에는 짧은 코드라도 오히려 가독성이 떨어지게 느껴질 수 있다. 잘 모르겠다면 우선 둘 다 해보고 더 나은 쪽을 택하자!

```