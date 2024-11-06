# 중첩 클래스보다는 정적 멤버 클래스를 사용하라

## ⛳️ 목표

- 중첩 클래스의 종류와 각 종류의 적절한 사용법을 이해하고 가능한 경우 정적 멤버 클래스를 사용해야하는 이유에 대해 알아보자

<br>

## 📄 핵심 요약

***중첩 클래스란?***

- 중첩 클래스는 다른 클래스 안에 정의된 클래스를 말하며 자신을 감싼 바깥 클래스에서만 사용되는 경우에 적합하다
- 중첩 클래스는 바깥 클래스와의 밀접한 관계를 가지며 바깥 클래스의 멤버에 접근할 수 있다

***중첩 클래스의 종류***

> 1. **정적 멤버 클래스**
> 2. **비정적 멤버 클래스**
> 3. **익명 클래스**
> 4. **지역 클래스**

👉 이 중 첫 번째인 정적 멤버 클래스는 내부 클래스에 포함되지 않으며 나머지 세 가지는 모두 **내부 클래스**로 분류된다

### 1. 정적 멤버 클래스

- **특징**: 클래스가 다른 클래스의 내부에 정의되며 바깥 클래스의 `private` 멤버에 접근할 수 있다
- **장점**: 바깥 클래스와 밀접한 관계가 있으나 독립적으로 존재할 수 있는 경우 유용하며 **메모리 누수를 방지**하고 **성능도 더 효율적**이다

예시 코드
```java
public class Calculator {
    public static class Operation {
        public static final Operation PLUS = new Operation("+");
        public static final Operation MINUS = new Operation("-");
        
        private final String symbol;
        
        private Operation(String symbol) {
            this.symbol = symbol;
        }
        
        public String getSymbol() {
            return symbol;
        }
    }
}
```

### 2. 비정적 멤버 클래스

- **특징**: 
> 1. 정적 멤버 클래스와 다르게 바깥 클래스의 인스턴스와 암묵적으로 연결된다
> 2. 비정적 멤버 클래스의 인스턴스는 바깥 인스턴스와 함께 사용되며 **외부 참조가 생긴다**

- **단점**: 바깥 인스턴스와 연결되어 있어 **메모리 공간을 더 사용**하며 **생성 시 추가 시간**이 필요하다

예시 코드
```java
public class MySet<E> extends AbstractSet<E> {
    @Override
    public Iterator<E> iterator() {
        return new MyIterator();
    }
    
    private class MyIterator implements Iterator<E> {
        // 바깥 인스턴스(MySet)에 접근 가능
    }
}
```

### 3. 익명 클래스

- **특징**: 
> 1. 이름이 없고 선언과 동시에 인스턴스가 생성되는 중첩 클래스이다 
> 2. 주로 작은 함수 객체나 처리 객체로 사용되며 **짧은 코드에 유리**하다
- **단점**: 사용 범위가 제한적이고 가독성을 위해 **짧은 코드에만 적합**하다
- **사용 예**: 람다를 사용하기 전에는 작은 함수 객체나 처리 객체로 사용되었다

예시 코드
```java
Runnable runnable = new Runnable() {
    @Override
    public void run() {
        System.out.println("익명 클래스 실행");
    }
};
```

### 4. 지역 클래스

- **특징**: 
> 1. 메서드 내에 정의되며 지역변수와 같은 유효 범위를 가진다 
> 2. 비정적 문맥에서만 사용 가능하고 코드 내에서 특정 작업에 대해 임시적으로 필요한 경우 적합하다
- **단점**: 짧고 단순한 작업 외에는 사용이 불편하다
- **사용 예**: 특정 메서드에서만 사용하는 **작은 작업**을 정의할 때 사용한다

예시 코드
```java
void someMethod() {
    class LocalClass {
        void printMessage() {
            System.out.println("지역 클래스 사용");
        }
    }
    LocalClass local = new LocalClass();
    local.printMessage();
}
```

### 5. 중첩 클래스 사용 시 주의사항

- 멤버 클래스가 바깥 클래스의 인스턴스에 접근할 필요가 없다면 **무조건 정적(static)을 붙여서 정적 멤버 클래스로 만들자**
- `static`을 붙이지 않으면 바깥 클래스에 대한 **숨은 외부 참조가 생기므로 메모리와 성능에 영향을 미칠 수 있다**
- 외부 참조가 눈에 보이지 않아 **메모리 누수**가 발생하기 쉽다

> **핵심 정리**  
> 중첩 클래스에는 네 가지가 있으며 각각의 쓰임새가 다르다
> 메서드 밖에서도 사용해야 하거나 메서드 안에 정의하기에는 너무 길다면 **멤버 클래스로** 만든다
> 멤버 클래스가 바깥 인스턴스를 참조하지 않는다면 **정적 멤버 클래스로** 만드는 것이 좋다