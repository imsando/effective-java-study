# 생성자 대신 정적 팩터리 메서드를 고려하라

---

## ⛳️ 목표

정적 팩터리 메서드의 사용을 고려할 수 있는 장점과 단점을 알아보자

<br>

## 📄 핵심 요약

***정적 팩터리 메서드는***

- 매개변수가 많을 경우, 생성되는 객체의 의미를 잘 전달할 수 있다.
- 다양한 구현체를 반환할 수 있다.
- 필요한 경우, 동일한 객체를 재사용 할 수 있어 메모리 사용을 최적화할 수 있다.
- 서비스 제공자 프레임워크에서 구현체의 클래스가 동적으로 결정될 수 있게 해준다.

***

## 1. 장점

### 1.1 이름을 가질 수 있다.

   정적 팩터리 메서드는 이름만 잘 지으면 반환될 객체의 특성을 쉽게 묘사할 수 있다.  

<br>

### 1-2. 호출될 때마다 인스턴스를 새로 생성하지 않아도 된다.
- `플라이웨이트(Flyweight) 패턴`과 유사하게 객체를 반복적으로 생성하지 않고 재사용하므로서 자원을 아낄 수 있다.
- 인스턴스를 통제하여 한 번 생성된 객체를 계속 반환하게 하거나 매번 새로운 인스턴스를 반환할 수 있게 할 수 있다.

  ```java
  // 이미 만들어진 상수를 재사용하므로 새로운 객체 생성 방지
  public static Boolean valueOf(boolean b) {
      return b ? Boolean.TRUE : Boolean.FALSE;
  }
    
  // 만들어져있는 상수
  public static final Boolean TRUE = new Boolean(true);
  public static final Boolean FALSE = new Boolean(false);
  ```

<br>

### 1-3. 반환 타입의 하위 타입 객체를 반환할 수 있는 능력이 있다.

`java.util.Collections`는 `인스턴스화 불가 클래스`로 정적 팩터리 메서드를 통해 컬렉션 객체를 관리하고 반환한다.

```java
// 반환 타입은 List 인터페이스이지만 반환 객체는 하위 타입인 UnmodifableList 객체
public static <T> List<T> unmodifiableList(List<? extends T> list) {
    return new UnmodifiableList<>(list);
}
```

- 내부적으로 정적 팩터리 메서드를 사용해 다양한 하위 타입의 객체를 반환할 수 있다.
- 외부에서는 인터페이스 타입(e.g. List, Set, Map)으로 사용하게 할 수 있다.
- 즉, **유연하고 확장 가능하게 설계될 수 있다.**

<br>

🏷️ **NOTE**
> * 자바 8 이전: 인터페이스에 정적 메서드 생성 불가  
> * 자바 8 이후: 인터페이스에서 정적 메서드 허용  
> * 자바 9 이후: private 정적 메서드 허용  
즉, 인터페이스의 정적 메서드 기능은 확장되었지만 public 외의 정적 필드, 정적 멤버 클래스를 관리하거나, 패키지 수준의 접근 제한이 필요한 경우에는 여전히 별도의 클래스가 필요하다.

<br>

### 1-4. 입력 매개변수에 따라 매번 다른 클래스의 객체를 반환할 수 있다.

```java
public abstract class EnumSet { 
    ...
    
    // 원소의 수에 따라 다른 인스턴스 반환
    public static noneOf(...) {
        if (enumElementSize <= 64)
            return new RegularEnumSet<>(...);  
        else
            return new JumboEnumSet<>(...);
    }
}
```

<br>

### 1-5. 정적 팩터리 메서드를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다.
- 서비스 제공자 인터페이스(SPI)를 사용하여 다양한 구현체를 쉽게 관리할 수 있으며, 각 구현체의 클래스 이름을 코드에 명시하지 않아도 된다.

```java
/**
    SPI를 사용하지 않았을 때
*/
public class JDBC Example {
    public static void main(String[] args) {
        String driverClassName = "com.mysql.cj.jdbc.Driver"; // 드라이버 클래스 이름
        try {
            // 리플렉션을 사용하여 클래스 로드
            Class<?> driverClass = Class.forName(driverClassName);
            
            // 드라이버 클래스의 인스턴스 생성
            Driver driver = (Driver) driverClass.getDeclaredConstructor().newInstance();
            
            // 드라이버를 등록하는 코드 (JDBC 연결 등)
            DriverManager.registerDriver(driver);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}

/**
    SPI 사용했을 때 - DriverManager를 통해 드라이버 로드 및 등록
*/
public class JDBC Example {
    public static void main(String[] args) {
        try {
        // 드라이버 클래스가 자동으로 등록됨
        Connection connection = DriverManager.getConnection(url, user, password);
        System.out.println("Database connected successfully!");

        ...

        connection.close();
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
}
```

- 즉, **프로그램의 유연성을 높이고 코드의 유지보수성을 개선하는 데 기여한다.**

<br>

🏷️ **NOTE**
> `브리지 패턴(Bridge Pattern)`은 구현과 추상화를 분리하여 코드의 유연성과 확장성을 높이는 데 유용한 디자인 패턴이다.


***

## 2. 단점

### 2-1. 상속을 하려면 public이나 protected 생성자가 필요하니 정적 팩터리 메서드만 제공하면 하위 클래스를 만들 수 없다.

대신 상속보다 컴포지션을 사용하도록 유도할 수 있으며 불변타입으로 만들기 위해 이 제약을 지켜야 한다는 점에서 오히려 장점이 될 수 있다.  

<br>

### 2-2. 정적 팩터리 메서드는 프로그래머가 찾기 어렵다.

API 설명이 없기 때문에 문서를 잘 작성하고 널리 알려진 규약을 따라 메서드 이름을 지어야 한다.  

<br>

| 명명                                                                | 설명                                                                                                                 |
   |-------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------|
| from()                                                            | 매개 변수를 하나 받아 해당 타입의 인스턴스를 반환하는 형변환 메서드 `e.g.` Date d = Date.from(instant);                                         |
| of()                                                              | 여러 매개변수를 받아 적합한 타입의 인스턴스를 반환하는 집계 메서드 `e.g.` Set<Rank> faceCards = EnumSet.of(JACK, QUEEN, KING);                  |
| valueOf()                                                         | from과 of의 더 자세한 버전 `e.g.` BigInteger prime = BigInteger.valueOf(Integer.MAX_VALUE);                                |
| instance()<br>getInstance()                                       | (매개 변수를 받는다면) 매개변수로 명시한 인스턴스를 반환하지만 같은 인스터임이 보장되지 않는다. `e.g.` StackWalker luke = StackWalker.getInstance(options); |
| create()<br>newInstance()                                                     | 매번 새로운 인스턴스를 생성해 반환함을 보장한다. `e.g.` Object newArray = Array.newInstance(classObject, arrayLen);                     |
| getType()                                                         | 생성할 클래스가 아닌 다른 클래스에 팩터리 메서드를 정의할 때 쓴다. `e.g.` FileStore fs = Files.getFileStore(path);                             |
| newType()                                                         | 생성할 클래스가 아닌 다른 클래스에 팩터리 메서드를 정의할 때 쓴다. `e.g.` BufferedReader br = Files.newBufferedReader(path;                    |
| type()                                                            | getType과 newType의 간결한 버전                                                                                           `e.g.` List<Complaint> Litany = Collections.list(legacyLitetany); |