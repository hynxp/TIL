
## 싱글톤 패턴은 thread safe할까?
싱글톤 패턴은 클래스의 인스턴스를 하나만 생성하고, 어디서든 이를 접근할 수 있도록 하는 디자인 패턴이다. 그러나 기본적인 싱글톤 구현 방식은 멀티 쓰레드 환경에서 안전하지 않다.

### 기본 싱글톤 구현 (Lazy Initialization)
```java
class Singleton {
    // 싱글톤 클래스 객체를 담을 인스턴스 변수
    private static Singleton instance = null;

    // 생성자를 private로 선언 (외부에서 new 사용 X)
    private Singleton() {}
	
    // 외부에서 정적 메서드를 호출하면 그제서야 초기화 진행 (lazy)
    public static Singleton getInstance() {
        if (instance == null) {
            instance = new Singleton(); // 오직 1개의 객체만 생성
        }
        return instance;
    }
}
```
위 방식은 **Lazy Initialization**(필요할 때 생성) 방식으로 싱글톤을 구현한 것이다. 하지만 멀티 쓰레드 환경에서는 다음과 같은 문제가 발생한다.

먼저 스레드 A, 스레드 B 가 존재한다고 가정한다.
![[Pasted image 20250201103425.png|400]]
1. 스레드 A와 스레드 B가 동시에 `getInstance()`를 호출한다.
2. 스레드 A가 `if (instance == null)` 조건을 확인하고, 인스턴스를 생성하려는 순간 스레드 B도 같은 조건을 평가한다.
3. 두 스레드가 `new Singleton()`을 각각 실행하여 **두 개의 인스턴스가 생성**될 가능성이 있다.

즉, 위 코드는 싱글톤 패턴을 깨뜨릴 수 있으며 **Thread Safe하지 않다.** 이를 해결하기 위해 여러 가지 방법이 존재한다.


## Thread safe initialization (동기화)
그럼 동기화를 위해서 `synchronized`를 붙이면 되지 않을까?
```java
class Singleton {
    private static Singleton instance = null;

    private Singleton() {}

	// synchronized로 락 걸어서 동기화
    public static synchronized Singleton getInstance() {
        if (instance == null) {
            instance = new Singleton();
        }
        return instance;
    }
}
```
### 개선점
- `synchronized` 키워드를 사용하여 동기화 문제 해결 → 멀티스레드 환경에서도 안전함.

### 문제점
- `getInstance()`가 호출될 때마다 `synchronized`로 인해 **성능 저하**가 발생할 수 있다.
- 스레드 수가 많아질수록 **병목 현상**이 발생하여 성능이 떨어진다.


## Double-Checked Locking (DCL, 이중 체크 락킹)
매번 `synchronized`를 실행하는 것이 성능 저하를 유발하므로, 최초 **초기화 시에만 동기화**를 수행하도록 최적화할 수 있다.
```java
class Singleton {
    private static volatile Singleton instance = null; // volatile 키워드 적용

    private Singleton() {}

    public static Singleton getInstance() {
        if (instance == null) {
        	// 메서드에 동기화 거는게 아닌, Singleton 클래스 자체를 동기화 걸어버림
            synchronized (Singleton.class) { 
                if(instance == null) { 
                    instance = new Singleton(); // 최초 초기화만 동기화 작업이 일어나서 리소스 낭비를 최소화
                }
            }
        }
        return instance; // 최초 초기화가 되면 앞으로 생성된 인스턴스만 반환
    }
}
```

### 설명
왜 if문이 2번이나 필요한걸까?
#### (1) 첫 번째 `if (instance == null)`
```java
if (instance == null) {
    synchronized (Singleton.class) {
```
첫 번째 `if` 문은 **이미 인스턴스가 생성되어 있으면 동기화 블록을 실행하지 않도록 최적화하는 역할**을 한다.
`instance`가 이미 생성된 경우 불필요하게 `synchronized` 블록에 들어가지 않고 바로 반환한다.
즉, **성능 최적화**를 위한 것이다.

#### (2) 두 번째 `if (instance == null)`
```java
synchronized (Singleton.class) {
    if (instance == null) { 
```
`synchronized` 블록에 들어왔을 때, **다른 스레드가 이미 인스턴스를 생성했을 수도 있다.**
따라서 두 번째 `if (instance == null)`을 통해 **인스턴스가 정말 생성되지 않았는지 다시 한 번 확인**해야 한다.
그래서 **이중 체크(Double-Checked Locking)** 기법이라고 부르는 것이다!

#### 시나리오
1. **스레드 A**가 `getInstance()`를 호출 → `if (instance == null)` 확인 후 `synchronized (Singleton.class)` 블록에 진입
2. **스레드 B**도 `getInstance()`를 호출 → `if (instance == null)` 확인 후 `synchronized` 블록에 진입하려 하지만, **스레드 A가 락을 점유하고 있어서 대기**
3. **스레드 A가 `new Singleton()`을 실행하고 `instance`가 생성됨**
4. **스레드 A가 락을 해제한 후, 스레드 B가 동기화 블록에 진입**
5. **스레드 B는 두 번째 `if (instance == null)`을 확인** → 이미 `instance`가 생성되었기 때문에, 새롭게 `new Singleton()`을 실행하지 않음

즉, **이중 체크 덕분에 단 하나의 인스턴스만 생성되도록 보장할 수 있다.** 후후

### 개선점
-  인스턴스 필드에 `volatile`을 사용하여 인스턴스가 완전히 초기화되기 전에 다른 스레드가 접근하는 문제 방지
- `synchronized` 블록 안에서 한 번 더 `INSTANCE == null`을 체크하여 불필요한 동기화 방지
- *최초 인스턴스 생성 시에만 동기화**가 일어나므로 불필요한 성능 저하가 방지된다.

> **volatile 키워드란?**
> Java에서는 쓰레드를 여러개 사용할경우, 성능을 위해서 각각의 쓰레드들은 변수를 메인 메모리(RAM)으로부터 가져오는 것이 아니라 캐시(Cache) 메모리에서 가져오게 된다. 
> 문제는 비동기로 변수값을 캐시에 저장하다가, 각 쓰레드마다 할당되어있는 캐시 메모리의 변수값이 일치하지 않을수 있다는 점이다.
> 그래서 ~~volatile~~ 키워드를 통해 이 변수는 캐시에서 읽지 말고 메인 메모리에서 읽어오도록 지정해주는 것이다.

![[Pasted image 20250201105244.png|300]]

### 문제점
그러나 `volatile` 키워드를 이용하기위해선 JVM 1.5이상이어야 되고, JVM에 대한 심층적인 이해가 필요하여, JVM에 따라서 여전히 쓰레드 세이프 하지 않는 경우가 발생하기 때문에 사용하기를 지양하는 편이다.


## Bill Pugh Solution (LazyHolder)
가장 권장되는 방식 중 하나로, **정적 내부 클래스(Static Inner Class)** 를 활용하여 싱글톤 객체를 생성하는 방식이다.
```java
class Singleton {

    private Singleton() {}

    // static 내부 클래스를 이용
    // Holder로 만들어, 클래스가 메모리에 로드되지 않고 getInstance 메서드가 호출되어야 로드됨
    private static class SingleInstanceHolder {
        private static final Singleton INSTANCE = new Singleton();
    }

    public static Singleton getInstance() {
        return SingleInstanceHolder.INSTANCE;
    }
}
```

### 개선점
- 내부클래스를 static으로 선언하였기 때문에, `Singleton` 클래스가 로드될 때 `SingleInstanceHolder` 클래스는 로드되지 않는다.
- 어떠한 모듈에서 `getInstance()` 메서드를 호출할 때, `SingleInstanceHolder` 내부 클래스의 static 멤버를 가져와 리턴하게 되는데, 이때 내부 클래스가 한번만 초기화되면서 싱글톤 객체를 최초로 생성 및 리턴하게 된다.
- 마지막으로 final 로 지정함으로서 다시 값이 할당되지 않도록 방지한다.
- synchronized 키워드를 사용하지 않아 성능 저하가 없다.

### 문제점 1 - 리플렉션
리플렉션(Reflection)을 사용하면 싱글톤이 깨질 수 있다.
```java
public class ReflectionSingletonBreak {
    public static void main(String[] args) {
        try {
            // 기존 싱글톤 객체 가져오기
            Singleton instance1 = Singleton.getInstance();

            // 리플렉션을 이용해 private 생성자 호출
            Constructor<Singleton> constructor = Singleton.class.getDeclaredConstructor();
            constructor.setAccessible(true); // private 접근 허용

            Singleton instance2 = constructor.newInstance(); // 새 객체 생성

            // 객체 비교
            System.out.println("instance1 hashcode: " + instance1.hashCode());
            System.out.println("instance2 hashcode: " + instance2.hashCode());
            System.out.println(instance1 == instance2); // false 출력 (싱글톤 깨짐!)

        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

```
instance1 hashcode: 12345678
instance2 hashcode: 87654321
false
```
위 코드 실행 결과, 리플렉션을 사용하면 **private 생성자를 강제로 호출하여 새로운 인스턴스를 만들 수 있다.**


### 문제 2 - 직렬화&역직렬화
직렬화(Serialization) & 역직렬화(Deserialization) 시 새로운 인스턴스가 생성될 수 있다.
```java
import java.io.*;

public class SerializationSingletonBreak {
    public static void main(String[] args) {
        try {
            Singleton instance1 = Singleton.getInstance();

            // 객체를 파일에 저장 (직렬화)
            ObjectOutputStream out = new ObjectOutputStream(new FileOutputStream("singleton.ser"));
            out.writeObject(instance1);
            out.close();

            // 파일에서 객체 불러오기 (역직렬화)
            ObjectInputStream in = new ObjectInputStream(new FileInputStream("singleton.ser"));
            Singleton instance2 = (Singleton) in.readObject();
            in.close();

            // 객체 비교
            System.out.println("instance1 hashcode: " + instance1.hashCode());
            System.out.println("instance2 hashcode: " + instance2.hashCode());
            System.out.println(instance1 == instance2); // false 출력 (싱글톤 깨짐!)

        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

```
instance1 hashcode: 12345678
instance2 hashcode: 87654321
false
```
직렬화 & 역직렬화를 사용하면 새로운 인스턴스가 생성되면서 싱글톤이 깨질 수 있다.


## Enum Singleton
가장 안전한 싱글톤 구현 방법이다. 
자바의 **Enum은 기본적으로 싱글톤을 보장**하므로 멀티스레드 환경에서도 안전하다.

```java
public enum MemberSingleton {
    INSTANCE; // 싱글톤 객체

    private final Member member;

    // 생성자에서 Member 객체 초기화
    MemberSingleton() {
        member = new Member("홍길동", 25);
    }

    public Member getMember() {
        return member;
    }
}
```

```java
public class Main {
    public static void main(String[] args) {
        // 같은 인스턴스를 가져옴
        Member member1 = MemberSingleton.INSTANCE.getMember();
        Member member2 = MemberSingleton.INSTANCE.getMember();

        // 두 객체가 같은지 확인
        System.out.println(member1 == member2); // true 출력 (같은 객체)

        // 정보 출력
        member1.showInfo(); // "이름: 홍길동, 나이: 25" 출력
    }
}
```
### 개선점
- `enum`은 **JVM이 자동으로 인스턴스를 하나만 생성**하기 때문에 추가적인 동기화 코드가 필요 없다.
- 리플렉션 공격을 방어할 수 있다.
- 직렬화 & 역직렬화 시에도 **같은 인스턴스가 유지**된다.

### 단점
- 클래스 상속이 불가능하다.
- `enum`을 사용하면 일반 클래스를 확장할 수 없기 때문에, 싱글톤 클래스에서 다른 클래스를 상속해야 할 경우 사용하기 어렵다.


## 요약

|구현 방식|Thread Safe|성능 저하|리플렉션 문제|직렬화 문제|
|---|---|---|---|---|
|기본 Lazy Initialization|❌|없음|❌|❌|
|synchronized 메서드|✅|성능 저하 있음|❌|❌|
|Double-Checked Locking|✅|성능 저하 적음|❌|❌|
|Bill Pugh 방식 (LazyHolder)|✅|성능 최적화|✅|✅|
|Enum Singleton|✅|없음|✅|✅|


## 싱글톤 패턴은 안티 패턴이다?
싱글톤 패턴은 하나의 인스턴스를 유지하여 메모리 낭비를 방지하고, DBCP(DataBase Connection Pool)와 같이 **공통된 자원을 여러 모듈에서 공유해야 하는 경우**에 유용하게 사용된다.

그러나 싱글톤 패턴은 여러 가지 문제점을 동반하기 때문에 무조건적으로 사용하기보다는 **trade-off를 고려하여 신중하게 적용해야 한다.**

### **싱글톤의 문제점**
#### 1. 모듈 간 의존성이 높아진다.
싱글톤 패턴은 보통 **인터페이스가 아닌 특정 클래스의 정적 메서드를 통해 인스턴스를 반환**하기 때문에, **클래스 간의 강한 결합도가 발생**한다.

즉, **하나의 싱글톤 클래스를 여러 모듈이 공유**하면,
- 싱글톤 클래스의 변경이 다른 모듈에 영향을 미치므로 유지보수가 어려워진다.
- 여러 곳에서 싱글톤을 직접 참조하면 코드의 유연성이 떨어진다.

#### 2. S.O.L.I.D 원칙 위배 가능성
싱글톤 패턴을 잘못 사용하면 **객체 지향 프로그래밍(OOP)의 핵심 원칙인 S.O.L.I.D 원칙을 위반**할 수 있다.

**SRP (단일 책임 원칙) 위반**
- 싱글톤은 애플리케이션 전역에서 공유되기 때문에, 여러 역할을 맡게 되는 경우가 많다.
- 특정 기능만 수행해야 하는 객체가, 싱글톤으로 인해 다양한 책임을 떠안게 되는 문제가 발생한다.

**OCP (개방-폐쇄 원칙) 위반**
- 싱글톤 객체가 직접 사용되면, 새로운 기능을 추가할 때 기존 코드를 수정해야 한다.
- 싱글톤 클래스의 변경이 여러 곳에 영향을 미쳐 확장에 유연하지 않다.

**DIP (의존 역전 원칙) 위반**
- 클라이언트 코드가 인터페이스가 아닌 **구체 클래스에 의존**하게 된다.

따라서 싱글톤 인스턴스를 너무 많은 곳에서 사용할 경우 잘못된 디자인 형태가 될 수도 있다.
그래서 싱글톤 패턴을 객제 지향 프로그래밍의 안티 패턴이라고 불리기도 한다.

#### 3. 단위 테스트(TDD) 시 어려움
싱글톤 인스턴스는 **어플리케이션 전역에서 공유**되기 때문에, **테스트가 독립적으로 수행되기 어렵다.**

단위 테스트는 테스트 간의 독립성이 유지되어야 하며, 테스트를 어떤 순서로든 실행 할 수 있어야 한다. 근데 싱글톤 인스턴스는 자원을 공유하고 있기 때문에, 테스트가 결함없이 수행되려면 매번 인스턴스의 상태를 초기화시켜주어야 한다. 그렇지 않으면 어플리케이션 전역에서 상태를 공유하기 때문에 테스트가 온전하게 수행되지 못할 수도 있다.

또한 많은 테스트 프레임워크가 Mock 객체를 생성할 때 상속에 의존하기 때문에 싱글턴의 클라이언트 코드를 테스트하기 어렵다.


---

그래서 직접 유저가 만들어 사용하는 것 보다는, 스프링 컨테이너 같은 **프레임워크의 도움을 받으면** 싱글톤 패턴의 문제점들을 보완하면서 장점의 혜택을 누릴 수 있다.

스프링 프레임워크에서는 싱글톤 패턴이란게 없고 내부적으로 클래스의 제어를 Ioc(Inversion Of Control) 방식의 컨테이너에게 넘겨 컨테이너가 관리하기 때문에, 이를 통해 평범한 객체도 하나의 인스턴스 뿐인 싱글턴으로 존재가 가능하기 때문에 싱글톤 단점이 없다.




참고
[싱글톤(Singleton) 패턴 - 꼼꼼하게 알아보자](https://inpa.tistory.com/entry/GOF-%F0%9F%92%A0-%EC%8B%B1%EA%B8%80%ED%86%A4Singleton-%ED%8C%A8%ED%84%B4-%EA%BC%BC%EA%BC%BC%ED%95%98%EA%B2%8C-%EC%95%8C%EC%95%84%EB%B3%B4%EC%9E%90#singleton_pattern)
[Enum과 싱글톤(Singleton)](https://scshim.tistory.com/361)