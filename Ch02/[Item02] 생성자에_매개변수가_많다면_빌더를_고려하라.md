# 생성자에 매개변수가 많다면 빌더를 고려하라


## ⛳️ 목표

- 생성자의 매개변수가 많거나 선택적 매개변수가 존재하는 경우 객체의 복잡성을 줄이고 가독성을 높이기 위해 빌더 패턴을 사용해야 하는 이유를 이해하자

<br>

## 📄 핵심 요약

***빌더 패턴은***

- 다양한 조합의 매개변수를 메서드 체이닝 방식으로 직관적으로 설정하게 하여 코드의 가독성과 유지보수성을 높인다.
- 불변 객체를 쉽게 생성할 수 있도록 돕는다.

***

### 1. 점층적 생성자 패턴(Telescoping Constructor Pattern)

필수 매개변수만 받는 생성자, 필수 매개변수와 선택 매개변수 1개를 받는 생성자 등 매개변수의 수에 따라 생성자를 늘려간다.

- 객체 생성 시 사용자가 설정하기 원치 않는 매개변수까지 포함하기 쉽다.
- 매개변수의 순서가 바뀌어도 컴파일 시 문제가 없기 때문에 의도와 맞지 않는 값을 가진 객체를 생성할 수 있다.

<br>

### 2. 자바빈즈 패턴(JavaBeans Pattern)

매개변수가 없는 생성자로 객체를 만든 후 세터 메서드를 호출해 원하는 매개변수의 값을 설정한다.

- 점층적 생성자 패턴에 비해 코드가 읽기 쉽다.
- 객체 하나를 만들기 위해 메서드를 여러 개 호출해야 하고 객체가 완성되기 이전에는 일관성이 떨어진다.
- 클래스를 불변으로 만들 수 없으며 스레드 안정성을 얻기 위해서는 개발자의 추가 작업이 필요하다. (e.g. freeze() 메서드 활용)

    ```java
    public class Main {
        public static void main(String[] args) {
            Person person = new Person(); // Person 객체 생성
    
            // setter 메서드를 사용하여 값 설정
            person.setName("Minji"); // 이름 설정
            person.setAge(20);       // 나이 설정
    
            // freezing 하기 전에는 정상적으로 값 접근 가능
            System.out.println("Name: " + person.getName());
            System.out.println("Age: " + person.getAge());
    
            // freezing 메서드를 호출하여 상태 고정
            person.freeze();
    
           // 이후 상태 변경 시도 (IllegalStateException 발생)
    //        person.setName("Bob"); 
        }
    }
  
    public class Person {
        private String name;
        private int age;
        private boolean isFrozen; // freezing 여부를 나타내는 플래그
    
        public Person() {
            this.isFrozen = false; // 초기 상태는 freezing되지 않음
        }
    
        public void setName(String name) {
            checkFrozen(); // freezing 여부 체크
            this.name = name;
        }
    
        public void setAge(int age) {
            checkFrozen(); // freezing 여부 체크
            this.age = age;
        }
    
        public String getName() {
            checkFrozen(); // freezing 여부 체크
            return name;
        }
    
        public int getAge() {
            checkFrozen(); // freezing 여부 체크
            return age;
        }
    
        // 상태를 고정하는 메서드
        public void freeze() {
            this.isFrozen = true; // freezing 상태로 변경
        }
    
        // freezing 여부를 체크하는 메서드
        private void checkFrozen() {
            if (isFrozen) {
                throw new IllegalStateException("This object is frozen and cannot be modified.");
            }
        }
    }
    ```

<br>

### 3. 빌더 패턴(Builder Pattern)

필수 매개변수만으로 생성자(혹은 정적 팩터리)를 호출해 빌더 객체를 얻으며, 원하는 선택 매개변수를 설정한다.

- 파이썬과 스칼라에 있는 `명명된 선택적 매개변수(named optional parameters)`를 흉내낸 것이다. 함수의 호출 시 유연하게 매개변수를 호출할 수 있다.

    ```python
    # 파이썬 예제 코드
    # 매개변수에 기본 값을 설정하여 함수 정의
    def create_user(name, age=20, city="Unknown"):
        return {
            "name": name,
            "age": age,
            "city": city
        }
    
    # 기본값 사용
    user1 = create_user("Minji")
    print(user1)  # {'name': 'Minji', 'age': 20, 'city': 'Unknown'}
    
    # 선택적 매개변수 설정
    user2 = create_user("Hanni", city="Melbourne")
    print(user2)  # {'name': 'Hanni', 'age': 20, 'city': 'Melbourne'}
    
    ```

- 빌더의 세터 메서드들은 빌더 자신을 반환하기 때문에 연쇄적으로 호출할 수 있다. 이런 방식을 `Fluent API` 혹은 `Method Chanining`이라 한다.

<br>

**[장점]**

1️⃣ 불변 객체를 생성할 수 있어 안전하며 코드를 읽고 쓰기 쉽다.

<br>

2️⃣ 계층적으로 설계된 클래스에 사용하면 형변환에 신경 쓰지 않고 빌더를 사용할 수 있다.

***Animal 클래스: 상위 클래스***

```java
public abstract class Animal {
	public enum Trait { SPEED, STRENGTH, AGILITY }
	
	final EnumSet<Trait> traits;
	
	// Animal의 추상 빌더 클래스
	abstract static class Builder<T extends Builder<T>> {
		EnumSet<Trait> tratis = EnumSet.noneOf(Trait.class);
		
		public T addTrait(Trait trait) {
			traits.add(Objects.requireNonNull(trait);
			return self();
		}
		
		// Animal 인스턴스를 생성하는 메서드 (하위 클래스에서 구현)
		abstract Animal build();
		
		// "self"를 반환하는 메서드 (하위 클래스에서 this를 반환해야 함)
		protected abstract T self();
		
		// Animal 생성자 > 빌더로부터 필드 복사
		Animal(Builder<?> builder) {
			traits = builder.traits.clone();
		}
	}	
}
```

<br>

***Dog 클래스: 하위 클래스***

```java
public class Dog extends Animal {
	private final String breed;
	
	public static class Builder extends Dog.Builder<Builder> {
		private String breed;
		
		public Builder(String breed) {
			this.breed = Objects.requireNonNull(breed);
		}
		
		// 공변 변환 타입 
		@Override
		public Dog build() {
			return new Dog(this);
		}
		
		// self type 관용구
		@Oberride
		protected Builder self() {
			return this;
		}
	}
}
```

- self() 메서드를 통해 하위 클래스의 Builder를 반환하도록 설정할 수 있다.

<br>

🏷️ **NOTE**

> `공변 반환 타입(convariant return typing)`이란 상위 클래스의 반환 타입을 하위 클래스에서 더 구체적으로 바꿔서 사용할 수 있게 해주는 기능이다.
>

<br>

🏷️ **NOTE**

> `시뮬레이트한 셀프 타입(simulated self-type) 관용구`란 쉽게 말해 자바에는 존재하지 않는 self type을 흉내내어 사용하는 것을 말한다.
> - 상위 클래스의 메서드가 자신의 하위 클래스 인스턴스를 반환하도록 하고 싶을 때 사용한다.
> - 주로 빌더 패턴 구현에 사용되며 상위 클래스 내부에 빌더를 정의하고 하위 클래스에서 빌더를 구현할 때 오버라이딩하여 자신을 반환한다.
>

```kotlin
// kotlin에서의 셀프 타입 예제 코드
open class Animal<T : Animal<T>> {
    fun withName(name: String): T {
        // 이름 설정
        return this as T
    }
}

class Dog : Animal<Dog>() {
    fun withBreed(breed: String): Dog {
        // 품종 설정
        return this
    }
}
```

<br>

3️⃣ 가변인수(varargs) 매개변수를 여러 개 사용할 수 있다.

***클라이언트 코드***

```java
Dog dog = new Dog.Builder()
            .breed("Bulldog")
            .addTrait(Animal.Trait.STRENGTH)
            .addTrait(Animal.Trait.SPEED)
            .build();
```

- `addTrait()` 메서드를 여러 번 호출하여 하나의 동물 객체에 여러 특성을 쉽게 추가할 수 있기 때문에 `가변 인수`처럼 유연하게 사용할 수 있다.

<br>

**[단점]**

1️⃣ 객체를 만드려면 빌더부터 생성해야 하기 때문에 성능에 민감한 상황에서 문제가 될 수 있다

2️⃣ 매개변수가 4개 이상일 때 효과가 두드러진다.