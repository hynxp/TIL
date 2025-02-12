보통 동시성 문제를 해결하기 위해 자바 코드레벨에서 해결하는 방법으로 `synchronized`이나 `volatile` 키워드를 사용할 수도 있지만 각각 문제점이 존재한다.

이미 두 키워드에 대해 어느정도 안다고 가정하고, 간단하게 얘기하자면 `synchronized`은 락을 사용하기 때문에 한번에 한 스레드만 수행할 수 있어 성능상 문제가 있다.
또한 `volatile`도 `++`와 같이 원자성을 보장하지 않는 연산에서는 동시성 문제가 해결되지 않는다.

그 해결법으로 Atomic 클래스가 있는데 어떤건지 알아보자 레쓰고
  

## Atomic Class
자바에서 동시성을 보장하는 중요한 개념 중 하나가 **`Atomic(원자적)`** 자료형이다. 
`Atomic`은 논블로킹이면서 락을 사용하지 않는다. 
`Atomic`의 핵심은 이러한 소모 비용을 줄이는 `non-blocking` 방식을 사용한다는 점에서 차이점이 존재한다. 즉, 어떤 스레드도 suspended 되지 않기 때문에 context switch를 피할 수 있다.

그 이유는  내부적으로 **Compare-And-Swap(CAS) 알고리즘**을 사용하여 동시성을 보장한다.


## CAS(Compare-And-Swap) 알고리즘
이들 클래스는 내부적으로 **CAS 알고리즘을 사용해 동시성을 보장**한다.

CAS 알고리즘은 "비교 후 교환"하는 방식으로 동작한다. 하드웨어에서 제공하는 **원자적 연산(Atomic Operation)**을 이용하여 경쟁 상태를 피한다.

### CAS의 동작 방식
CAS 알고리즘은 다음과 같은 3가지 값을 사용한다.

- **기대 값 (Expected Value)**: 현재 예상하는 값
- **새로운 값 (New Value)**: 변경하고자 하는 값
- **메모리 주소 (Memory Address)**: 값이 저장된 주소

연산 과정은 다음과 같다.
1. 현재 값과 기대 값이 같은지 비교한다.
2. 같다면 새로운 값으로 변경한다.
3. 다르다면 변경하지 않고 아무 작업도 수행하지 않는다.

이 방식은 `synchronized` 키워드와 달리 블로킹 없이 동작하여 성능이 향상된다.


### CAS 예제 (AtomicInteger)
아래는 `AtomicInteger`를 이용하여 CAS 연산을 수행하는 예제이다.

```java
import java.util.concurrent.atomic.AtomicInteger;

public class AtomicExample {
    public static void main(String[] args) {
        AtomicInteger atomicInteger = new AtomicInteger(0);

        // CAS를 이용한 값 변경
        boolean success = atomicInteger.compareAndSet(0, 10);

        System.out.println("CAS 연산 성공 여부: " + success);
        System.out.println("현재 값: " + atomicInteger.get());
    }
}
```

위 코드는 `atomicInteger`가 0일 때만 10으로 변경한다. `compareAndSet(0, 10)` 메서드는 내부적으로 CAS 연산을 수행하여 값이 예상 값과 동일하면 새로운 값으로 설정한다.


## CAS의 장점과 단점

### 장점
- **비교적 높은 성능**: `synchronized`를 사용하지 않으므로, 블로킹 없이 동작하여 성능이 향상된다.
- **경량화된 동기화**: 락을 사용하지 않으므로 오버헤드가 줄어든다.

### 단점
- **ABA 문제 발생 가능**: 값이 예상한 값으로 변경되었는지는 확인하지만, 변경 과정에서 다른 값으로 변했다가 다시 원래 값으로 돌아온 경우 이를 감지하지 못한다.
- **무한 루프 가능성**: CAS 연산이 반복적으로 실패할 경우 무한 루프가 발생할 수도 있다.

## ABA 문제 해결 방법
CAS의 한 가지 단점은 **ABA 문제**이다. 예를 들어, `A → B → A`로 값이 변한 경우 CAS는 단순히 `A`를 확인하기 때문에 값이 바뀌었는지를 감지할 수 없다. 이를 해결하기 위해 `AtomicStampedReference<T>`를 사용하면 **버전 정보(스탬프)**를 함께 관리할 수 있다.

```java
import java.util.concurrent.atomic.AtomicStampedReference;

public class ABAExample {
    public static void main(String[] args) {
        AtomicStampedReference<Integer> atomicStampedRef = new AtomicStampedReference<>(100, 1);

        int[] stampHolder = new int[1];
        int currentValue = atomicStampedRef.get(stampHolder);

        boolean success = atomicStampedRef.compareAndSet(currentValue, 200, stampHolder[0], stampHolder[0] + 1);

        System.out.println("CAS 연산 성공 여부: " + success);
        System.out.println("현재 값: " + atomicStampedRef.getReference());
        System.out.println("현재 스탬프: " + atomicStampedRef.getStamp());
    }
}
```
여기서 `AtomicStampedReference`는 값을 변경할 때마다 **스탬프(버전)를 증가**시키므로, 값이 같은지뿐만 아니라 변경된 이력이 있는지도 확인할 수 있다.

