자바를 접하게 되면 제일 먼저 하는 것이 `System.out.println("Hello, World!")`일 것이다.
그만큼 기초이자, 아주 많이 사용하는 코드이다.
하지만 `System.out.println`은 실무에서 절대 사용하지 말라고들 한다.


## 왜일까?
성능 이슈로는 크게 2가지가 있다. 블로킹 I/O작업을 한다는 점과 멀티스레드에서 락이 발생한다는 점이다.

### 1. 블로킹 I/O 작업
먼저 System 클래스의 static 변수인 `out`은 **PrintStream** 타입의 인스턴스이다.
```java
public static final PrintStream out = null;
```

그리고 **PrintStream** 클래스는 java의 io 패키지 내에 있다.
```java
package java.io;

public class PrintStream extends FilterOutputStream  
    implements Appendable, Closeable
```
즉, println()은 I/O 작업이며 이는 **프로그램이 데이터를 출력할 때 운영체제(OS)의 표준 출력 장치(콘솔 창, 터미널)로 데이터를 보내는 작업을 수행한다는 뜻**이다.

이는 [[Sync&Async, Blocking&Non-Blocking#Blocking&Non-Blocking|Blocking I/O 방식]]으로 동작하며 이는 데이터 출력 작업이 완료될 때까지 호출한 스레드가 블록(block)되어 다른 작업을 처리하지 못하는 방식을 의미한다.


### 2. synchronized 작업
표준 출력(System.out)은 공유 자원이기 때문에 여러 스레드에서 동시에 접근할 경우 출력 순서가 뒤섞이거나 충돌할 가능성이 있다.
이를 방지하기 위해 println 내부에선 `snychronized` 를 사용해 동기화 처리한다.
(String 뿐만 아니라 다른 매개변수도 비슷하다.)

```java
public void println(String x) {  
    synchronized (this) {  
        print(x);  
        newLine();  
    }  
}
```
`synchronized`은 한 번에 하나의 스레드만 자원을 사용할 수 있도록 제한한다.
따라서 멀티스레드 환경에서 다수의 스레드가 동시에 로그를 출력하려고 하면 대기 시간이 증가하고 병목 현상이 발생한다.

즉, `System.out.println()`이 끝날때까지 아무 일도 할 수 없고 대기해야 하기에 오버헤드가 발생해 성능이 저하되는 것이다.


### + 로그 레벨관리가 어려움
`System.out.println`은 로그의 중요도(DEBUG, INFO, WARN, ERROR 등)를 구분하지 못한다. 이는 프로덕션 환경에서 디버깅 메시지와 중요한 오류 로그가 섞이는 문제를 야기할 수 있다.


## 해결 방법은?
실무에서는 `Log4j, SLF4J` 같은 전문 로깅 프레임워크를 사용하는 것이 일반적이다.
디버깅 및 모니터링에 더 효과적이다.
이 도구들은 로그를 별도의 스레드에서 처리해 병목 현상을 최소화하고, 로그의 중요도를 구분해 출력할 수 있다.

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class MyClass {
    private static final Logger logger = LoggerFactory.getLogger(MyClass.class);

    public static void main(String[] args) {
        logger.info("This is an INFO message.");
        logger.debug("This is a DEBUG message.");
        logger.error("This is an ERROR message.");
    }
}
```





참고
[System.out.println() 사용을 자제해야 하는 이유](https://velog.io/@destiny1616/System.out.println-%EC%82%AC%EC%9A%A9%EC%9D%84-%EC%9E%90%EC%A0%9C%ED%95%B4%EC%95%BC-%ED%95%98%EB%8A%94-%EC%9D%B4%EC%9C%A0)
[System.out.println 메소드는 실무에서 `절대 사용하지마라.`](https://systemdata.tistory.com/21)