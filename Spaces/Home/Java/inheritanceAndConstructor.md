# 상속 시 조심해야 할 생성자

```java
public class Parent {
    public Parent() {
        System.out.println("Parent Contructor");
    }

    public void printName() {
        System.out.println("Parent printName()");
    }
}

public class Child extends Parent{
    public Child() {
        System.out.println("Child Constructor");
    }
}
```

```java
public class InheritancePrint {
    public static void main(String[] args) {
        Child child = new Child();
        child.printName();
    }
}
```

**실행결과**

```
Parent Contructor
Child Constructor
Parent printName()
```

Parent 클래스의 메소드를 호출하지도 않았는데, 자식 클래스의 생성자가 호출되면 자동으로 부모 클래스의 기본 생성자(매개 변수가 없는)가 호출된다.

만약 Parent의 기본 생성자를 주석처리한다면 실행결과는 다음과 같다.

```java
public class Parent {
//    public Parent() {
//        System.out.println("Parent Contructor");
//    }

    public void printName() {
        System.out.println("Parent printName()");
    }
}
```

```
Child Constructor
Parent printName()
```

별 문제 없다고 생각할 수 있다.

하지만 Parent 클래스에 매개변수가 있는 생성자가 있다면?

```java
public class Parent {
    **public Parent(String name) {
        System.out.println("Parent Contructor");
    }**

    public void printName() {
        System.out.println("Parent printName()");
    }
}
public class Child extends Parent{
    public Child() {
        System.out.println("Child Constructor");
    }
}
```

```java
public class InheritancePrint {
    public static void main(String[] args) {
        Child child = new Child();
        child.printName();
    }
}
```

컴파일 에러가 발생한다.

Child 클래스의 모든 생성자가 실행될 때 Parent()라는 기본 생성자를 찾는다.

→ 자동으로 `super();` 라는 문장이 들어가기 때문

매개변수가 있는 생성자가 있으면 기본 생성자가 자동으로 생성되지 않기 때문에

부모 클래스의 기본 생성자를 찾지 못한다고 에러가 발생하는 것이다.

이 경우에 해결방법은 2가지다.

1. 부모 클래스에 기본 생성자를 만든다.
2. 자식 클래스에서 부모 클래스의 생성자를 명시적으로 지정하는 `super()`를 사용한다.
    
    ```java
    public class Child extends Parent{
        public Child() {
    				super("Child");
            System.out.println("Child Constructor");
        }
    }
    ```
    

그런데, Parent 클래스에 매개변수가 하나인 생성자가 더 있다면?

```java
public class Parent {
    public Parent(String name) {
        System.out.println("Parent Constructor");
    }
    
    **public Parent(InheritancePrint obj) {
        System.out.println("Parent(InheritancePrint) Constructor");
    }**

    public void printName() {
        System.out.println("Parent printName()");
    }
}
```

```java
public class Child extends Parent{
    public Child() {
				super(null);
        System.out.println("Child Constructor");
    }
}
```

Child 클래스의 생성자에서 super에 null을 넘겨주면?

“reference to Parent is ambiguous” 클래스로의 참조가 모호하다라는 에러가 발생한다.

`super()`를 사용하여 생성자를 호출할 때는 모호하게 null을 넘기지 말고 호출하고자 하는 생성자의 기본 타입을 넘겨주는 것이 좋다.


**결론**

자바는 부모의 매개 변수가 없는 기본 생성자를 찾는 것이 기본이므로 부모 클래스에 매개 변수가 있는 생성자만 있을 경우에는 `super()`를 이용해서 부모 생성자를 꼭 호출해줘야 한다.