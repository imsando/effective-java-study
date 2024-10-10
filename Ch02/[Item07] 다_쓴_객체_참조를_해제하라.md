# 메모리 누수를 막자

## ⛳️ 목표

- **메모리 누수**가 자주 일어나는 곳을 인지하고 미리 예방하는 방법을 익히자

<br>

## 📄 핵심 요약

***메모리 누수***

- 프로그램이 더 이상 사용하지 않는 메모리를 제대로 해제하지 못하고 계속해서 점유하는 상황이다
- 메모리 부족, 성능 저하, 프로그램 비정상 종료 등의 문제를 발생시킨다

***WeakHashMap***

- Java에서 제공하는 특별한 형태의 맵(Map) 자료구조이다
- **약한 참조(Weak Reference)** 를 사용해 메모리 누수를 방지하는 데 유용하다
- 동작 예시

    ```java
        public class WeakHashMapExample {
            public static void main(String[] args) {
                WeakHashMap<Object, String> weakMap = new WeakHashMap<>();

                Object key1 = new Object();  // 강한 참조
                Object key2 = new Object();  // 강한 참조

                weakMap.put(key1, "Value1");
                weakMap.put(key2, "Value2");

                System.out.println("Before GC: " + weakMap);  // 두 엔트리 모두 존재

                // key1을 null로 설정하여 참조 해제
                key1 = null;

                // 강제로 가비지 컬렉션 수행
                System.gc();

                // 가비지 컬렉션 후 weakMap에서 key1은 제거됨
                System.out.println("After GC: " + weakMap);  // key2만 남아 있음
            }
        }
    ```
  
### 1. 메모리 누수 원인 : 스택
- 스택이 자기 메모리를 직접 관리하기 때문이다

> #### ① 객체 참조 유지로 인한 문제
> - 스택에서 요소를 꺼내(pop)더 이상 사용하지 않더라도 배열 내에 해당 인덱스에 대한 참조가 남아 있는 경우 메모리 누수가 발생할 수 있다
>
> #### ② 내부 배열 크기 문제
> - Stack 클래스는 Vector를 상속받아 내부적으로 크기를 동적으로 증가시킨다
> - 스택에서 요소를 꺼내더라도 배열의 크기는 줄어들지 않아 그 공간이 메모리 누수를 유발한다
>
> #### ③ 순환참조 및 콜백 문제
> - 이벤트 리스너나 콜백을 포함하고 있는 경우 순환 참조로 인해 메모리 누수가 발생할 수 있다
> - 객체가 다른 객체나 자신을 참조하는 방식으로 메모리를 해제하지 못하게 하는 상황이 생길 수 있다

<br>

### 2. 메모리 누수 원인 : 캐시
- 유효하지 않은 객체를 캐시에 계속해서 보관할 경우 발생한다

<br>

### 3. 메모리 누수 원인 : 리스너 or 콜백
- 클라이언트가 등록한 콜백을 해지해주지 않는다면 콜백은 계속 쌓이기 때문이다
- 이때 약한참조로 콜백을 저장하면 가비지 컬렉터가 수거해갈 수 있다

<br>

### 4. 방법[1] : null 처리(참조해제)
- 사용이 끝난 참조를 null 처리해주면 된다
- 예시 코드
```java
  public Object pop () {
        if (size = 0)
        throw new EmptyStackException();
        Object result = elements [—sizel;
        elements[size] = null; // 다쓴 참조 해제
        return result;
  ｝
```

<br>

### 5. 방법[2] : WeakHashMap 사용(약한참조)
- WeakHashMap을 사용하면 참조되지 않는 엔트리는 자동으로 가비지 컬렉터에 의해 제거된다
- 캐시에 저장된 데이터가 필요 없을 때 자동으로 삭제되어 메모리 누수를 방지할 수 있다
- ***주의*** 이 방법은 특정한 상황에서만 유용하며 항상 적합한 방법은 아니다
