## 프로세스 스케줄링(Process Scheduling)
프로세스 스케줄링은 멀티 프로그래밍과 타임 셰어링의 목적을 달성하기 위해, 실행 가능한 여러 프로세스 중 하나를 선택해 CPU에 할당하는 방식이다.

- **멀티 프로그래밍 (multiprogramming)** : CPU 사용률을 최대화하기 위해 항상 프로세스를 실행하도록 한다. 어떤 프로세스가 CPU를 사용하다가 I/O 작업 등 CPU를 필요로 하지 않는 순간이 오면 다른 프로세스가 CPU를 사용할 수 있도록 한다.
- **시분할 (time sharing)** : 각 프로그램이 실행되는 동안 사용자들이 상호작용할 수 있도록 프로세스 간 CPU 코어를 자주 전환하는 것이다. CPU가 하나의 프로그램을 수행하는 시간을 매우 짧은 시간(ms)으로 제한하여 프로그램을 번갈아 수행하도록 하면 CPU가 하나인 환경에서도 여러 사용자가 동시에 사용하는 듯한 효과를 가져올 수 있다.

각 CPU 코어는 한 번에 하나의 프로세스만 실행할 수 있기 때문에 단일 CPU 코어 시스템에서는 여러 프로세스를 처리할 때 프로세스 스케줄링이 필요하다. 
이렇게 하면 단일 코어에서 여러 작업이 교대로 빠르게 실행되어 효율적인 자원 사용이 가능해진다. 
그러나 멀티 코어 시스템에서는 각 코어가 동시에 별도의 프로세스를 실행할 수 있다.
이로 인해 멀티 코어 시스템은 단일 코어 시스템에 비해 더 많은 작업을 병렬로 처리할 수 있어 성능이 향상된다.

스케줄링의 예로 FCFS(First-Come, First-Served), SJF(Shortest-Job-First), Priority, RR(Round-Robin), Multilevel Queue 등이 있다. 자세한 건 [[CPU 스케줄링(FCFS, SJF, RR, MLFQ)|여기]]서 확인하자.. 

## 선점형과 비선점형 스케줄링
### 선점형 스케줄링
하나의 프로세스가 자원을 사용하고 있을 때 다른 프로세스가 해당 자원을 빼앗을 수 있는 스케줄링

### 비선점형 스케줄링
하나의 프로세스가 자원을 사용하고 있을 때 다른 프로세스가 해당 자원을 빼앗을 수 없는 스케줄링

## 스레드 스케줄링(Thread Scheduling)
프로세스 스케줄링과 마찬가지로 스레드 스케줄링은 운영체제에서 다중 스레드를 관리하며, CPU를 사용할 수 있는 스레드를 선택하고 CPU를 할당하는 작업을 말한다.
## 스케줄링 큐

### Ready Queue

- 프로세스가 시스템에 들어오면 ready queue에 들어가서 CPU 코어에서 실행되기를 기다린다.
- Linked List 형태로 저장되며, ready queue의 header는 list의 첫번째 PCB를 가리키고, 각 PCB의 포인터는 ready queue에 있는 다음 PCB를 가리킨다.

### Wait Queue

- I/O 요청과 같은 특정 이벤트가 처리 완료되기까지를 기다리는 프로세스가 wait queue에 배치된다.
- 프로세스는 waiting 상태에서 ready 상태로 바뀌면 ready queue에 들어가게 된다.



## CPU 스케줄링 알고리즘
CPU 스케줄링은 위의 Ready Queue에 있는 프로세스들을 대상으로 이루어진다.

#### FCFS (First-Come, First-Served) Scheduling

- **비선점 스케줄링**
- 먼저 CPU를 요청하는 프로세스에 먼저 CPU가 할당된다.
- FIFO queue 를 사용해 쉽게 구현할 수 있다.
- 문제점) **convoy effect** : 먼저 들어온 어떤 프로세스의 CPU 처리 시간이 길 경우 다른 모든 프로세스들이 기다림으로서 더 짧은 프로세스가 먼저 진행될 수 있는 경우보다 CPU 및 장치 사용률이 낮아지는 현상

#### SJF (Shortest-Job-First) Scheduling

- **비선점 스케줄링 방식** : CPU burst time이 가장 작은 프로세스에게 먼저 CPU를 할당한다. 만일 CPU burst time이 같다면, FCFS 방식을 적용한다.
- **선점 스케줄링 방식 (SRTF (Shortest-Remaining-Time-First) Scheduling)** : 새로 들어온 프로세스의 CPU burst time이 현재 실행 중인 프로세스의 남은 burst time 보다 작다면 현재 실행 중인 프로세스를 새로 들어온 프로세스가 선점한다.
- [Priority Scheduling](https://github.com/Seogeurim/CS-study/tree/main/contents/operating-system#priority-scheduling)의 한 예이다. (우선순위 = CPU burst time)
- 주어진 프로세스 집합에 대해 **최소 평균 대기 시간**을 제공한다는 점에서 최적의 알고리즘이다. 하지만 CPU burst time을 알 수 있는 방법이 없기 때문에 CPU 스케줄링 수준에서 구현할 수 없다. 이에 대한 한 가지 접근 방식은 SJF 스케줄링을 근사화하는 것이며, 이전 CPU burst time 지수 평균으로 예측할 수 있다.
- 문제점) **starvation** : CPU 처리 시간이 긴 프로세스는 계속 Ready Queue의 뒤로 밀려나기 때문에 무한정 기다리는 상황이 발생할 수 있다.

#### RR (Round-Robin) Scheduling

- **선점 스케줄링**
- 각 프로세스는 **time quantum (or time slice)** 이라는 작은 시간 단위(10-100ms)를 갖게 된다. 프로세스는 1 time quantum 동안 스케줄러에 의해 CPU를 할당 받고, 시간이 끝나면 interrupt를 받아 Ready Queue의 tail에 배치된다.
- Ready Queue는 Circular FIFO queue 형태이다.
- RR의 평균 대기 시간은 긴 편이다. 하지만 공정하다. `n`개의 프로세스가 있고 time quantum이 `q`일 때, 어떤 프로세스도 `(n-1) x q` 시간 단위 이상 기다리지 않는다.

Time quantum 설정 시 주의할 점

- Time quantum이 너무 크다면 : FCFS와 같아진다.
- Time quantum이 너무 작다면 : Context Switching이 너무 빈번하게 일어나 overhead가 발생한다.

위와 같이 RR 알고리즘의 성능은 time quantum의 크기에 좌우될 수 있으므로 적절히 선정해야 하며 이는 context-switch time보다 큰 것이 좋지만 또 너무 커서는 안 된다. (경험적으로 CPU burst의 80프로는 time quantum보다 짧은게 좋다고 함)

#### Priority Scheduling

- 정수로 표현된 우선순위가 더 높은 프로세스에게 CPU를 할당하는 스케줄링이다. 우선순위는 내부적/외부적으로 정의할 수 있다.
    - 내부적 : 시간 제한, 메모리 요구 사항, 열린 파일 수, 평균 I/O burst 대 평균 CPU burst 비율 등 측정 가능한 수량 사용해 계산
    - 외부적 : 프로세스의 중요성, 컴퓨터 사용에 대해 지불되는 자금의 유형 및 금액, 작업을 후원하는 부서, 기타 정치적 요인 등
- **선점 / 비선점 스케줄링** 모두 가능하다.
    - 선점 방식 : 새로 도착한 프로세스의 우선 순위가 현재 실행 중인 프로세스의 우선 순위보다 높으면 CPU 선점
    - 비선점 방식 : 같은 경우 단순히 새 프로세스를 Ready Queue의 맨 앞에 둔다.
- 문제점) **indefinite blocking**, **starvation**
    - 실행할 준비가 되었으나 CPU를 기다리는 프로세스는 block된 것으로 간주될 수 있다.
    - 우선 순위가 낮은 일부 프로세스는 무기한 대기 상태가 될 수 있다.
- 해결 방안) **Aging**, **Round-Robin과 결합**
    - Aging : 오랫동안 대기하는 프로세스의 우선순위를 점진적으로 높이는 방식으로 문제점을 해결할 수 있다. 예를 들어 대기 중인 프로세스의 우선순위를 매초 늘리는 것이다.
    - RR+PS : 우선순위가 가장 높은 프로세스를 실행하는데, 동일한 우선순위의 프로세스에 대해서는 Round-Robin 스케줄링을 적용한다.











참고
[Operating System (운영체제)](https://github.com/Seogeurim/CS-study/tree/main/contents/operating-system#%ED%94%84%EB%A1%9C%EC%84%B8%EC%8A%A4%EC%99%80-%EC%8A%A4%EB%A0%88%EB%93%9C)
