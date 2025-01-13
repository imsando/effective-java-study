# 전통적인 for 문보다는 for-each 문을 사용하라

## ⛳️ 목표

전통적인 for문과 비교했을 때 for-each문의 장점을 알아보자

<br>

## 📄 핵심 요약

***for-each문은 전통적인 for문에 비해***

- 가독성이 좋으며 오류를 방지할 수 있다.
- 하나의 관용구로 컬렉션과 배열을 모두 처리할 수 있다.
- 반복문이 중첩됐을 때의 실수를 줄일 수 있다.

---

## 전통적인 for문의 문제점

```java
// 컬렉션
for (Iterator<Element> i = c.iterator(); i.hasNext();) {
	Element e = i.next();
	... // e로 무언가를 한다.
}

// 배열
for (int i = 0; i < a.length; i++) {
	... // a[i]로 무언가를 한다.
}
```

- 반복자와 인덱스 변수가 코드의 가독성을 떨어뜨린다.
- 쓰이는 요소 종류가 늘어나면 오류가 생길 가능성이 높아진다.

<br>

## for-each문(향상된 for문)의 장점

```java
for (Element e : elements) {
	... // e로 무언가를 한다.
}

// 반복문 중첩
for (Suit suit : suits) {
	for (Rank rank : ranks) {
		deck.add(new Card(suit, rank));
	}
}
```

- 반복자와 인덱스 변수를 사용하지 않아 코드가 깔끔하며 오류를 방지할 수 있다.
- 하나의 관용구로 컬렉션과 배열을 모두 처리할 수 있으며 속도도 그대로이다.
- 반복문을 중첩해서 사용할 때의 실수를 줄일 수 있다.

<br>

## for-each문을 사용할 수 없는 경우

**[1] 파괴적인 필터링(destructive filtering)**

- 컬렉션을 순회하면서 선택된 요소를 제거해야 할 때 반복자의 remove 메서드를 호출해야 한다.
- 자바 8부터는 Collection의 removeIf 메서드를 사용해 컬렉션을 명시적으로 순회하는 일을 피할 수 있다.

<br>

**[2] 변형(transforming)**

- 리스트나 배열을 순회하면서 그 원소의 값 일부 혹은 전체를 교체해야 한다면 인덱스를 사용해야 한다.

<br>

**[3] 병렬 반복(parallel iteration)**

- 여러 컬렉션을 병렬로 순회해야 한다면 각각의 반복자와 인덱스 변수로 명시적으로 제어해야 한다.

<br>

```text
핵심 정리

전통적인 for문과 비교했을 때 for-each문은 명료하고, 유연하고, 버그를 예방해준다. 성능 저하도 없다. 가능한 모든 곳에서 for문이 아닌 for-each문을 사용하자.
```
