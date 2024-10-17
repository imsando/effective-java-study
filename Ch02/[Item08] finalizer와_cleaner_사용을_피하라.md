# finalizer와 cleaner 대신 try-with-resources 를 사용하자


## ⛳️ 목표

- `finalizer`와 `cleaner`의 문제점을 이해하고 효율적인 리소스 관리 방식을 익히자
  <br>

## 📄 핵심 요약

***finalizer 란?***


- 자바 객체가 가비지 컬렉터에 의해 회수되기 전에 호출될 수 있는 메서드이다
- 자바의 Object 클래스에 기본적으로 정의된 finalize() 메서드를 오버라이드하여 자원 해제나 정리 작업을 수행할 수 있다
- 동작 방식 : 객체가 더 이상 참조되지 않아 가비지 컬렉션 대상이 될 때 자바 런타임은 그 객체의 finalize() 메서드를 호출하여 리소스 정리를 진행한다

```java
@Override
protected void finalize() throws Throwable {
    try {
        // 자원 해제 코드
    } finally {
        super.finalize();  // 부모 클래스의 finalize 호출
    }
}
```

***cleaner 란?***

- 자바 9에서 도입된 자원 정리 메커니즘이다
- finalizer보다 안전하게 설계된 대체 방법이다
- 백그라운드 스레드에서 실행되어 객체가 가비지 컬렉션 대상으로 확인되면 정리 작업을 수행한다
- 동작 방식 : Cleaner 클래스를 사용해 cleanable 객체를 등록하고 객체가 더 이상 참조되지 않으면 백그라운드에서 등록된 cleaner가 호출되어 자원을 해제한다

```java
class MyResource implements AutoCloseable {
    private static final Cleaner cleaner = Cleaner.create();

    private static class State implements Runnable {
        @Override
        public void run() {
            // 자원 해제 작업
        }
    }

    private final State state;
    private final Cleaner.Cleanable cleanable;

    MyResource() {
        state = new State();
        cleanable = cleaner.register(this, state);
    }

    @Override
    public void close() {
        cleanable.clean();  // 명시적으로 자원 해제
    }
}
```

***

### 1. finalizer의 문제점

`finalizer`는 가비지 컬렉션 중에 호출되는 메서드로 자원 해제와 같은 정리 작업을 하지만 여러 문제점이 있다

> - **⓵ 비효율성** : `finalizer`가 있는 객체는 가비지 컬렉션 과정에서 추가적인 단계가 필요하므로 성능 저하를 유발할 수 있다
> - **② 예측 불가능성** : `finalize()`가 호출되는 시점을 예측할 수 없으므로 중요한 자원을 즉시 해제할 수 없으며 시스템 자원을 낭비할 수 있다
> - **⓷ 안정성 문제** : `finalize()`에서 예외가 발생할 경우 그 예외는 무시되어 다른 문제가 발생할 수 있다

<br>

### 2. cleaner의 문제점

`cleaner`는 `finalizer`의 단점을 개선한 대체 방법으로 별도의 스레드에서 객체의 정리 작업을 수행한다
하지만 여전히 성능 저하와 예측 불가능성 문제에서 자유롭지 않다

> - **⓵ 백그라운드에서 실행** : `cleaner`는 별도의 스레드에서 실행되므로 중요한 자원 해제를 즉시 처리하기에 부적합하다
> - **② 추가적인 복잡성** : 자원을 정리하기 위한 메커니즘을 더 추가해야 하므로 코드가 복잡해질 수 있다

<br>

### 3. 리소스 관리 방법 : `try-with-resources`

- 자바 7에서 도입된 `try-with-resources`는 명시적으로 자원을 관리하는 방법이다
- `AutoCloseable` 인터페이스를 구현한 객체는 이 구문을 통해 자동으로 자원이 해제된다
- `try-with-resources`는 예외가 발생하더라도 자원을 안전하게 해제할 수 있으며 코드가 간결하고 명확하다

```java
try (InputStream in = new FileInputStream("file.txt")) {
    // 파일 읽기 작업
} catch (IOException e) {
    e.printStackTrace();
}
```

<br>

🏷️ **NOTE**

> `AutoCloseable`을 구현한 자원들은 `try-with-resources` 블록이 종료될 때 자동으로 `close()` 메서드가 호출되어 자원을 안전하게 해제할 수 있다

<br>

### 4. 정리

- `finalizer`와 `cleaner`는 자원 관리에 있어서 불필요한 위험과 성능 저하를 야기할 수 있다
- 자바의 최신 버전에서는 `try-with-resources`를 사용하여 명시적으로 자원을 관리하는 것이 더 안전하고 효율적이다