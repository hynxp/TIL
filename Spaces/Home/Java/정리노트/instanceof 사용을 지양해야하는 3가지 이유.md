## instanceof란?
자바에서 객체가 특정 클래스의 인스턴스인지 확인하는 데 사용하는 연산자이다.

```java
abstract class Animal {
}

class Dog extends Animal {
}

class Cat extends Animal {
}
```

위와 같은 상속 관계일 때 

```java
public class AnimalGame {

	public void makeSound(Animal animal) {
		if (animal1 instanceof Dog) {
		//생략
		}
		
		if (animal2 instanceof Cat) {
		//생략
		}
	}
}
```
이렇게 각 객체의 실제 타입을 확인할 때 주로 `instanceof`를 사용할 수 있다.


## instanceof의 문제점 3가지
위의 예시를 보면 객체가 상위 클래스나 인터페이스의 인스턴스인지도 검사할 수 있어 꽤 안전해 보이는데 어떤 문제점이 있을까?

### 1. 캡슐화 위반
첫번째로 객체지향 프로그래밍(OOP)의 중요한 원칙 중 하나인 캡슐화를 위반한다.
캡슐화란 객체의 내부 상태를 외부에서 알지 못하게 숨기고, 객체와의 상호작용은 정해진 인터페이스를 통해 이루어지도록 하는 것이다.

```java
public class AnimalGame {

	public void makeSound(Animal animal) {
		if (animal1 instanceof Dog) {
		//로직
		}
		
		if (animal2 instanceof Cat) {
		//로직
		}
	}
}
```
위 코드는 `Animal` 객체로 들어온 인스턴스가 `Dog`일 때, `Cat`일 때를 `instanceof`로 구분하여 로직을 처리한다.

AnimalGame 코드를 작성하는 개발자라고 생각해보자.
그럼 이 Animal 객체를 사용하려면 `Dog`가 Animal의 하위 계층이구나, `Cat`도 하위 계층이구나. 
이 구조를 직접적으로 알아야 한다.
`instanceof`를 사용함으로써 외부 코드가 객체의 구체적인 타입을 알도록 강제한 것이다.

**instanceof를 쓰지 않고 자식 객체를 어떻게 활용할까?**
```java
public class AnimalGame {

	public void makeSound(Animal animal) {
		animal.move();
	}
}
```

이렇게 추상화된 상위 계층인 Animal 객체에게 "너 밑에 애들한테 move()하라고 해~" 메세지만 전달하면 된다.

### 2. OCP(Open Closed Principle) 위반
OCP는 "확장에는 열려 있고, 수정에는 닫혀 있어야 한다"는 설계 원칙이다. 그러나 `instanceof`를 사용하여 타입별로 동작을 정의하면 새로운 타입이 추가될 때마다 기존 코드를 수정해야 한다.

```java
if (animal instanceof Dog) {
    //Dog의 로직
} else if (animal instanceof Cat) {
    //Cat의 로직
} else if (animal instanceof Bird) { // 새로운 타입 추가
    //Bird의 로직
}
```

Animal의 새로운 하위 객체가 추가될 때마다 기존 코드에 분기문을 추가해야 한다.
확장성이 굉장히 구려진다.


### 3. SRP(Single Responsibility Principle) 위반
SRP는 "클래스는 하나의 책임만 가져야 한다"는 원칙이다. 그러나 `instanceof`를 사용하여 한 곳에서 여러 타입에 대한 처리를 구현하면 해당 클래스나 메서드가 여러 역할을 동시에 담당하게 되어 SRP를 위반하게 된다.

이 부분은 1, 2번의 이유와 비슷하다.
```java
if (animal instanceof Dog) {
    //Dog의 로직
} else if (animal instanceof Cat) {
    //Cat의 로직
} else if (animal instanceof Bird) { // 새로운 타입 추가
    //Bird의 로직
}
```

위에서 봤듯이 `instanceof`를 사용하게 됨으로써 캡슐화가 위반되었고, 계속 분기 코드를 추가하게 되었다.
이 메서드는 다양한 타입의 동물을 처리하고 있으므로 하나의 책임이 아니라 Animal의 각 하위객체에 대한 역할을 다 수행하고 있는 것이다.


## instanceof 대신 무엇을 사용해야 할까?
단점이 너무 커보이는데 그럼 어쩌라는거냐!
객체 지향의 핵심인 **다형성**을 이용하면 된다.


```java
abstract class Animal {
	public boolean isDog();
}

class Dog extends Animal {
	@Override
	public boolean isDog() {
		return true;
	}
}

class Cat extends Animal {
	@Override
	public boolean isDog() {
		return false;
	}
}
```

Animal 클래스에 다형성을 이용하기 위해 각 객체가 Dog인지 체크하는 isDog() 메서드를 사용하면 된다.

```java
public class AnimalGame {

	public void makeSound(Animal animal) {
		if (animal.isDog()) {
			// Dog의 로직
		}
	}
}
```

AnimalGame은 Animal가 Dog인지 몰라도 되고, Animal에게 메시지를 보내서 알아서 처리하게 시킨다.

아니면 1번의 예시처럼 사용하면 더욱 간단하다.
```java
public class AnimalGame {

	public void makeSound(Animal animal) {
		animal.move();
	}
}
```