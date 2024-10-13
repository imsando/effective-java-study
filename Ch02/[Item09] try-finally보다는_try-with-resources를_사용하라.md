# try-finally보다는 try-with-resources를 사용하라

## ⛳️ 목표

AutoCloseable 인터페이스와 try-with-resources 구문을 이해하자

<br>

## 📄 핵심 요약

***AutoCloseable 인터페이스와 try-with-resources 구문을 사용하면***

- 자원을 명확하고 안전하게 해제할 수 있다.
- 여러 자원을 사용해도 코드의 가독성을 유지할 수 있다.
- 예외 누락을 방지할 수 있다.

---

## try-finally

전통적으로 자원 해제를 보장하는 수단으로 `try-finally`가 사용됐다.

```java
// 하나의 자원 사용 후 자원 해제
BufferedReader br = new BufferedReader(new FileReader(path));
try {
	return br.readLine();
} finally {
	br.close();
}

// 두 개의 자원 사용 후 자원 해제
InputStream in = new FileInputStream(source);
try {
	OutputStream out = new FileOutputStream(destination);
	try {
		byte[] bytes = new byte[BUFFER_SIZE];
		int n;
		while ((n = in.read(bytes)) >= 0)
			out.write(bytes, 0, n);
	} finally {
		out.close();
	}
} finally {
	in.close();
}

```

**[문제점]**

- 개발자가 별도로 `close()` 메서드를 작성하다 보면 자원 관리를 놓칠 수 있다.
- 하나의 자원 사용 코드 중 `br.readLine()` 메서드에서 예외가 발생하고 `close()` 메서드도 실행되지 못했을 때 첫 번째 예외에 관한 정보는 남지 않고 두 번째 예외 정보만 남아 디버깅이 어렵게 된다.
- 여러 자원을 사용하면 `close()` 메서드를 여러 번 작성하게 되어 코드가 지저분해진다.

<br>

## try-with-resources (Java 7 도입)

원하는 객체에 `AutoCloseable` 인터페이스를 구현하면 자원 관리를 쉽게 할 수 있는 기능이다.

- 자원을 명확하고 안전하게 해제할 수 있다.
- 여러 자원을 사용해도 코드의 가독성을 유지할 수 있다.
- 예외 누락을 방지할 수 있다.

<br>

### AutoCloesable

자바에서 자원을 자동으로 해제하기 위한 인터페이스이다. 내부에 `close()` 메서드가 구현되어있어 `try-with-resources` 구문이 종료될 때 자동으로 호출된다. 그러므로 개발자가 직접 자원 해제를 위한 코드를 작성하지 않아도 된다.

<br>

### 예제 코드

```java
// 한 개의 자원
try(BufferedReader br = new BufferedReader(new FileReader(path));) {
    return br.readLine();
} catch {
// 예외 처리 코드
}

// 두 개의 자원
try(
    InputStream in = new FileInputStream(source),
    OutputStream out = new FileOutputStream(destination)
) {
    // 처리해야 하는 작업 코드
} catch {
    // 예외 처리 코드
}
```