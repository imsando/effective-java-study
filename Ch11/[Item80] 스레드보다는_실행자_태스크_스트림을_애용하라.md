## ⛳️ 목표

- 스레드보다 실행자(Executor), 태스크(Task), 스트림(Stream)을 활용하는 것이 좋은 이유에 대해 알아보자.

<br>

## 📄 핵심 요약

#### **실행자(Executor)란?**
- 태스크(작업)를 실행하는 높은 수준의 메커니즘을 제공하는 프레임워크이다.
- 직접 스레드를 생성하고 관리하는 것보다 효율적이며, `ExecutorService` 인터페이스를 통해 다양한 실행 전략을 제공한다.

#### **ThreadPoolExecutor란?**
- 실행자 프레임워크의 핵심 클래스로, 스레드 풀(Thread Pool)을 관리하고 실행하는 기능을 제공한다.
- 스레드 개수, 큐 크기 등을 조절할 수 있어, 원하는 실행 정책을 적용할 수 있다.

#### **CachedThreadPool이란?**
- `Executors.newCachedThreadPool()`을 통해 생성되는 실행자로, 필요한 만큼 스레드를 생성하고 재사용한다.
- 적절한 제한 없이 무분별하게 스레드가 생성될 수 있어, 무거운 프로덕션 서버에서는 적절하지 않다.

#### **Runnable과 Callable이란?**
- `Runnable`은 반환값이 없는 태스크를 정의하는 인터페이스이다.
- `Callable`은 `Runnable`과 유사하지만, 값을 반환하며 예외를 던질 수 있다.

#### **포크-조인(Fork-Join)이란?**
- 큰 작업을 작은 단위로 나누어 병렬 처리할 수 있도록 돕는 프레임워크이다.
- `ForkJoinPool`을 활용해 효율적으로 CPU를 사용하며, 자바의 병렬 스트림(Parallel Stream)에서도 사용된다.

---
<br>

### 1. **스레드보다 실행자를 사용해야 하는 이유**

#### 1️⃣ **작업 단위와 실행 메커니즘의 분리**
- 실행자 프레임워크는 태스크(Task)와 실행 메커니즘을 분리하여 관리한다.
- 이를 통해 실행 정책을 유연하게 조정할 수 있다.

#### 2️⃣ **효율적인 스레드 관리**
- 스레드를 직접 다루면 과도한 생성과 소멸이 발생할 수 있으며, 실행자 서비스는 이를 효율적으로 관리한다.
- 스레드 풀을 사용하면 필요한 만큼의 스레드만 유지하면서 성능을 최적화할 수 있다.

#### 3️⃣ **우아한 종료 처리**
- `shutdown()` 및 `awaitTermination()`을 사용하여 작업이 완료된 후 안전하게 실행자를 종료할 수 있다.
- 직접 스레드를 다루는 경우, 명시적인 종료 처리가 어려울 수 있다.

<br>

### 2. **실행자 서비스(ExecutorService) 활용 예제**
#### ✅ 단일 스레드 실행자 생성
```java
ExecutorService executor = Executors.newSingleThreadExecutor();
executor.execute(() -> System.out.println("작업 수행"));
executor.shutdown();
```
- 단일 스레드에서 태스크를 실행하고 안전하게 종료한다.

#### ✅ 고정 크기 스레드 풀 사용
```java
ExecutorService executor = Executors.newFixedThreadPool(5);
for (int i = 0; i < 10; i++) {
    executor.execute(() -> System.out.println(Thread.currentThread().getName()));
}
executor.shutdown();
```
- 5개의 스레드만 사용하여 작업을 처리한다.

#### ✅ 스케줄된 실행자 활용
```java
ScheduledExecutorService scheduler = Executors.newScheduledThreadPool(2);
scheduler.schedule(() -> System.out.println("3초 후 실행"), 3, TimeUnit.SECONDS);
scheduler.shutdown();
```
- 특정 시간 후 실행하는 태스크를 등록할 수 있다.

<br>

### 3. **포크-조인 프레임워크 활용**
#### ✅ 병렬 처리를 위한 포크-조인 사용 예제
```java
ForkJoinPool pool = new ForkJoinPool();
RecursiveTask<Integer> task = new RecursiveTask<Integer>() {
    @Override
    protected Integer compute() {
        return 42; // 실제 구현에서는 작은 작업으로 분할
    }
};
int result = pool.invoke(task);
System.out.println(result);
```
- `ForkJoinPool`을 이용하여 병렬 연산을 수행한다.

<br>

### 4. **실행자 선택 가이드**

#### 1️⃣ **경량 서비스 또는 테스트 환경**
- `Executors.newCachedThreadPool()` → 제한 없이 스레드를 생성하므로 가벼운 작업에 적합

#### 2️⃣ **무거운 프로덕션 서버**
- `Executors.newFixedThreadPool(n)` → n개의 스레드만 유지하여 리소스를 제한적으로 사용

#### 3️⃣ **스케줄링이 필요한 경우**
- `Executors.newScheduledThreadPool(n)` → 일정 간격으로 작업을 실행하는 경우 유용

#### 4️⃣ **대량 병렬 작업 처리**
- `ForkJoinPool` → 큰 작업을 작은 단위로 나눠 병렬 처리하는 경우 적합

<br>

### 💡 **핵심 정리**
- **스레드를 직접 다루기보다 실행자(Executor)를 사용하라.**
- **태스크와 실행 메커니즘을 분리하면 유연성과 확장성이 향상된다.**
- **고정 크기 스레드 풀이나 스케줄링 기능을 적절히 활용하라.**
- **대규모 병렬 작업에는 포크-조인 프레임워크를 고려하라.**
