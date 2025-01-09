## 스레드 로컬이란?
`ThreadLocal`은 멀티스레드 환경에서 **각 스레드가 독립적인 데이터를 저장하고 관리할 수 있도록 지원하는 메커니즘**이다.

여러 스레드가 동시에 실행되는 애플리케이션에서는 공유 데이터로 인해 경쟁 상태(race condition)가 발생할 수 있다. 이러한 문제를 방지하기 위해 데이터를 스레드 간에 독립적으로 관리해야 할 때 `ThreadLocal`을 사용할 수 있다.


## 스레드 로컬이 필요한 이유
### 1. 스레드 간 데이터 충돌 방지
멀티스레드 환경에서 동일한 데이터를 여러 스레드가 동시에 접근하면, 의도치 않은 결과가 발생할 수 있다.(ex. race condition) `ThreadLocal`은 각 스레드가 독립적인 데이터를 유지할 수 있도록 하여 이런 충돌을 방지한다.

### 2. 세션 및 사용자 정보 관리
웹 애플리케이션에서 각 클라이언트 요청은 별도의 스레드에서 처리된다. `ThreadLocal`을 사용하여 요청 ID, 사용자 정보 등을 저장하면 스레드 간 간섭 없이 데이터를 안전하게 관리할 수 있다. 
그래서 `ThreadLocal`은 웹 애플리케이션에서 사용자 인증 정보를 관리하는 데 자주 사용된다.

### 3. 데이터베이스 트랜잭션 관리
트랜잭션 컨텍스트를 `ThreadLocal`에 저장하면, 동일한 스레드에서 실행되는 모든 데이터베이스 작업이 동일한 트랜잭션 범위에 속하도록 할 수 있다.
 
### 4. 로깅 
로그를 기록할 때 요청별로 고유한 ID를 사용해야 할 경우, `ThreadLocal`을 사용하여 각 요청의 데이터를 관리할 수 있다.


## Thread 내부 코드로 보는 스레드 로컬
```java
public class Thread implements Runnable {
    ThreadLocal.ThreadLocalMap threadLocals;
    ThreadLocal.ThreadLocalMap inheritableThreadLocals;
}
```
`Thread` 클래스에는 `threadLocals`와 `inheritableThreadLocals`라는 두 개의 멤버 변수가 있다. 이 변수들은 스레드별로 데이터를 저장하는 데 사용된다.

- `threadLocals`: 각 스레드가 독립적으로 데이터를 저장하는 맵.
- `inheritableThreadLocals`: 부모 스레드의 데이터를 자식 스레드로 상속하기 위한 맵.

### ThreadLocal 내부 구조
ThreadLocal클래스는 내부적으로 `ThreadLocalMap`을 사용하여 key/value로 데이터를 저장한다.
```java
public class ThreadLocal<T> {
    static class ThreadLocalMap {
        static class Entry extends WeakReference<ThreadLocal<?>> {
            Object value;
            Entry(ThreadLocal<?> k, Object v) {
                super(k);
                value = v;
            }
        }
    }
}
```
`ThreadLocalMap`은 `ThreadLocal` 객체를 키로 사용하여 데이터를 저장하고 값은 해당 객체에 할당된 데이터이다.

예를 들어 `myThreadLocal.set(value)`를 호출하면 `threadLocals` 맵에 `{myThreadLocal: value}` 형태로 저장된다.

## ThreadLocal의 주요 메서드
```java
public void set(T value) {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null) {
        map.set(this, value);
    } else {
        createMap(t, value);
    }
}

public T get() {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null) {
        ThreadLocalMap.Entry e = map.getEntry(this);
        if (e != null) {
            @SuppressWarnings("unchecked")
            T result = (T)e.value;
            return result;
        }
    }
    return setInitialValue();
}
```
ThreadLocal의 `get, set`등의 메서드를 보면 Thread에서 현재 수행중인 thread를 `currentThread()` 메서드를 통해 꺼낸 뒤 이 Thread에서 ThreadLocalMap을 찾아 리턴하는 것을 볼 수 있다.


## inheritableThreadLocals
`inheritableThreadLocals`은 ThreadLocal과 유사하지만, **부모 스레드의 데이터를 자식 스레드로 상속**할 수 있다는 차이가 있다.

요청을 처리하는 스레드가 오랜 특정 작업 때문에 기간 동안 점유되어 스레드 풀로 반환되지 않는다면 동시성이 떨어지게 된다. 이러한 문제를 해결하기 위해 요청의 작업들 중 일부는 비동기로 실행되도록 백그라운드 스레드로 위임시킬 수 있다. 그러면 백그라운드 스레드에서는 스레드 로컬에 저장된 값이 없게 되므로 문제가 생길 수 있으므로, 자바는 자식 스레드에게 스레드 로컬의 값을 위임시켜주는 상속 가능한 `InheritableThreadLocal`을 제공하고 있다.

#### 어떻게? 🤔
부모 스레드가 새로운 스레드를 생성할 때, `inheritableThreadLocals` 맵의 데이터를 복사하여 자식 스레드의 `inheritableThreadLocals`로 전달한다.
이를 통해 자식 스레드는 부모 스레드의 데이터를 초기값으로 가질 수 있다.


### 두 변수의 차이점

|속성|`threadLocals`|`inheritableThreadLocals`|
|---|---|---|
|**관리 클래스**|`ThreadLocal`|`InheritableThreadLocal`|
|**데이터 상속**|상속되지 않음|부모 스레드의 데이터가 자식 스레드에 상속|
|**사용 목적**|각 스레드의 독립적 데이터 저장|부모-자식 스레드 간 데이터 공유|
|**값 관리 주체**|현재 스레드에만 접근 가능|자식 스레드에서 부모의 초기값 접근 가능|


## 예제
다음은 `ThreadLocal`을 사용하여 각 스레드가 독립적인 값을 관리하는 예제이다.
```java
public class ThreadLocalExample {  
    // ThreadLocal 변수 선언  
    private static ThreadLocal<Integer> threadLocal = ThreadLocal.withInitial(() -> 0);  
  
    public static void main(String[] args) {  
        Runnable task = () -> {  
            for (int i = 0; i < 5; i++) {  
                threadLocal.set(threadLocal.get() + 1); // 현재 스레드의 값 갱신  
                System.out.println(Thread.currentThread().getName() + ": " + threadLocal.get());  
            }  
        };  
  
        // 두 개의 스레드 실행  
        Thread thread1 = new Thread(task);  
        Thread thread2 = new Thread(task);  
  
        thread1.start();  
        thread2.start();  
    }  
```

결과를 보면 스레드별로 독립된 값을 유지하는 것을 볼 수 있다.
```
Thread-0: 1
Thread-0: 2
Thread-0: 3
Thread-0: 4
Thread-0: 5
Thread-1: 1
Thread-1: 2
Thread-1: 3
Thread-1: 4
Thread-1: 5
```

## ThreadLocal 사용  시 주의점
ThreadLocal을 사용할 때 반드시 인지해야할 주의할 점이 있다. `ThreadLocal`을 사용할 때 반드시 초기화가 필요하다. 

#### 왜냐!
WAS(Tomcat)와 같은 환경에서는 스레드 풀을 사용하기 때문에, 스레드가 재사용되면서 이전 데이터가 남아 있는 문제가 발생할 수 있다. 따라서 사용이 끝난 후 반드시 `ThreadLocal.remove()`를 호출하여 데이터를 초기화해야 한다.
사용 후에 비워주지 않는다면 해당 Thread를 부여받게 되는 다른 사용자가 기존에 세팅된ThreadLocal의 데이터를 공유하게 될 수도 있다.  


## @Transactional
스프링은 `TransactionSynchronizationManager`라는 내부 클래스를 사용하여 트랜잭션 컨텍스트를 관리한다. 이 클래스는 트랜잭션 속성을 저장하여 스레드 로컬을 활용한다.

트랜잭션 컨텍스트는 스레드별로 독립적이어야 하며, 동일한 스레드 내에서는 일관성을 유지해야 하기 때문이다.

```java
public abstract class TransactionSynchronizationManager {
    private static final ThreadLocal<Map<Object, Object>> resources = new ThreadLocal<>();
    private static final ThreadLocal<Set<TransactionSynchronization>> synchronizations = new ThreadLocal<>();
    private static final ThreadLocal<String> currentTransactionName = new ThreadLocal<>();
    private static final ThreadLocal<Boolean> currentTransactionReadOnly = new ThreadLocal<>();
    private static final ThreadLocal<Integer> currentTransactionIsolationLevel = new ThreadLocal<>();
}
```

| **변수 이름**                          | **역할**                                                                                                           |
| ---------------------------------- | ---------------------------------------------------------------------------------------------------------------- |
| `resources`                        | 현재 스레드에서 사용 중인 트랜잭션 리소스를 저장한다. 리소스는 데이터베이스 연결(`Connection`), 세션(`Session`), 캐시 같은 객체이다.                          |
| `synchronizations`                 | `TransactionSynchronization`은 후처리 작업을 정의하는 인터페이스다. 트랜잭션이 끝난 후 수행해야 할 후처리 작업(커밋 후 로깅, 롤백 후 알림 전송)을 저장한다.          |
| `currentTransactionName`           | 현재 트랜잭션의 이름을 저장한다. 애플리케이션에서 여러 트랜잭션이 있을 경우, 이름을 통해 어떤 트랜잭션이 실행 중인지 구분할 수 있다.                                     |
| `currentTransactionReadOnly`       | 현재 트랜잭션이 읽기 전용인지 여부를 저장한다. 읽기 전용 트랜잭션은 데이터베이스에 변경을 가하지 않으므로 최적화(예: 읽기 전용 커넥션)를 적용할 수 있다. 쓸모없는 업데이트나 성능 저하를 방지한다. |
| `currentTransactionIsolationLevel` | 현재 트랜잭션의 격리 수준을 저장한다.                                                                                            |



참고
[Java Thread Local(쓰레드 로컬)은 무엇일까?](https://velog.io/@wken5577/Java-Thread-Local%EC%93%B0%EB%A0%88%EB%93%9C-%EB%A1%9C%EC%BB%AC%EC%9D%80-%EB%AC%B4%EC%97%87%EC%9D%BC%EA%B9%8C)
[스레드 로컬(ThreadLocal)과 상속 가능한 스레드 로컬( InheritableThreadLocal)에 대하여](https://mangkyu.tistory.com/333)
