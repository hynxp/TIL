객체지향 프로그래밍을 배우다 보면 클래스, 객체, 인스턴스라는 용어를 자주 접하게 된다. 이 세 가지는 서로 밀접하게 관련이 있지만, 각각의 개념은 명확히 구분된다. 이를 이해하기 위해 각각의 정의와 예를 통해 살펴보자.


## 클래스(Class)
클래스는 객체를 생성하기 위한 설계도 또는 청사진이다. 변수와 메서드로 구성되며, 객체가 어떤 속성과 동작을 가질지를 정의한다. 하지만 클래스 자체로는 실행 가능한 것이 아니라, 객체를 생성하기 위한 틀 역할을 한다.

### 예제
```java
// 클래스 정의
class Car {
    String brand; // 속성
    int speed; // 속성

    void drive() { // 메서드
        System.out.println(brand + " is driving at " + speed + " km/h");
    }
}
```
위 코드에서 `Car`는 클래스이다. 이 클래스는 자동차의 브랜드와 속도를 나타내는 속성, 그리고 `drive`라는 동작을 정의하고 있다.


## 객체(Object)
객체는 클래스를 기반으로 만들어진 실체이다. 클래스에서 정의한 속성과 메서드를 가지며, 프로그램 내에서 메모리에 실제로 할당된 데이터와 동작을 가진다. 즉, 객체는 클래스의 구체적인 구현물이다.

### 예제
```java
// 객체 생성
Car myCar = new Car();
myCar.brand = "Toyota";
myCar.speed = 120;

// 메서드 호출
myCar.drive();

```
여기서 `myCar`는 `Car` 클래스의 객체이다. 클래스의 설계도를 바탕으로 메모리에 생성되었으며, `brand`와 `speed` 속성에 값을 할당한 후, `drive` 메서드를 실행할 수 있다.


## 인스턴스(Instance)
인스턴스는 객체와 밀접하게 연관된 개념이다. 객체는 클래스의 실체이고, 인스턴스는 "특정 클래스에서 생성된 객체"라는 점에서 조금 더 좁은 의미를 가진다. 쉽게 말해, 객체가 어떤 클래스의 인스턴스인지를 강조하고자 할 때 사용하는 용어이다.

### 예제
```java
Car anotherCar = new Car();
```

위 코드에서 `anotherCar`는 `Car` 클래스의 인스턴스라고 말할 수 있다. 또한, `myCar` 역시 `Car` 클래스의 인스턴스이다. 모든 인스턴스는 객체이지만, 모든 객체가 특정 클래스의 인스턴스인 것은 아니다(예: 객체 지향 언어 외부의 일반적인 객체 개념).


## 클래스, 객체, 인스턴스의 관계

| 개념   | 정의              | 역할                            |
| ---- | --------------- | ----------------------------- |
| 클래스  | 객체를 생성하기 위한 설계도 | 속성과 메서드 정의                    |
| 객체   | 클래스에 의해 생성된 실체  | 프로그램에서 실제 동작 가능               |
| 인스턴스 | 특정 클래스에서 생성된 객체 | 객체가 어떤 클래스에서 만들어졌는지를 강조할 때 사용 |


어떤 역할과 기능을 갖고있는 추상적으로 존재하는 개념 `객체`를 정의했다.
이거를 자바 코드로 나타내기 위해 쓰는 게 `클래스`라는 도구고
그 클래스를 new()를 붙여서 메모리에 실체화된 거를 `인스턴스`라고 한다.

### 객체 == 인스턴스?
메모리의 실체화된 클래스를 인스턴스라고 하는 거기 때문에 객체랑 인스턴스랑 같다고 표현하진 않는다.