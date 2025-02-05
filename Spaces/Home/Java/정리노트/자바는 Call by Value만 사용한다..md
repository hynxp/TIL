프로그래밍을 하다 보면 Call by Value / Call by Reference 개념을 배우게 된다.
~~특히 C언어에서 포인터할 때 고통받았던 기억이...~~

옛날에 자바 공부할 때는 어디서는 Call by Reference도 사용한다고 봤다..!
결론만 보자면 **자바는 Call by Value만 사용한다.**

## Call by Value

### 기본 데이터 타입
Call by Value란 값에 의한 호출을 의미한다.
메서드 호출 시 전달되는 값의 복사본이 메서드로 전달된다는 뜻이다.

```java
public class CallByValueExample {

	public static void main(String[] args) {
        int original = 5;
        modify(original);
        System.out.println(original); // 5
    }
    
    public static void modify(int number) {
        number = 10; // number의 값을 변경
    }
}
```

위 코드에서 `modify()`메서드에 `original` 변수를 보내 값을 변경한다고 해도 5로 출력될 것이다.
메서드 내의 지역변수, 매개변수는 JVM 메모리의 [[자바 코드의 메모리 영역(스택&힙)#스택(Stack)|스택]] 영역에 적재된다.

1. `main()`의 스택 프레임 내에 `original` 변수
2. `modify()`의 스택 프레임 내에 `number` 변수

이렇게 메서드에 대한 스택 프레임이 각각 생성되고, 각 메서드가 종료되면 소멸되기 때문에 변수의 원본값이 바뀌지 않는 것이다.

### 참조 데이터 타입
기본 타입이 아닌 객체 타입을 사용할 때도 Call by Value를 사용한다.
이 경우 "객체 참조(reference)의 값"이 전달된다. 즉, 객체 자체가 복사되는 것이 아니라 객체를 참조하는 주소 값을 전달한다.

기본타입에서는 변수를 메서드의 매개변수로 전달했을 때 변경되지 않은 것을 확인했다.
참조 타입인 배열을 메서드로 전달해 변경하면 바뀔까?

```java
public class CallByValueExampleWithArray {

    public static void main(String[] args) {
        int[] arr = {5, 10, 15};
        modifyArray(arr);
        System.out.println(arr[0]); // 10
    }
    
    public static void modifyArray(int[] array) {
        array[0] = 10; //arr의 0번째 값을 변경
    }
}
```

위 코드의 결과, 배열의 0번째 값이 10으로 변경된다.

main메서드가 실행하면 힙 영역에 `arr` 변수가 적재될 것이다.
그리고 `modifyArray` 메서드가 실행되면 스택 영역에 `modifyArray` 스택 프레임이 생성될 것이고, 그 안에 `array`변수는 힙 영역의 `arr`의 주소값을 참조한다.
(그래서 **참조** 데이터 타입인 것이다.)


## 그럼 Call by Reference도 사용하는거 아니야?
위에 예제 보니까 객체가 참조로 전달되는 거 같은데 Call by Reference도 사용하는거 아니야?! 할 수 있다.

하지만 자바 언어의 창시자인 제임스 고슬링이 말하길 "자바는 객체를 참조로 전달하지 않는다(not pass objects by reference). 대신 객체에 대한 **참조를 값으로 전달한다**(passes object references by value)"고 했다.

객체 자체를 전달하는 게 아니라 객체를 참조할 수 있는 **값(value)** 을 전달하는 것이다.
그 값이 힙에 저장된 위치를 가리키는 메모리 주소인 것이다.
따라서 참조는 실제 객체에 대한 alias가 아니라 실제 객체에 접근하고 조작하는 방법이라고 볼 수 있다.

```java
class Person {
    String name;
}

public class CallByValueExample2 {
    public static void main(String[] args) {
        Person p1 = new Person();
        p1.name = "Alice";

        changeReference(p1);

        System.out.println(p1.name);  // "Alice"
    }

    public static void changeReference(Person p) {
        p = new Person();  // 새로운 객체 할당
        p.name = "Bob";    // 새로운 객체의 name 변경
    }
}
```
위 코드를 보면 `p1`의 참조값이 복사되어 `p`로 전달되었다.
changeReference 메서드에서는 전달받은 p를 새로운 Person 객체로 재할당한다.
만약 자바에서 객체의 복사가 Call by Reference로 동작하는 거라면 p1 객체가 새롭게 할당한 객체로 바껴야할 것이다.
but! **p는 새로운 객체를 참조하지만 원래 `p1`에는 영향이 없다.** 이게 Call by Value인 결정적인 이유다.


토비님의 페이스북 글을 읽으면 더욱 명확하게 이해할 수 있다.
![[IMG-20241115193327599.png|500]]

`reference`와 `reference value`를 꼭 구분짓도록 하자! 

***"레퍼런스 값(reference value)를 call by value로 전달하면 호출된 메서드 내에서 호출 한 쪽에서 참조하고 있는 오브젝트 내부 값을 변경할 수 있다."***


## NullPointerException의 pointer는?
자바에는 포인터(pointer)개념이 없어서 더욱 견고한 언어라고 했는데, NullPointerException의 포인터는 뭘까?

![[String Constant Pool.excalidraw]]
자바에서 객체는 힙(heap) 메모리에 저장되며, 변수(레퍼런스)는 해당 객체의 메모리 위치를 간접적으로 가리킨다.
이런 레퍼런스는 내부적으로 포인터와 비슷한 역할을 하지만, 프로그래머가 메모리 주소를 직접 조작하거나 관리할 수 없다.

null은 레퍼런스가 어떤 객체도 가리키지 않음을 나타내는 값이다.
즉, `NullPointerException`은 **null 레퍼런스를 사용하려고 시도할 때 발생**한다.

자바에서 비록 "포인터"라는 용어를 쓰지만, 이는 메모리 주소를 직접 다루는 C/C++의 포인터와는 다른 개념이다.



참고
[자바는 Call By Value(Pass By Value) 방식으로만 동작한다](https://mangkyu.tistory.com/322)
[Call by Value, Call by Reference](https://velog.io/@ahnick/Java-Call-by-Value-Call-by-Reference)











