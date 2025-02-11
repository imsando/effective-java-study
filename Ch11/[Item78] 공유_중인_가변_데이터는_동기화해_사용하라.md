# 공유 중인 가변 데이터는 동기화해 사용하라

## ⛳️ 목표

여러 스레드에 의해 가변 데이터가 공유될 때 동기화 해야 하는 이유와 방법을 알아보자

<br>

## 📄 핵심 요약

- 여러 스레드가 가변 데이터를 공유한다면 그 데이터를 읽고 쓰는 동작은 반드시 동기화하자.
- 공유되는 가변 데이터를 동기화하는 데 실패하면 응답 불가 상태에 빠지거나 안전 실패로 이어질 수 있다.

---

## 배타적 실행의 관점

많은 프로그래머가 동기화를 한 스레드가 변경하는 중이라서 상태가 일관되지 않은 순간의 객체를 다른 스레드가 보지 못하게 막는 용도로만 생각한다.

**[프로세스]**

- 한 객체가 일관된 상태를 가지고 생성되고, 이 객체에 접근하는 메서드는 그 객체에 락을 건다.
- 락을 건 메서드는 객체의 상태를 확인하고 필요하면 수정한다.
- 즉, 객체를 하나의 일관된 상태에서 다른 일관된 상태로 변화시킨다.

⇒ 동기화의 또 다른 중요한 기능은 일관성이 깨진 상태를 볼 수 없게 하는 것은 물론, 동기화된 메서드나 블록에 들어간 스레드가 이전에 같은 락에서 수행된 모든 변경 사항이 완료된 최종 결과를 보게 해준다는 것이다.

<br>

## 스레드 사이의 안정적인 통신

동기화는 스레드 사이의 안정적인 통신에 꼭 필요하다. 공유 중인 가변 데이터를 원자적(atomic)으로 읽고 쓸 수 있을지라도 동기화에 실패하면 문제를 초래할 수 있다.

### [1] 동기화하지 않은 코드

메인 스레드가 1초 후 `stopRequeted`를 true로 설정하면 `backgroundThread`가 반복문을 빠져나올 것 같이 구현한 코드이다.

```java
public class StopThread {
	private static boolean stopRequeted;
	
	public static void main(String[] args) throws InterruptedException {
		Thread backgroundThread = new Thread(() -> {
			int i = 0;
			while (!stopRequested)
				i++;
		})
		backgroundThread.start();
		
		TimeUnit.SECONDS.sleep(1);
		stopRequested = true;
	}
}
```

- 하지만 동기화하지 않으면 메인 스레드가 수정한 값을 백그라운드 스레드가 언제 보게 될지 보증할 수 없다.
- 동기화가 빠지면 가상 머신이 다음과 같은 최적화(OpenJDK 서버 VM의 `끌어올리기(hoisting)`라는 최적화 기법)를 수행할 수 있다. 결과적으로 `응답 불가(liveness failure)` 상태가 되어 프로그램의 진전이 없다.

```java
// 원래 코드
while (!stopRequested)
	i++;
				
// 최적화한 코드
if (!stopRequeted)
	while (true)
		i++;
```

<br>


### [2-1] 동기화를 적용한 코드

쓰기 메서드(requestStop)과 읽기 메서드(stopRequested) 모두를 동기화했다.

```java
public class StopThread {
	private static boolean stopRequeted;
	
	private static synchronized void requestStop() {
		stopRequested = true;
	}
	
	public static synchronized boolean stopRequested() {
		return stopRequested;
	}
	
	public static void main(String[] args) throws InterruptedException {
		Thread backgroundThread = new Thread(() -> {
			int i = 0;
			while (!stopRequested())
				i++;
		})
		backgroundThread.start();
		
		TimeUnit.SECONDS.sleep(1);
		requestStop();
	}
}
```

- 쓰기와 읽기 모두가 동기화되지 않으면 동작을 보장하지 않는다.
- 해당 코드의 두 메서드는 단순해서 동기화 없이도 원자적으로 동작한다. 이 코드에서는 동기화의 기능 중 통신 목적으로만 사용되었다.

<br>

### [2-2] 동기화를 적용한 코드 - 속도가 더 빠른 대안

`stopRequested` 필드를 `volatile`으로 선언하면 동기화를 생략해도 된다. `volatile` 한정자는 배타적 수행과는 상관없지만 항상 가장 최근에 기록된 값을 읽게 됨을 보장한다.

```java
public class StopThread {
	private static volatile boolean stopRequeted;
	
	public static void main(String[] args) throws InterruptedException {
		Thread backgroundThread = new Thread(() -> {
			int i = 0;
			while (!stopRequested())
				i++;
		})
		backgroundThread.start();
		
		TimeUnit.SECONDS.sleep(1);
		stopRequested = true;
	}
}
```

- 다만 `volatile`은 주의해서 사용해야 한다.

<br>

## Volatile, Synchronized, AtomicLong

### [1] Volatile → Synchronized

```java
private static volatile int nextSerialNumber = 0;

public static int generateSerialNumber() {
	return nextSerialNumber++;
}
```

- 실제로 증가 연산자(++)는 필드에 두 번 접근한다. 먼저 값을 읽고, 그런 다음 1을 증가한 새로운 값을 저장한다.
- 만약 이 사이에 두 번째 스레드와 접근해 값을 읽으면 첫 번째 스레드와 똑같은 값을 돌려받게 된다. 프로그램이 잘못된 결과를 계산해내는 이런 오류를 `안전 실패(safety failure)`라고 한다.
- 안전 실패를 피하기 위해서는 `volatile`을 제거한 후, `generateSerialNumber` 메서드에 `synchronized`를 사용하자.

<br>

### [2] Synchronized → AtomicLong

`AtomicLong` 이 속한 `java.util.concurrent.atomic` 패키지에는 락 없이도 스레드 안전한 프로그래밍을 지원하는 클래스들이 담겨 있다. `volatile`은 스레드 간의 통신만 지원하지만 이 패키지는 원자성(배타적 실행)까지 지원한다.

```java
private static final AtomicLong nextSerialNumber = new AtomicLong();

public static long generateSerialNumber() {
	return nextSerialNumber.getAndIncrement();
}
```

<br>
## 정리

- 가변 데이터는 단일 스레드에서만 쓰도록 하자.
- 여러 스레드가 가변 데이터를 공유한다면 그 데이터를 읽고 쓰는 동작은 반드시 동기화하자.
- 공유되는 가변 데이터를 동기화하는 데 실패하면 응답 불가 상태에 빠지거나 안전 실패로 이어질 수 있다.
- A 스레드에서 데이터 수정 후 B 스레드에 공유할 때 공유하는 부분만 동기화하여 객체(사실상 불변)를 건네는 행위를 `안전 발행(safe publication)`이라고 한다.
- 객체를 안전하게 발생하기 위해 클래스 초기화 과정에서 객체를 정적 필드, volatile 필드, final 필드, 혹은 보통의 락을 통해 접근하는 필드에 저장해도 된다.