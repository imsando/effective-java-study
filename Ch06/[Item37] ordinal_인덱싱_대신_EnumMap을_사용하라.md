# ordinal 인덱싱 대신 EnumMap을 사용하라

## ⛳️ 목표

ordinal 인덱싱을 사용했을 때의 문제점을 파악하고 EnumMap, Stream 활용 방식을 알아보자

<br>

## 📄 핵심 요약

- ordinal 인덱싱은 타입 안전하지 않고 상수의 의미를 파악하기 어렵다.
- EnumMap을 활용하면 타입 안전하게 열거 타입을 다룰 수 있다.
- Stream을 활용해 맵을 관리하면 코드를 줄일 수 있다.

---

## ordinal() 메서드 활용

배열이나 리스트에서 원소를 꺼낼 때 `ordinal` 메서드로 인덱스를 얻는 코드가 있다. 다음은 식물을 간단히 나타낸 클래스이다.

```java
class Plant {
	enum LifeCycle { ANNUAL, PERENNIAL, BIENNIAL }
	
	final String name;
	final LifeCycle lifeCycle;
	
	Plant(String name, LifeCycle lifeCycle) {
		this.name = name;
		this.lifeCycle = lifeCycle;
	}
	
	@Override public String toString() {
		return name;
	}
}
```

<br>

### Bad Case

`ordinal()`을 배열 인덱스로 사용하는 경우이다.

```java
public static void main(String[] args) {
		Set<Plant>[] plantsByLifeCycle =
			(Set<Plant>[]) new Set[Plant.LifeCycle.values().length];

		for (int i = 0; i < plantsByLifeCycle.length; i++) {
			plantsByLifeCycle[i] = new HashSet<>();
		}

		Plant[] garden = {
			new Plant("Cosmos", LifeCycle.ANNUAL), // 코스모스
			new Plant("Chrysanthemum", LifeCycle.PERENNIAL), // 국화
			new Plant("Evening Primrose", LifeCycle.BIENNIAL) // 달맞이꽃
		};

		for (Plant plant : garden) {
			plantsByLifeCycle[plant.lifeCycle.ordinal()].add(plant);
		}

		// 결과 출력
		for (int i = 0; i < plantsByLifeCycle.length; i++) {
			System.out.printf("%s: %s%n", Plant.LifeCycle.values()[i], plantsByLifeCycle[i]);
		}
	}
```

***출력 결과***

```text
ANNUAL: [Cosmos]
PERENNIAL: [Chrysanthemum]
BIENNIAL: [Evening Primrose]
```

- 생애주기 별로 총 3개의 집합을 만든다.
- 정원을 한 바퀴 돌며 각 식물을 해당 집합에 넣는다.
- 집합들을 배열에 넣고 생애주기의 ordinal 값을 그 배열의 인덱스로 사용하려 시도할 수 있다.

<br>

**[문제점]**

- 배열은 제네릭과 호환되지 않으니 비검사 형변환을 수행해야 하고 깔끔히 컴파일되지 않는다.
- 배열은 각 인덱스의 의미를 모르니 출력 결과에 직접 레이블을 달아야 한다.
- 정수는 열거 타입과 달리 타입 안전하지 않기 때문에 잘못된 값을 사용하면 잘못된 동작이 수행된다. 혹은 `ArrayIndexOutOfBoundsException`이 발생한다.

<br>

## EnumMap 활용

열거 타입을 키로 사용하도록 설계한 Map 구현체인 EnumMap을 활용해보자.

<br>

### Good Case

```java
Map<LifeCycle, Set<Plant>> plantsByLifeCycle =
	new EnumMap<>(Plant.LifeCycle.class);

for (Plant.LifeCycle lifecycle : Plant.LifeCycle.values()) {
	plantsByLifeCycle.put(lifecycle, new HashSet<>());
}

Plant[] garden = {
	new Plant("Cosmos", LifeCycle.ANNUAL), // 코스모스
	new Plant("Chrysanthemum", LifeCycle.PERENNIAL), // 국화
	new Plant("Evening Primrose", LifeCycle.BIENNIAL) // 달맞이꽃
};

for (Plant plant : garden) {
	plantsByLifeCycle.get(plant.lifeCycle).add(plant);
}

// 결과 출력
System.out.println(plantsByLifeCycle);
```

***출력 결과***

```text
{ANNUAL=[Cosmos], PERENNIAL=[Chrysanthemum], BIENNIAL=[Evening Primrose]}
```

<br>

**[장점]**

- 코드가 짧고 명료해지며 성능도 비슷하다.
- 안전한 형변환을 사용할 수 있다.
- 맵의 키인 열거 타입이 그 자체로 출력용 문자열을 제공하므로 출력 결과에 직접 레이블을 달지 않아도 된다.
- 배열 인덱스를 계산하는 과정에서 오류가 날 가능성이 없다.

<br>

## Stream 활용

스트림을 사용해 맵을 관리하면 코드를 더 줄일 수 있다. 참고로, EnumMap만 사용했을 때는 식물의 생애주기당 값이 없더라도 하나씩의 중첩 맵을 만들지만 스트림을 사용하면 값이 있는 맵만 만든다.

<br>

### EnumMap을 사용하지 않는 경우

```java
public static void main(String[] args) {
		Plant[] garden = {
			new Plant("Cosmos", LifeCycle.ANNUAL), // 코스모스
			new Plant("Chrysanthemum", LifeCycle.PERENNIAL), // 국화
			new Plant("Evening Primrose", LifeCycle.BIENNIAL) // 달맞이꽃
		};

		System.out.println(Arrays.stream(garden)
			.collect(groupingBy(p -> p.lifeCycle)));
	}
```

- 고유한 맵 구현체를 사용했기 때문에 EnumMap을 써서 얻은 공간과 성능 이점이 사라진다.

<br>

### EnumMap을 사용하는 경우

```java
public static void main(String[] args) {
	Plant[] garden = {
		new Plant("Cosmos", LifeCycle.ANNUAL), // 코스모스
		new Plant("Chrysanthemum", LifeCycle.PERENNIAL), // 국화
		new Plant("Evening Primrose", LifeCycle.BIENNIAL) // 달맞이꽃
	};

	System.out.println(Arrays.stream(garden)
		.collect(groupingBy(p -> p.lifeCycle, 
			() -> new EnumMap<>(LifeCycle.class), toSet())));
}
```

- mapFactory 매개변수에 원하는 맵 구현체를 명시해 호출한다.

<br>

```text
핵심 정리
배열의 인덱스를 얻기 위해 ordinal을 쓰는 것은 일반적으로 좋지 않으니, 대신 EnumMap을 사용하라. 다차원 관계는 EnumMap<…, EnumMap<…>>으로 표현하라. “애플리케이션 프로그래머는 Enum.ordinal을 (웬만해서는) 사용하지 말아야 한다(아이템 35)”는 일반 원칙의 특수 사례이다.
```