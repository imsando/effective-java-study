# wait과 notify보다는 동시성 유틸리티를 애용하라

## ⛳️ 목표

동시성 컬렉션과 동기화 장치를 알아보자

<br>

## 📄 핵심 요약

- 동시성 컬렉션은 자체적으로 동기화를 수행하며, `ConcurrentHashMap`과 `BlockingQueue` 같은 구조를 제공한다.
- 동기화 장치는 스레드 간 작업 조율을 돕고, `CountDownLatch`, `CyclicBarrier`, `Phaser` 등이 있다.
- 레거시 코드에서는 `wait`, `notify`를 사용할 수 있다. `notify`보다는 `notifyAll`을 사용하자.

---

> java.util.concurrent의 고수준 유틸리티는 세 범주로 나눌 수 있다.
바로 실행자 프레임워크, 동시성 컬렉션, 동기화 장치다.
>

## 동시성 컬렉션(concurrent collection)

List, Queue, Map 같은 표준 컬렉션 인터페이스에 동시성을 가미해 구현한 고성능 컬렉션이다. 높은 동시성에 도달하기 위해 동기화를 각자의 내부에서 수행한다.

- 동시성 컬렉션에서 동시성을 무력화하는 것은 불가능하다.
    - 여러 기본 동작을 하나의 원자적 동작으로 묶는 `상태 의존적 수정` 메서드들의 추가되었다.
- 외부에서 락을 추가로 사용하면 오히려 속도가 느려진다.

<br>

**[1] Map.putIfAbsent(key, value)**

```java
private static final ConcurrentMap<String, String> map = 
	new ConcurrentHashMap<>();
	
public static String intern(String s) {
	String result = map.get(s); // ConcurrentHashMap은 검색 기능에 최적화
	if (result == null) {
		result = map.putIfAbsent(s, s); // 필요할 때만 호출
		if (result == null) {
			result = s;
		}
	}
	return result;
}
```

- 주어진 키에 매핑된 값이 아직 없을 때만 새 값을 집어넣는다.
- 기존 값이 있었다면 그 값을 반환하고, 없었다면 null을 반환한다.
- 이 메서드 덕에 스레드 안전한 정규화 맵(canonicalizing map)을 쉽게 구현할 수 있다.

<br>

> **정리**
> - `ConcurrentHashMap`은 동시성이 뛰어나며 속도도 무척 빠르다.
> - Collections.synchronizedMap(동기화된 맵) 보다는 ConcurrentHashMap(동시성 맵)을 사용하는 게 훨씬 좋다.

<br>

**[2] BlockingQueue**

`Queue`를 확장한 `BlockingQueue`에 추가된 메서드 중 `take`는 큐의 첫 원소를 꺼낸다. 이때 만약 큐가 비어있다면 새로운 원소가 추가될 때까지 기다린다.

- `작업 큐(생산자-소비자 큐)`로 쓰기에 적합하다.
    - **작업 큐** : 하나 이상의 생산자 스레드가 작업을 큐에 추가하고, 하나 이상의 소비자 스레드가 큐에 있는 작업을 꺼내 처리하는 형태
- `ThreadPoolExecutor`를 포함한 대부분의 실행자 서비스 구현체에서 사용한다.

<br>

## 동기화 장치

스레드가 다른 스레드를 기다릴 수 있게 하여, 서로 작업을 조율할 수 있게 해준다.

<br>

**[1] CountDownLatch**

**일회성 장벽**으로, 하나 이상의 스레드가 또 다른 하나 이상의 스레드 작업이 끝날 때까지 기다리게 한다.

- 유일한 생성자는 int 값을 받는다. 이 값이 래치의 countDown 메서드를 몇 번 호출해야 대기 중인 스레드들을 깨우는지 결정한다.

<br>

***예제 코드***

```java
public static long time(Executor executor, int concurrency,
	Runnable action) throws InterruptedException {
	CountDownLatch ready = new CountDownLatch(concurrency);
	CountDownLatch start = new CountDownLatch(1);
	CountDownLatch done = new CountDownLatch(concurrency);
	
	for (int i = 0; i < concurrency; i++) {
		executor.execute(() -> {
			// 타이머에게 준비를 마쳤음을 알린다.
			ready.countDown();
			try {
				// 모든 작업자 스레드가 준비될 때까지 기다린다.
				start.await();
				action.run();
			} catch (InterruptedException e) {
				Thread.currentThread().interrupt();
			} finally {
				// 타이머에게 작업을 마쳤음을 알린다.
				done.countDown();
			}
		});
	}	
	
	ready.await(); // 모든 작업자가 준비될 때까지 기다린다.
	long startNanos = System.nanoTime();
	start.countDown(); // 작업자들을 깨운다.
	done.await(); // 모든 작업자가 일을 끝마치기를 기다린다.
	return System.nanoTime() - startNanos;
}
```

- `스레드 기아 교착상태(Thread starvation deadlock)`
    - time 메서드에 넘겨진 `실행자(executor)`는 `concurrency` 매개변수로 지정한 동시성 수준만큼의 스레드를 생성할 수 있어야 하는데 그렇지 못해 메서드가 종료되지 않는 것
- 시간 간격을 잴 때는 항상 `System.currentTimeMillis`가 아닌 `System.nanoTime`을 사용한다.
    - 더 정확하고 정밀하며 시스템의 실시간 시계의 시간 보정에 영향받지 않는다.
- 이 예제의 코드는 1초 이상 걸리지 않는다면 정확한 시간을 측정할 수 없다.
    - 꼭 해야 한다면 `jmh` 같은 특수 프레임워크를 사용해야 한다.
- 위의 카운트다운 래치 3개는 `CyclicBarrier`(혹은 `Phaser`) 인스턴스 하나로 대체할 수 있다. (난이도 Up)

<br>

## wait과 notify, notifyAll

새로운 코드라면 `wait`과 `notify`가 아닌 동시성 유틸리티를 써야 한다. 하지만 어쩔 수 없이 레거시 코드를 다뤄야 할 때도 있다.

<br>

**[1] wait**

```java
synchronized (obj) {
	while (<조건이 충족되지 않았다>)
		obj.wait(); // (락을 놓고, 깨어나면 잡는다.)
	
	... // 조건이 충족됐을 때의 동작을 수행한다.
}
```

- 스레드가 어떤 조건이 충족되기를 기다리게 할 때 사용한다.
- 락 객체의 wait 메서드는 반드시 그 객체를 잠근 동기화 영역 안에서 호출해야 한다.
- wait 메서드를 사용할 때는 반드시 대기 반복문(wait loop) 관용구를 사용해야 한다. wait 호출 전후로 조건이 만족하는지를 검사한다.
- 대기 전에 조건을 검사하여 이미 충족된다면 wait을 건너뛰게 한 것은 응답 불가 상태를 예방하는 조치다.

<br>

**[2] notify vs notifyAll**

- `notify` : 스레드 하나만 깨운다.
- `notifyAll` : 모든 스레드를 깨운다.
- 일반적으로 항상 notifyAll을 사용하는 게 합리적이고 안전하다. 깨어나야 하는 모든 스레드가 깨어남을 보장하기에 항상 정확한 결과를 얻을 수 있게 때문이다.
- 모든 스레드가 같은 조건을 기다리고, 조건이 한 번 충족될 대마다 단 하나의 스레드만 혜택을 받을 수 있다면 notifyAll 대신 notify를 사용해 최적화할 수 있다.
- notify 대신 notifyAll을 사용하면 관련 없는 스레드가 실수로 혹은 악의적으로 wait을 호출하는 공격으로부터 보호할 수 있다.