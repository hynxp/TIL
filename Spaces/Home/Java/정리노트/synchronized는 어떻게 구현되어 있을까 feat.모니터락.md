동기화 기법은 멀티스레드 환경에서 필수적이다. 동기화 기법으로 [[임계 구역(Critical Section)과 뮤텍스, 세마포어, 모니터|뮤텍스와 세마포어 ]]정도는 알고 있었는데 자바의 synchronized가 세마포어의 단점을 개선하기 위해 나온 모니터를 구현했다고 해서 알아보게 되었다. 

## 세마포어의 단점
세마포어는 멀티스레드 프로그래밍에서 동기화를 구현하기 위해 사용되는 대표적인 도구이다. 그러나 세마포어를 잘못 사용하면 여러 가지 문제가 발생할 수 있다. 

예를 들어 `wait`와 `signal` 연산의 순서를 바꿔서 실행하거나 둘 중 하나라도 생략할 경우 상호 배제(Mutual Exclusion)가 깨지거나 교착 상태(Deadlock)가 발생할 위험이 있다. 
이처럼 세마포어의 단점들을 극복하기 위해 모니터(Monitor)라는 동기화 도구가 도입되었다.


## 모니터란?
![[IMG-20241222025050708.png|400]]
모니터는한번에 하나의 스레드만 실행되어야 할 때, 여러 스레드와 협업이 필요할 때 사용하는 동기화 기법이다.
모니터는 세마포어보다 고수준의 동기화 도구로 뮤텍스를 사용해서 구현한다.

### 모니터의 구성 요소
모니터는 크게 두 가지 요소로 구성되어 있다.

#### 1. 뮤텍스
뮤텍스는 임계 구역(Critical Section)에 대한 접근을 제어하는 락(Lock)이다. 스레드가 임계 구역에 진입하려면 먼저 뮤텍스 락을 취득해야 한다.
만약 다른 스레드가 이미 락을 점유하고 있다면, 해당 스레드는 큐에 들어가 대기 상태로 전환된다.

그 후 뮤텍스 락을 쥔 스레드가 락을 반환하면 대기 상태로 있던 스레드 중 하나가 이어서 락을 갖고 임계 구역에서 실행한다.

#### 2. 조건 변수(condition variable)
내가 제일 어려웠던 개념이 바로 이 조건 변수다.
조건 변수는 **모니터 락** 안에서 사용할 수 있는 도구로, **모니터 내부에서 특정 조건이 충족될 때까지 스레드를 대기 상태로 만들거나, 조건이 충족되었음을 다른 스레드에게 알리는 역할**을 한다.

조건 변수는 웨이팅 큐(`Waiting Queue`)를 가지며, 이 큐는 조건이 충족되기를 기다리는 스레드들이 대기 상태로 머무는 곳이다.

조건 변수에서 제공하는 주요 기능은 다음 세 가지이다

| 기능              | 설명                             |     |
| --------------- | ------------------------------ | --- |
| `wait`      | 현재 스레드를 웨이팅 큐에 넣고 대기 상태로 전환한다. |     |
| `signal`    | 웨이팅 큐에서 대기 중인 스레드 하나를 깨운다.     |     |
| `broadcast` | 웨이팅 큐에서 대기 중인 모든 스레드를 깨운다.     |     |

### 간단한 조건 변수 예제 

다음은 조건 변수를 활용한 **생산자-소비자 문제**의 간단한 구현이다. 
```java
class SharedQueue {
    private Queue<Integer> queue = new LinkedList<>();
    private final int LIMIT = 5;

    public synchronized void produce(int value) throws InterruptedException {
        while (queue.size() == LIMIT) {
            wait(); // 큐가 가득 찼을 경우 대기
        }
        queue.add(value);
        System.out.println("Produced: " + value);
        notify(); // 소비자에게 알림
    }

    public synchronized int consume() throws InterruptedException {
        while (queue.isEmpty()) {
            wait(); // 큐가 비어 있으면 대기
        }
        int value = queue.poll();
        System.out.println("Consumed: " + value);
        notify(); // 생산자에게 알림
        return value;
    }
}
```
이 예제에서 `wait`와 `notify`를 사용하여 **생산자와 소비자가 큐의 상태를 공유하며 협력**하는 구조를 구현하였다.


## 자바의 Synchronized
자바의 `synchronized` 키워드는 내부적으로 모니터 메커니즘을 사용하여 동기화를 구현한다. 
자바에서 모든 객체는 **모니터 락(Monitor Lock)**을 가지고 있다. 스레드가 객체에 접근하려면 먼저 모니터 락을 획득해야 하며, 락을 얻지 못한 스레드는 대기 상태로 전환된다.

자바의 모니터는 한 개의 조건 변수만 가지며, 실행 순서의 제어가 필요할 경우 `wait`, `notify`, `notifyAll` 메서드를 사용할 수 있다.

### Synchronized의 문제점
`synchronized`는 블로킹 방식을 사용하여 동기화를 구현하기 때문에 몇 가지 성능상의 단점을 가지고 있다. 
예를 들어 특정 스레드가 `synchronized` 블록에서 락을 점유하고 있는 동안 다른 스레드들은 대기 상태에 들어가게 된다. 이는 자원을 낭비하고 프로그램의 성능을 저하시킬 수 있다.








참고
[10분 테코톡 베로의 Monitor & Synchronized](https://www.youtube.com/watch?v=4t8BennljZA)
[쉬운코드 유튜브: 모니터가 어떻게 동기화에 사용되는지 아주 자세히 설명합니다! 자바에서 모니터는 어떤 모습인지도 설명하니 헷갈리시는 분들 꼭 보세요!](https://www.youtube.com/watch?v=Dms1oBmRAlo)