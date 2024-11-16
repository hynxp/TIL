## Heap 영역
가비지 컬렉터(GC)를 이해하기 전에 먼저 힙 영역에 대한 이해가 필요하다.

### Heap 영역의 역할
자바의 힙 영역은 [[자바 코드의 메모리 영역(스택&힙)|JVM 메모리 구조]]에서 가장 중요한 부분 중 하나로, 크게 세 가지 주요 역할을 한다.

#### 1. 객체 저장 공간
자바의 객체와 배열(Array)과 같은 참조 타입 변수는 [[자바 코드의 메모리 영역(스택&힙)#힙(Heap)|힙]]영역에 저장된다.
힙 영역에 저장된 데이터는 스택(Stack) 영역에 있는 참조 변수(reference value)를 통해 접근할 수 있다.
[[자바는 Call by Value만 사용한다.#그럼 Call by Reference도 사용하는거 아니야?]]


#### 2. 동적 메모리 할당
프로그램 시작 시에 한 번에 전체 메모리가 할당되는 것이 아니라, **객체가 실제로 필요할 때** 동적으로 메모리가 할당된다.

```java
class Main {
    public static void main(String[] args) {
        // 여기서 person은 아직 생성되지 않음
        Person person; 
        
        // 실행 시점에 메모리 할당
        person = new Person("Alice", 25); 
    }
}
```


#### 3. GC(Garbage Collection)의 대상
힙 영역에 저장된 객체는 참조되지 않을 경우 **가비지 컬렉터(GC)** 에 의해 제거된다.
이는 메모리 누수를 방지하고, 불필요한 객체를 정리하여 프로그램 성능을 유지하는 역할을 한다!



### 힙 영역의 구조

![[IMG-20241116231503273.png]]

힙 영역은 `Young` , `Old`, 그리고 과거에 사용되었던 `Perm` 영역으로 나뉜다.

> PermGen은 JDK 8 이후부터 **Metaspace**로 대체되었다.

#### 1. Young 영역(Young Generation)
새롭게 생성된 객체가 저장되는 영역이다.

`Young` 영역은 다시 `Eden`과 2개의 `Survivor`영역으로 나뉜다.

- **Eden**
객체가 생성되었을 때 처음 할당되는 영역으로, `Young` 영역의 주요 공간이다.
Minor GC가 실행되면 이 영역의 살아남은 객체들이 `Survivor` 영역으로 이동된다.

 - **Survivor 0, Survivor 1(S0, S1)**
Eden 영역에서 살아남은 객체들이 저장되는 공간으로, 두 영역(S0, S1)이 번갈아 사용된다.
즉, 두영역 중 하나는 반드시 비어 있어야 한다.

#### 2. Old 영역(Old Generation)
`Young` 영역에서 여러 번 Minor GC를 거친 **장수 객체**가 이동되는 영역이다.
생명 주기가 긴 객체(전역 객체, 캐시 데이터)가 저장 되기 때문에  `Young` 영역보다 크고, 상대적으로 GC 빈도가 낮다.

#### 3. Metaspace 영역(JDK 8 이상)
PermGen을 대체한 영역으로, 클래스의 메타데이터(구조, 메서드, 필드 정보)를 저장한다.  
Metaspace는 힙 외부의 네이티브 메모리를 사용하며, 동적으로 크기를 조정할 수 있다.


## 가비지 컬렉터(GC)란?
GC는 자바의 메모리 관리 방식 중의 하나로, 힙 영역에 동적으로 할당된 메모리 중 필요 없어진 영역, **즉 어떤 변수도 가리키지 않는 객체를 제거하는 프로세스**를 의미한다.

## 가비지 컬렉터(GC)의 대상
GC는 어떤 오브젝트를 Garbage, 즉 쓰레기로 판단하고 제거하는 것일까?

GC는 객체가 메모리에서 참조되고 있는지의 여부를 기준으로 `Reachable`, `Unreachable` 2가지의 객체로 구분한다.

**1. Reachable (도달 가능한 객체)**
현재 실행 중인 프로그램에서 참조되거나, 참조될 가능성이 있는 객체

**2. Unreachable (도달 불가능한 객체)**
프로그램 내에서 더 이상 참조되지 않거나, 어떤 객체도 이를 참조하지 않는 객체

![[IMG-20241116231503336.png]]
JVM 메모리에서 객체들은 실질적으로 힙 영역에 생성되고 메서드, 스택 영역에는 힙 영역에 생성된 객체의 주소를 참조하는 형식이다.

만약 힙 영역의 객체가 더 이상 참조되지 않아 메서드, 스택 영역에서 해당 객체를 참조하는 변수가 삭제되면 `Unreachable` 객체가 발생하게 된다.
GC는 이 객체를 주기적으로 제거해주는 것이다.


## 가비지 컬렉터(GC)의 동작 방식

GC는 어떤 방식으로 `Unreachable` 객체들을 청소할까?

### Mark-and-Sweep
GC는 보통 **Mark-and-Sweep** 알고리즘을 사용하여 동작하며, 다음 세 단계로 구성된다.

![[IMG-20241116231503376.png]]

#### Mark(식별) 단계
GC가 Root(참조의 시작점)에서 시작하여 객체 그래프를 탐색하며, 각각 어떤 객체를 참조하고 있는지 찾아낸다.
참조되고 있는 모든 객체(`Reachable` 객체)를 식별하고 "마크(Mark)"한다.

#### Sweep(제거) 단계
마크되지 않은 객체(`Unreachable` 객체)를 메모리에서 제거한다.

#### Compaction(압축) 단계
Sweep 후에 메모리에 남아 있는 객체들을 한쪽으로 모아 **연속된 빈 공간**을 확보한다.
객체가 여기저기 분산되어 메모리의 연속된 공간이 없어 큰 객체를 할당할 수 없는 상황 **단편화(Fragmentation)** 를 방지하기 위해 수행된다.

작업 비용(시간과 자원)이 크기 때문에 필요할 때만 수행된다.

## 가비지 컬렉션(GC)의 동작 과정

위에서 힙 메모리 영역은 Young, Old 영역으로 나뉜다고 했다.
**Young** 영역에서 수행되는 GC를 `Minor GC`, **Old** 영역에서 발생하는 GC를 `Major GC, 또는 Full GC`라고 한다.

### Minor GC 과정
**Young 영역**에서 수행되는 GC를 `Minor GC`라고 한다.

Young 영역은 수명이 짧은 객체들이 생성되기 때문에 Old 영역에 비해 상대적으로 작다.
고로 작은 메모리 공간에서 객체를 찾아 제거하기 때문에 적은 시간이 걸려 **Minor**인 것이다.


![[IMG-20241116231503426.png]]



1. 객체가 생성되면 Young의 `Eden` 영역에 저장된다.
2. `Eden` 영역이 가득 차, 새 객체를 저장할 공간이 없을 때 Minor GC가 실행된다.
3. Reachable 객체를 골라내 비어있는 `Survivor` 영역으로 이동한다.(mark)
4. Unreachable 객체의 메모리는 제거한다.(sweep)
5. 살아남은 객체들은 Survivor 영역으로 이동하며, age값이 1씩 증가한다.
6. 특정 임계값(age)에 도달한 객체는 `Old` 영역으로 이동한다(= **Promotion**).


#### Survivor 간 객체 이동과 age
![[IMG-20241116231503464.png|600]]
`Eden` 영역의 Reachable 객체가 살아남으면 활성 `Survivor` 영역(S0 또는 S1)으로 이동한다.
이미 `Survivor` 영역에 있는 객체는 다음 Minor GC 동안 **다른 Survivor 영역**으로 이동한다.
이 과정을 반복하여 객체가 `Survivor` 영역에서 살아남을 때마다 생명 횟수(age)를 증가시켜 기록하는 것이다.

이 age 값이 임계값에 도달하면 `Old` 영역으로 이동한다. = **Promotion(승격)**
이 임계값은 JVM 설정에 따라 변경 가능하다.
예를 들어, `-XX:MaxTenuringThreshold=10`으로 설정하면, 객체가 10번 Minor GC를 살아남았을 때 승격된다.



### Major GC(Full GC) 과정

![[IMG-20241116231503514.png]]
![[IMG-20241116231503549.png]]
 Young 영역에서 살아남은 객체들이 임계값에 도달하면 Old 영역으로 이동한다.(Promotion)
 
이렇게 Young 영역에서 계속 승격(Promotion)되어 Old 영역의 메모리가 부족해지면 **Major GC**가 실행된다.

![[IMG-20241116231503596.png]]

**Major GC**는 Reachable 객체를 식별하고(`mark`), 참조되지 않는 객체를 제거(`sweep`)한 후 필요 시 압축(`Compaction`)하는 단순한 과정이다.

Old 영역은 Young 영역에서 살아남은 **수명이 긴 객체**들이 저장되는 공간으로, **더 큰 메모리 공간**을 필요로 한다.
이로 인해 GC 과정에서 더 많은 작업이 이루어지며, 시간이 오래 걸리므로 Minor GC보다 **소요 시간이 길다**.


## GC의 단점
GC는 메모리를 **언제 해제할지 개발자가 직접 제어할 수 없다는 비결정성**을 가지고 있다. 이로 인해 메모리 해제가 필요한 시점을 정확히 알기 어렵고, GC 동작 중에는 애플리케이션의 **다른 작업이 중단**되어 성능 저하를 유발할 수 있다.
이 중단 시간으로 인해 발생하는 오버헤드 현상을 **Stop-The-World**라 한다.

### Stop-The-World(STW)
**GC를 수행하기 위해 프로그램 실행을 멈추는 것**을 말한다.

GC가 실행되는 동안에는 Mark and Sweep 작업을 수행하는 스레드를 제외한 모든 스레드가 중단된다. 이로 인해 프로그램이 일시적으로 멈추거나 응답 속도가 느려질 수 있기 때문에 너무 자주 GC가 실행되면 성능 저하의 원인이 된다.
따라서 GC로 인한 STW 시간을 최소화하기 위해 다양한 GC 알고리즘이 개발되었다.


## 가비지 컬렉션(GC)의 알고리즘 종류

### Serial GC
**하나의 스레드로 GC를 실행하는 방식**이다.
즉 GC를 처리하는 스레드가 1개(싱글 스레드)여서 Stop The World 시간이 길다.
Minor GC 에는 `Mark-Sweep`을 사용하고, Major GC에는 `Mark-Sweep-Compact`를 사용한다.

장비가 엄청 안 좋지 않은 이상 실무에서 사용하는 경우는 없다.
![[IMG-20241116233430929.png|300]]

프로그램 실행 시 -XX:+UseSerialGC 옵션을 지정하여 사용할 수 있다.
```bash
java -XX:+UseSerialGC -jar Application.java
```

### Parallel GC
Java 8의 디폴트 GC로, **Young 영역의 Minor GC를 멀티 스레드로 수행하는 방식**이다. (Old 영역은 여전히 싱글 스레드로)
Serial GC에 비하면 Stop The World 시간이 짧다.

GC 스레드는 기본적으로 cpu 개수만큼 할당되지만 아래 옵션으로 개수를 설정할 수 있다.
```bash
java -XX:+UseParallelGC -jar Application.java 
# -XX:ParallelGCThreads=N : 사용할 쓰레드의 갯수
```


### Parallel Old GC
Parallel GC를 개선한 버전으로, Young 영역 뿐만 아니라 Old 영역도 멀티 스레드로 GC를 수행하는 방식이다.


### CMS GC(Concurrent Mark Sweep)
**GC스레드와 애플리케이션 스레드를 동시에 실행하는 방식**을 말한다.
즉, GC 작업을 애플리케이션과 동시에 실행하는 것이며 Stop The World 시간을 최대한 줄이기 위해 고안된 방식이다.
![[IMG-20241116234706189.png|300]]

```bash
# deprecated in java9 and finally dropped in java14
java -XX:+UseConcMarkSweepGC -jar Application.java
```

하지만 메모리와 CPU를 많이 사용하고, 메모리 파편화를 해결하는 Compaction이 기본적으로 지원되지 않아 Java9 버젼부터 deprecated 되었고 결국 Java14에서는 사용이 중지되었다.


### G1 GC(Garbage First)
CMS GC를 대체하기 위해 설계된 방식으로, **가비지가 많은 영역**(Garbage First)을 우선적으로 청소하는 방식으로 동작한다. Java 9부터 디폴트 GC로 지정되었다.


![[IMG-20241116235230243.png]]
G1 GC는 힙 메모리를 **Young 영역과 **Old 영역**으로 나누지 않고, **고정된 크기의 여러 개의 Region(영역)** 으로 나눈 다음 아래의 영역을 동적으로 할당한다.

- **Eden Region**: 새롭게 생성된 객체 저장
- **Survivor Region**: Eden에서 살아남은 객체 저장
- **Old Region**: 장수 객체 저장
- **Humongous Region**: 매우 큰 객체(히스토그램 기준 특정 크기 이상) 저장

그 다음 G1 GC는 가장 많은 가비지가 쌓여 있는 Region을 먼저 청소하여 최대한 효율적으로 메모리를 확보한다.
결국 GC 빈도가 줄어드는 효과를 얻는 것이다.

**여러 GC 쓰레드**를 사용하여 병렬로 작업을 수행하고 각 Region에 대해 Compaction(압축)도 하기 때문에 단편화를 방지할 수 있다는 장점이 있다.
특히 **대규모 힙 메모리**를 사용하는 애플리케이션에서 우수한 성능을 발휘한다. (힙이 작을경우 미사용을 권장)


```bash
java -XX:+UseG1GC -jar Application.java
```






참고
[GC(Garbage Collector) 종류 및 내부 원리](https://dongwooklee96.github.io/post/2021/04/04/gcgarbage-collector-%EC%A2%85%EB%A5%98-%EB%B0%8F-%EB%82%B4%EB%B6%80-%EC%9B%90%EB%A6%AC.html)
[☕ 가비지 컬렉션 동작 원리 & GC 종류 💯 총정리](https://inpa.tistory.com/entry/JAVA-%E2%98%95-%EA%B0%80%EB%B9%84%EC%A7%80-%EC%BB%AC%EB%A0%89%EC%85%98GC-%EB%8F%99%EC%9E%91-%EC%9B%90%EB%A6%AC-%EC%95%8C%EA%B3%A0%EB%A6%AC%EC%A6%98-%F0%9F%92%AF-%EC%B4%9D%EC%A0%95%EB%A6%AC#%EA%B0%80%EB%B9%84%EC%A7%80_%EC%BB%AC%EB%A0%89%EC%85%98_%EB%8C%80%EC%83%81)
[10분 테코톡 🤔 조엘의 GC](https://www.youtube.com/watch?v=FMUpVA0Vvjw)