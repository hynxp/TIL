Java에서 `Stack` 클래스는 종종 "**상속의 실패 사례**"나 "**디자인이 망가진 클래스**"로 언급된다.
뭐 때문인지 알아보자!


## Vector를 잘못 상속받았다.
Stack 클래스의 가장 큰 문제는 `Vector` 클래스를 상속받았기 때문이다. 
`Stack`의 선언을 보면 다음과 같다.

```java
public class Stack<E> extends Vector<E> {
```
`Stack`은 LIFO(Last In, First Out) 방식의 스택 자료구조를 구현하기 위해 설계되었지만, `Vector`는 일반적인 동적 배열을 나타낸다.

`Stack`이 `Vector`를 상속 받음으로써 `vector`의 모든 메서드를 사용할 수 있게 되었고, 이로 인해 원래 스택에서는 허용되지 않을 **중간에 요소를 추가하거나 삭제**하는 등의 동작이 가능해졌다.

```java
Stack<Integer> stack = new Stack<>();
stack.add(0);
stack.add(1);
stack.insertElementAt(99, 0); // 스택의 첫 번째 위치에 99 삽입

System.out.println(stack); // 출력: [99, 0, 1]
```
위처럼 `Stack` 클래스에서 비스택적인 동작을 허용하게 된 것이다.

### 왜 이런 일이 생겼을까?
`Stack`과 `Vector`는 둘 다 JDK 1.0에 추가된 클래스이다. 당시에는 객체 지향 설계 원칙이 지금처럼 엄격히 적용되지 않았고, 이후 자바가 하위 호환성을 유지해야 하는 이유로 이 설계가 변경되지 못했다고 한다.


## 상속 대신 조합이 적합했다.
상속은 클래스 간의 "is-a" 관계를 표현해야 한다. 하지만 `Stack`과 `Vector`의 관계는 "is-a"가 아니라 "uses-a" 관계에 더 가깝다.

> "Uses-a" 관계란 객체 지향 프로그래밍에서 '한 클래스가 다른 클래스를 사용'하는 관계를 말한다.

다시 말해 상속 대신 내부에서 다른 클래스의 **객체를 멤버 변수로 포함하는 방식(조합, Composition)**을 사용하는 게 적절했다.

```java
public class Stack<T> {
    private Vector<T> vector;

    public Stack() {
        this.vector = new Vector<>();
    }

    public void push(T item) {
        vector.add(item);
    }

    public T pop() {
        return vector.remove(vector.size() - 1);
    }

    public T peek() {
        return vector.get(vector.size() - 1);
    }

    public boolean isEmpty() {
        return vector.isEmpty();
    }

    public int size() {
        return vector.size();
    }
}

```
이렇게 스택의 내부 데이터 구조로 `Vector`를 사용하면서 스택 자료구조에 필요한 메서드만 제공하는 것이다.



## 해결법은? ArrayDeque를 사용하자
자바 컬렉션 프레임워크에서는 Stack 대신 `Deque` 인터페이스(특히 `ArrayDeque` 클래스)를 사용할 것을 권장한다.

`ArrayDeque`는 스택에 필요한 모든 작업을 제공하며, Stack보다 빠르다. 또 Stack은 초기 크기를 지정할 수 없었지만 `ArrayDeque`은 생성자로 초기 크기를 지정할 수 있다.

```java
public class ArrayDequeTest {
    public static void main(String[] args) {
        Deque<Integer> stack = new ArrayDeque<>();

        stack.push(10);
        stack.push(20);
        stack.push(30);

        System.out.println("Top of the stack: " + stack.peek()); // Output: 30

        // Pop elements from the stack
        System.out.println("Popped: " + stack.pop()); // Output: 30
        System.out.println("Popped: " + stack.pop()); // Output: 20

        System.out.println("Stack size: " + stack.size()); // Output: 1
    }
}
```
더 효율적인 스택 및 큐 동작을 지원하며, 불필요한 상속 문제를 피할 수 있다!





참고
[자바의 Stack 클래스는 왜 사용하지 않는 걸까?](https://velog.io/@jhl221123/%EC%9E%90%EB%B0%94%EC%9D%98-Stack%EC%9D%80-%EC%99%9C-%EC%82%AC%EC%9A%A9%ED%95%98%EC%A7%80-%EC%95%8A%EB%8A%94-%EA%B1%B8%EA%B9%8C)
[상속보다는 컴포지션을 사용하라(Feat. Stack)](https://colabear754.tistory.com/125)