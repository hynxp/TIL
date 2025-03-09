Java의 `Queue`와 `BlockingQueue`는 둘 다 **FIFO(First-In-First-Out) 원칙**을 따르는 데이터 구조이지만, **멀티스레드 환경에서 동작 방식이 다르다**.  
특히 `BlockingQueue`는 **스레드 간 안전한 데이터 교환을 위해 추가적인 동기화 메커니즘을 제공**한다.


## BlockingQueue란?
**BlockingQueue**는 `Queue` 인터페이스를 상속받아 **큐의 기본 작업에 블로킹 기능이 추가된 인터페이스**이다.  
동기화된 방식으로 동작하므로 여러 스레드가 안전하게 접근할 수 있다.


### 블로킹 기능이란?
BlockingQueue는 한 쪽에서는 `Producer Thread`로 작동하고, 나머지 한 쪽에서는 `Consumer Thread`로 작동한다.

![[Pasted image 20250309064945.png|300]]
한마디로 하나의 쓰레드는 `put`을 하고, 나머지 쓰레드는 `take()`를 하는데 사용된다.

큐가 꽉차있으면 `put()` 과정은 `take()`가 일어날 때까지 대기하게 된다.(큐가 **가득 찼을 때** 요소를 추가하려고 하면 대기)
그리고 큐가 비어있다면 `put()`이 일어날 때까지 `take()` 과정은 대기한다.(큐가 **비어 있을 때** 요소를 가져오려고 하면 대기)
즉, `put()`, `take()` 메서드에서 블로킹이 발생하여, 연산이 완료될 때까지 다음 코드가 실행되지 않는다




## 메서드
![[Pasted image 20250309062003.png]]
각각의 메소드들은 조금씩 다르게 동작한다.


### Throw Exception
바로 실행이 되지 못한다면 `Exception 예외가 발생합니다.`

### Special Value
바로 실행이 되지 못한다면 `true or false를 반환합니다.`

### Blocks
바로 실행이 되지 못한다면 반대 메소드가 실행될 때까지 대기합니다.(put -> take, take -> put)

### Times out
바로 실행이 되지 못한다면 지정한 시간만큼 기다리고 그래도 안된다면 true or false를 반환합니다.


## BlockingQueue와 일반 Queue의 차이점

|특징|Queue|BlockingQueue|
|---|---|---|
|**스레드 안전(Thread Safe)**|❌ 미지원|✅ 지원|
|**동기화 방식**|X (사용자가 직접 동기화해야 함)|내부적으로 락을 사용하여 동기화|
|**요소 추가 시 동작**|가득 차면 예외 발생 또는 `false` 반환|가득 차면 **스레드가 대기(블록)**|
|**요소 제거 시 동작**|비어 있으면 예외 발생 또는 `null` 반환|비어 있으면 **스레드가 대기(블록)**|
|**멀티스레드 환경**|❌ 적합하지 않음|✅ 적합|



## BlockingQueue의 구현체

### 1. [ArrayBlockingQueue](https://github.com/wjdrbs96/Today-I-Learn/blob/master/Java/Collection/Concurrent/Block/ArrayBlockingQueue.md)
- 고정 크기 배열 기반**의 큐 (크기 지정 필수, 변경 불가)
- 요소를 FIFO(선입선출) 방식으로 정렬
- 새 요소는 대기열의 끝에 삽입
- 고정된 크기의 배열에 요소를 보관하는 전형적인 `Bounded Buffer`

### 2. [LinkedBlockingQueue](https://github.com/wjdrbs96/Today-I-Learn/blob/master/Java/Collection/Concurrent/Block)
- **연결 리스트 기반** 큐
- **최대 크기 설정 가능 (설정하지 않으면 무한 크기 큐로 동작)**
- 연결된 노드 기반으로 바인딩된 `Bounded Buffer`
- 요소를 FIFO(선입선출) 방식으로 정렬
- 새 요소는 큐의 끝에 삽입
- 생성 시 사이즈를 지정하지 않으면 사이즈는 `Integer.MAX_VALUE`

### 3. [PriorityBlockingQueue](https://github.com/wjdrbs96/Today-I-Learn/blob/master/Java/Collection/Concurrent/Block)
**우선순위에 따라 요소를 저장**하는 큐


### 4.[SynchronousQueue](https://github.com/wjdrbs96/Today-I-Learn/blob/master/Java/Collection/Concurrent/Block)
- **버퍼 없이 두 스레드 간 직접 데이터 전달**
- `put()`을 호출한 스레드는 `take()`가 호출될 때까지 블로킹됨




## Java BlockingQueue Example
BlockingQueue의 구현체인 `ArrayBlockingQueue`를 이용해서 예제 코드를 보겠습니다.

```java
public class BlockingQueueExample {

    public static void main(String[] args) throws Exception {

        BlockingQueue queue = new ArrayBlockingQueue(1024);

        Producer producer = new Producer(queue);
        Consumer consumer = new Consumer(queue);

        new Thread(producer).start();
        new Thread(consumer).start();
    }
}
```

위와 같이 `Producer` 역할을 하는 클래스와 `Consumer` 역할을 하는 클래스를 만들어 각각 쓰레드로 실행시켜 테스트를 해볼 것입니다.
```java
public class Producer implements Runnable {

    protected BlockingQueue queue = null;

    public Producer(BlockingQueue queue) {
        this.queue = queue;
    }

    public void run() {
        try {
            queue.put("1");
            Thread.sleep(3000);
            queue.put("2");
            Thread.sleep(3000);
            queue.put("3");
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

```java
public class Consumer implements Runnable{

    protected BlockingQueue queue = null;

    public Consumer(BlockingQueue queue) {
        this.queue = queue;
    }

    public void run() {
        try {
            System.out.println(queue.take());
            System.out.println(queue.take());
            System.out.println(queue.take());
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```
위와 같이 `put`을 하고 나서 3초씩 대기하는 코드를 작성하였습니다. 그러면 첫 번째 원소가 INSERT 된 후에 3초를 대기하면 `Consumer` 쓰레드에서 `take()` 과정이 일어날 것입니다.

하지만 위에서 보았던 것처럼 `take()` 메소드는 큐가 비어있다면 `put()` 과정이 일어날 때까지 대기하게 됩니다. 그래서 비어있는 큐에 take() 작업을 해서 예외가 발생하는 것이 아니라 정상적으로 결과가 출력됩니다.




참고
[생산자-소비자 문제와 BlockingQueue](https://velog.io/@be_have98/Java-%EC%83%9D%EC%82%B0%EC%9E%90-%EC%86%8C%EB%B9%84%EC%9E%90-%EB%AC%B8%EC%A0%9C%EC%99%80-BlockingQueue)
[Java BlockingQueue](https://www.youtube.com/watch?v=d3xb1Nj88pw&list=PLL8woMHwr36EDxjUoCzboZjedsnhLP1j4&index=17)
[`BlockingQueue란?`](https://github.com/wjdrbs96/Today-I-Learn/blob/master/Java/Collection/Concurrent/Block/BlockingQueue.md)