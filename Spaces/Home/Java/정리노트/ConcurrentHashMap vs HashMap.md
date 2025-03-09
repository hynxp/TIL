Java에서 HashMap 사용 시 동시성 문제를 해결하기 위해 ConcurrentHashMap을 사용한다고 한다.
![[Pasted image 20250309034223.png]]

| 특징                       | HashMap                | ConcurrentHashMap      |
| ------------------------ | ---------------------- | ---------------------- |
| **내부적으로 동기화?**           | ❌ (동기화되지 않음)           | ✅ (동기화 지원)             |
| **스레드 안전(Thread Safe)?** | ❌ (아님)                 | ✅ (스레드 안전)             |
| **도입된 버전**               | JDK 1.2                | JDK 1.5                |
| **패키지**                  | `java.util`            | `java.util.concurrent` |
| **Null 키 허용 여부**         | ✅ (허용)                 | ❌ (허용 안 함)             |
| **Null 값 허용 여부**         | ✅ (허용)                 | ❌ (허용 안 함)             |
| **반복자 특성**               | Fail-Fast (변경 시 예외 발생) | Fail-Safe (변경해도 예외 없음) |
| **속도**                   | 빠름                     | 상대적으로 느림               |
| **사용 추천 환경**             | 단일 스레드 애플리케이션          | 멀티 스레드 애플리케이션          |


## 1. 스레드 안전
### HashMap
내부적으로 스레드-세이프(thread-safe)를 지원하지 않는다. 
`Collections.synchronizedMap()`을 사용하여 외부적으로 동기화 할 수 있다.

### ConcurrentHashMap
내부적으로 동기화되어있다.
세분화된 락(segment lock)과 CAS(Compare-And-Swap) 기법을 사용하여 동시성을 제어한다. 


## 2. **내부 구조**
### HashMap
동기화를 지원하지 않는다. 멀티스레드 환경에서 `HashMap`을 안전하게 사용하려면 `Collections.synchronizedMap()` 메서드를 사용하여 수동으로 동기화해야 한다.
하지만 모든 동작은 동기화되기 때문에, 성능이 저하될 수 있다.

### ConcurrentHashMap
ConcurrentHashMap의 모든 작업이 동기화되는 것은 아니다. 
추가(`add`) 및 삭제(`delete`)와 같은 수정 작업만 동기화되고 읽기 작업은 동기화되지 않기 때문에
멀티 스레드 애플리케이션에서 외부적으로 동기화된 HashMap보다 우선순위가 될 수 있는 것이다.

하나의 스레드가 해시 맵 객체를 반복하는 동안 다른 스레드가 객체의 내용을 추가/수정하려고하면 `ConcurrentModificationException`이라는 런타임 예외가 발생한다.


## 3. 도입된 버전
### HashMap
JDK 1.2에 `java.util`의 일부가 되었다.

### ConcurrentHashMap
JDK 1.5에 `java.util.concurrent`(동시성 컬렉션 패키지)의 일부가 되었다.
레거시 클래스인 HashTable의 대체제로 취급된다.

> HashTable 클래스의 메서드에는 synchronized 키워드를 사용하고 있어 속도 저하 이슈가 있음



## 4. Null 키 및 null 값 허용 여부

### HashMap
한 개의 `null` 키와 여러 개의 `null` 값을 허용한다.
```java
HashMap<String, String> map = new HashMap<>();
map.put(null, "value1");  // 허용됨
map.put("key2", null);    // 허용됨
map.put("key2", null);    // 허용됨
```

### ConcurrentHashMap
`null` 키와 `null` 값을 모두 허용하지 않는다.
```java
ConcurrentHashMap<String, String> map = new ConcurrentHashMap<>();
map.put(null, "value1");  // NullPointerException 발생
map.put("key2", null);    // NullPointerException 발생
```


## 5. Fail-Fast Vs Fail-Safe
### HashMap
 Fail-Fast 방식을 사용하기 때문에  Iterator가 생성된 이후에 Map 내의 데이터가 변경되었을 때`ConcurrentModificationException`을 던진다.
```java
 Map<String, String> map = new HashMap<>();
map.put("A", "Apple");
map.put("B", "Banana");

Iterator<String> iterator = map.keySet().iterator();
while (iterator.hasNext()) {
    map.put("C", "Cherry"); // ConcurrentModificationException 발생
    System.out.println(iterator.next());
}

```


### ConcurrentHashMap
Fail-Safe 방식을 사용하여 예외가 발생하지 않는다.
Iterator가 반환하는 값은 Iterator가 생성된 시점에서의 Map 상태를 기반으로 하기 때문에 항상 일관된 값을 반환한다.
```java
ConcurrentHashMap<String, String> map = new ConcurrentHashMap<>();
map.put("A", "Apple");
map.put("B", "Banana");

Iterator<String> iterator = map.keySet().iterator();
while (iterator.hasNext()) {
    map.put("C", "Cherry"); // 예외 발생 안 함
    System.out.println(iterator.next());
}
```


## 동작 과정 비교
- **HashMap**
    
    - 배열 + 연결 리스트 + 트리(Red-Black Tree) 구조
    - `put(key, value)`를 호출하면, `key.hashCode()`를 기반으로 **해시 버킷을 결정**
    - 같은 버킷 내에 충돌이 발생하면 **Linked List** 또는 **Red-Black Tree**에 저장됨
- **ConcurrentHashMap**
    
    - 여러 개의 `Segment`(작은 HashMap 그룹)를 사용하여 **버킷 단위로 락을 적용**
    
    JDK 8에서는 **버킷 수준의 락을 제거하고 CAS(Compare-And-Swap) + 스핀락(Spin Lock)을 적용**하여 성능을 향상시켰다.
    
    1. **`put()` 연산 과정**
        - `key.hashCode()`를 기반으로 **버킷 위치 결정**
        - 해당 버킷이 **비어있다면 CAS를 사용하여 값을 삽입** (비동기적으로 동작)
        - 버킷이 **비어있지 않다면 synchronized를 사용하여 동기화 후 삽입**
        - 해시 충돌이 발생하면, **Linked List 또는 Red-Black Tree 구조**로 저장
    2. **`get()` 연산 과정**
        - **락 없이 빠르게 조회 가능**
        - 특정 버킷을 직접 접근하여 데이터를 읽음 (읽기 연산은 동기화 필요 없음)
    3. **`remove()` 연산 과정**
        - 특정 버킷을 찾아서 `synchronized` 블록을 사용해 안전하게 삭제

## 성능
### HashMap
읽기 작업은 HashMap과 ConcurrentHashMap 모두 동기화하지 않기 때문에 동일한 성능을 보인다.

### ConcurrentHashMap
수정 작업만 동기화 된다.
따라서 ConcurrentHashMap의 추가 또는 삭제 작업은 HashMap 보다 느리다.



## 언제 무엇을 사용해야 할까?
### HashMap
HashMap은 내부에서 동기화를 제공하지 않기 때문에 단일 스레드(single-threaded) 애플리케이션에 적합하다.

### ConcurrentHashMap
ConcurrentHashMap은 내부에서 동기화(synchronization)를 제공하여 여러 개의 스레드가 동시에 맵에 접근하고 수정할 수 있도록 하기 때문에 동시 다중 스레드(concurrent multi-threaded) 애플리케이션에 적합하다.




참고
[자바에서 HashMap과 ConccurentHashMap의 차이](https://velog.io/@twinsgemini/%EB%8F%99%EC%8B%9C%EC%84%B1-%EC%9D%B4%EC%8A%88-HashMap-vs-ConcurrentHashMap)
[Difference between HashMap and ConcurrentHashMap](https://www.geeksforgeeks.org/difference-hashmap-concurrenthashmap/)
[동시성 이슈 HashMap vs ConcurrentHashMap](https://velog.io/@twinsgemini/%EB%8F%99%EC%8B%9C%EC%84%B1-%EC%9D%B4%EC%8A%88-HashMap-vs-ConcurrentHashMap)