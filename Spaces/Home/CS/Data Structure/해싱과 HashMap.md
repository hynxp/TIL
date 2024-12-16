## 해싱이란 무엇인가?
데이터를 키(key)와 값(value) 형태로 저장하는 경우, `키를 입력받아 특정 위치를 계산하는 과정`을 해싱이라고 한다. 
이때 사용되는 함수가 **해시 함수(Hash Function)**이며,  해시테이블의 **키 값으로 레코드가 저장되어 있는 주소(혹은 색인)를 산출하는 함수**이다.
함수의 출력값을 **해시 값(Hash Value)** 또는 **해시 코드(Hash Code)**라고 부른다.


## HashMap의 구조와 작동 원리
 `HashMap`은 데이터를 **키-값 쌍**으로 저장하는 자료구조로 내부적으로 **배열**과 **연결 리스트**를 조합하여 구현된다.

### 데이터 저장의 기본 구조
![[IMG-20241216225920507.png]]
`HashMap`은 데이터를 저장하기 위해 **배열**을 사용한다. 
이 배열의 각 위치를 **버킷(Bucket)** 이라고 부르며, **해시 함수가 반환한 값을 기반으로 데이터가 특정 버킷에 저장**된다. 

### 성능
**HashMap**은 해시 함수와 충돌 해결 방식에 따라 성능이 달라진다. 평균적으로 데이터의 검색, 삽입, 삭제 연산은 `O(1`)의 시간 복잡도를 가지지만, 충돌이 많을 경우에는 `O(n`)까지 성능이 저하될 수 있다. 


### 해시 함수의 역할
해시 함수는 입력값(키)을 받아 배열의 인덱스를 반환한다.
```java
int hash(Object key) {
    return key.hashCode() % arraySize;
}
```
위 코드에서 `hashCode()` 메서드는 객체의 고유한 정수 값을 반환하며, 배열 크기로 나눈 나머지를 통해 배열의 인덱스를 결정한다. 이러한 방식으로 각 키는 특정 버킷에 저장될 위치를 얻게 된다.

### Hash load factor(해시 로드팩터)란?
`HashMap`의 **Load Factor**는 **해시 테이블에 저장된 데이터 개수 n을 버킷의 개수 k로 나눈 것**으
기본 값은 `0.75`이다.
즉, 해시맵이 75% 차면 크기를 두 배로 늘리고 데이터를 재배치(리해싱)한다는 말이다.

**낮은 Load Factor**로 사용하면 버킷 수(`k`)가 데이터 개수(`n`)에 비해 많다는 것을 의미한다.
즉 버킷이 크기 때문에 충돌이 적어져 조회 속도가 빠르지만, 메모리 낭비가 발생한다.
그렇다고 **높은 Load Factor**를 사용하면 메모리 사용이 효율적이지만 충돌이 늘어나 성능 저하가 발생할 수 있다.


## 해시 충돌
해시 충돌은 2가지의 경우에서 발생한다.
1. `key`는 다른데 hash가 같을 때
2. `key, hash` 둘 다 다른데 hash % map_capacity 결과가 같을 때

즉, 다른 내용의 데이터가 같은 키를 갖고 있을 때 발생한다.
해시 충돌은 해시 함수가 아무리 잘 설계되더라도 완전히 피할 수는 없다. 입력값은 무한하지만 출력값의 가짓수는 유한하기 때문이다. (비둘기집 원리)

> **비둘기집 원리란?**
> n개의 비둘기를 m개의 비둘기집에 넣을 때, 만약 n > m이라면 최소한 하나의 비둘기집에는 두 개 이상의 비둘기가 들어간다.
> 쉽게 말해 수용 가능한 공간보다 넣어야 할 항목이 많을 때 겹치지 않고 넣는 것은 불가능하다는 원리!

충돌이 버킷에 할당된 슬롯 수보다 많이 발생하게 되면 더이상 항목을 저장할 수 없게 되는 **오버플로우**가 발생하게 된다.

자바의 HashMap에서는 이러한 해시 충돌을 처리하기 위해 `체이닝(Chaining)`과 `오픈 어드레싱(Open Addressing)`같은 방법을 사용한다.


### 체이닝(Chaining)
체이닝은 충돌된 데이터를 **연결 리스트**로 관리하는 방식이다. 해시 값이 같아 동일한 버킷에 저장되어야 하는 데이터들은 연결 리스트 형태로 이어진다. 아래는 체이닝 구조를 시각적으로 나타낸 표이다.

|인덱스|데이터|
|---|---|
|0|(없음)|
|1|[A -> B -> C]|
|2|(없음)|
예를 들어, 키 `A`, `B`, `C`가 동일한 해시 값을 가졌다면, 이들은 인덱스 1의 연결 리스트로 저장된다. 하지만 연결 리스트를 순차적으로 탐색해야 하므로 충돌이 많아질수록 성능이 저하될 수 있다.


### 개방 주소법(Open Addressing)
오픈 어드레싱은 충돌이 발생했을 때, 데이터를 저장할 **새로운 빈 버킷**을 찾아나서는 방식이다. 연결 리스트를 사용하는 체이닝과 달리, 데이터는 배열 내의 다른 위치에 저장된다. 이 방식에서는 충돌이 많아질수록 빈 공간을 찾는 데 더 많은 시간이 소요될 수 있다.

#### 1. 선형 탐색(Linear Probing)
해시충돌 시 다음 버켓, 혹은 몇 개를 건너뛰어 데이터를 삽입한다.

#### 2. 제곱 탐색(Quadratic Probing)
해시충돌 횟수의 제곱만큼 이동하면서 빈 버킷을 탐색한다.(1,4,9,16..)

#### 3. 이중 해시(Double Hashing)
해시충돌 시 다른 해시함수를 한 번 더 적용한 결과를 이용함.


### 체이닝 vs 개방 주소법
#### 체이닝(Chaining)의 장점
연결 리스트만 사용하면 된다. 즉, 복잡한 계산식을 사용할 필요가 개방주소법에 비해 적다.
해시테이블이 채워질수록, Lookup 성능저하가 Linear하게 발생한다.

#### 개방주소법(Open Addressing)의 장점
체이닝처럼 포인터가 필요없고, 지정한 메모리 외 추가적인 저장공간도 필요없다.
삽입,삭제시 오버헤드가 적다.
저장할 데이터가 적을 때 더 유리하다.


### 자바에선 어떤 방식을 사용할까?
Java 8부터 HashMap은 **체이닝 방식(Chaining)** 을 사용하여 해시 충돌을 처리한다.
그러다 충돌이 많아지는 경우 `Red-Black Tree`를 활용하여 성능을 개선한다. 

처음에는 연결 리스트(Linked List)를 사용하지만 데이터가 n번 충돌해 연결 리스트의 n번째 노드에 저장된다면 해당 노드 탐색을 위한 시간복잡도는 O(n)이 된다.
따라서 충돌이 많아질수록 효율이 점점 떨어지기 때문에 트리를 사용해 평균 시간 복잡도를 O(n)에서 O(log n)으로 개선한다.

```java
/**  
 * The bin count threshold for using a tree rather than list for a * bin.  Bins are converted to trees when adding an element to a * bin with at least this many nodes. The value must be greater * than 2 and should be at least 8 to mesh with assumptions in * tree removal about conversion back to plain bins upon * shrinkage. */
 * static final int TREEIFY_THRESHOLD = 8;  
  
/**  
 * The bin count threshold for untreeifying a (split) bin during a * resize operation. Should be less than TREEIFY_THRESHOLD, and at * most 6 to mesh with shrinkage detection under removal. */
 * static final int UNTREEIFY_THRESHOLD = 6;
```
HashMap 내부 코드를 보면  `TREEIFY_THRESHOLD` 상수가 있는데 주석을 해석하면 아래와 같다.
"`TREEIFY_THRESHOLD`는 **연결 리스트에서 트리로 변환되는 임계값**이다. 특정 버킷에 저장된 노드(데이터)의 개수가 `TREEIFY_THRESHOLD` 값(기본 8) 이상이 되면 연결 리스트 대신 **레드-블랙 트리**로 변환된다."
즉 같은 버킷에 저장된 노드 수가 8 이상이면 연결 리스트가 레드-블랙 트리로 변환된다는 말이다.

 `UNTREEIFY_THRESHOLD`는 **트리에서 연결 리스트로 다시 변환되는 임계값**이다.
 특정 버킷의 노드(데이터) 개수가 `UNTREEIFY_THRESHOLD` 값(기본 6) 이하로 줄어들면 트리는 다시 연결 리스트로 변환된다.

## 스레드 안전성과 ConcurrentHashMap
기본적으로 `HashMap`은 멀티스레드 환경에서 안전하지 않다. 여러 스레드가 동시에 `HashMap`을 수정하면 데이터 손상이 발생할 수 있다. 
이를 해결하기 위해 동기화된 맵(`Collections.synchronizedMap`) 또는 `ConcurrentHashMap`을 사용할 수 있다.

`ConcurrentHashMap`은 내부적으로 세분화된 락을 사용하여 멀티스레드 환경에서도 안전하게 동작한다.




참고
[해시(Hash)와 해시충돌(Hash Collision)](https://preamtree.tistory.com/20)
[쉬운코드 유튜브](https://youtu.be/ZBu_slSH5Sk?si=ijQo3ouK244EIgJH) <- 이 영상 대박임