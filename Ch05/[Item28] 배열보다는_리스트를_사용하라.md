# 배열보다는 리스트를 사용하라

## ⛳️ 목표

- 배열과 제네릭의 차이를 이해하자

<br>

## 📄 핵심 요약

***배열과 제네릭의 차이점***

1. **배열은 공변(Covariant)**
> - 배열은 상위타입과 하위타입이 함께 변한다 (공변)
> - 배열은 하위 타입 관계를 그대로 유지하여 `Sub[]` 타입이 `Super[]` 타입의 하위 타입으로 간주된다
> 예를 들어 `Long[]`은 `Object[]`의 하위 타입이 된다
2. **제네릭은 불공변(Invariant)**
> - 리스트는 상위타입과 하위타입이 함께 변하지 않는다 (불공변)
> - 제네릭은 서로 다른 타입 간에 상하위 관계가 없으며
> 예를 들어 `List<String>`은 `List<Object>`의 하위 타입도 상위 타입도 아니다

<br>    

### 1. 배열과 제네릭의 주요 차이점

> - **배열은 런타임 타입 확인**을 지원하므로 배열에 잘못된 타입의 값을 넣으려고 하면 `ArrayStoreException`이 발생한다
> - **제네릭은 컴파일타임에만 타입 검사**가 가능하며 런타임에는 타입 정보가 소거(erasure)된다
> - 따라서 제네릭 타입에서는 런타임 타입 확인이 불가능하다

<br>    

### 2. 배열의 문제점

- 배열을 사용할 때 **잘못된 타입이 저장되는 것을 런타임에야 알 수 있다**는 점에서 문제가 발생할 수 있다
- 반면 제네릭을 사용하면 컴파일 시점에 타입 오류를 발견할 수 있어 더 안전하다

예시 코드
```java
// 배열은 공변으로 런타임에 오류 발생 가능
Object[] objectArray = new Long[1];
objectArray[0] = "타입이 달라 넣을 수 없다."; // ArrayStoreException 발생

// 제네릭은 컴파일 시점에 오류 발생
List<Object> list = new ArrayList<Long>(); // 컴파일 오류 발생
```
<br>    

### 3. 제네릭 배열을 만들 수 없는 이유

- **타입 소거**: 제네릭의 타입 정보는 컴파일 후 소거되어 런타임에는 알 수 없기 때문에 `T[]`와 같은 제네릭 배열을 생성할 수 없다
- **타입 안전성**: 제네릭 배열을 허용하면 컴파일러가 자동 생성한 형변환 코드에서 런타임에 `ClassCastException`이 발생할 수 있다

<br>

### 4. 제네릭 배열 생성의 위험성

예시코드
```java
// 제네릭 배열 생성 시 컴파일 오류 발생
List<String>[] stringLists = new List<String>[1]; // (1) 컴파일 오류 발생
List<Integer> intList = List.of(42);
Object[] objects = stringLists; // (3)
objects[0] = intList;           // (4)
String s = stringLists[0].get(0); // (5) ClassCastException 발생
```

코드설명
- `List<String>[]` 타입의 배열을 생성한 후, 잘못된 타입을 저장해 런타임에 `ClassCastException`이 발생할 수 있다
- 이로 인해 **제네릭 배열 생성은 컴파일 시점에 금지**된다

<br>    

### 5. 제네릭과 배열의 타입 안전성 문제 해결: 리스트 사용

- 제네릭 타입에서는 배열 대신 **리스트를 사용하는 것이 타입 안전성을 보장**하는데 유리하다 
- 배열보다 조금 더 복잡해지고 성능이 떨어질 수 있지만 타입 안정성을 높여 `ClassCastException`을 예방할 수 있다

예시코드 : 제네릭을 사용하여 배열 대신 리스트를 사용하는 Chooser 클래스
```java
// Chooser 클래스 - 리스트 사용으로 타입 안전성 확보
public class Chooser<T> {
    private final List<T> choiceList;

    public Chooser(Collection<T> choices) {
        choiceList = new ArrayList<>(choices); // 리스트에 컬렉션 복사
    }

    public T choose() {
        Random rnd = ThreadLocalRandom.current();
        return choiceList.get(rnd.nextInt(choiceList.size())); // 리스트에서 무작위로 선택
    }
}
```
<br>    

### 6. 주의사항

- 제네릭 타입과 배열을 혼용하여 사용해야 하는 경우가 생기면 **배열을 리스트로 대체**하는 것이 가장 안전하다

<br>

> **💡핵심 정리**
> 1. 배열과 제네릭의 타입 규칙은 다르며 배열은 공변이고 실체화되지만 제네릭은 불공변이고 소거된다
> 2. 타입 안정성을 위해 배열 대신 제네릭 타입의 리스트를 사용하는 것이 좋다