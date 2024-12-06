자바에서 동적 배열을 다룰 때 가장 많이 사용되는 자료구조는 `ArrayList`와 `Vector`이다. 이 둘은 모두 `List` 인터페이스를 구현하고 있어 비슷해 보이지만, 내부적으로 많은 차이점을 가지고 있다. 


## ArrayList와 Vector의 공통점
`ArrayList`와 `Vector`는 자바에서 동적 배열을 지원하는 클래스이다. 기본적으로 배열을 기반으로 하며, 배열의 크기가 가득 차면 자동으로 크기를 확장한다. 두 클래스 모두 다음과 같은 특징을 을 갖고있다.

- 데이터 삽입 순서를 유지한다.
- 인덱스를 기반으로 요소에 접근할 수 있다.
- 중복된 요소를 허용한다.


## ArrayList와 Vector의 차이점
### 동기화 지원 여부
ArrayList와 Vector의 가장 큰 차이점은 **동기화(Synchronization) 지원 여부**이다. `Vector`는 기본적으로 동기화를 지원한다.
```java
public synchronized int capacity() {  
    return elementData.length;  
}  
  
public synchronized int size() {  
    return elementCount;  
}  
  
public synchronized boolean isEmpty() {  
    return elementCount == 0;  
```
내부 코드를 보면 대부분의 메서드가 `synchronized` 키워드로 동기화되어있는 것을 확인할 수 있다.
이로 인해 데이터의 일관성은 지킬 수 있지만 오버헤드가 생겨 일반 메서드보다 속도가 느릴수도 있다. 또한 Vector의 동기화는 메서드에만 `synchronized` 키워드로 되어 있어 메서드 자체 실행으로는 스레드 세이프하지만, **Vector 인스턴스 객체 자체에는 동기화가 되어있지 않다.**
그래서 멀티 스레드 환경에서도 마냥 안전하다고는 할 수 없다.

ArrayList는 동기화를 지원하지 않아 불안전하고 여러 스레드가 동시에 액세스가 가능하다.


### 확장 방식
배열의 크기가 가득 찼을 때 확장되는 방식도 차이가 있다.
`ArrayList`는 기존 배열 크기의 50%만큼 확장한다.
`Vector`는 기존 배열 크기의 100%만큼 확장한다.

이로 인해 `Vector`는 대규모 데이터를 처리할 때 더 많은 메모리를 차지할 수 있지만 빈번한 크기 변경을 피할 수 있다.


### 요약

|특징|ArrayList|Vector|
|---|---|---|
|**동기화 지원 여부**|지원하지 않음|지원 (기본 동기화)|
|**성능**|더 빠름|더 느림|
|**확장 방식**|기존 크기의 50%|기존 크기의 100%|
|**사용 환경**|단일 스레드 환경에 적합|멀티스레드 환경에 적합|
|**레거시 여부**|컬렉션 프레임워크의 일부|레거시 클래스|

## 그럼 Vector를 사용해야하나?
그렇다면 배열 기반 컬렉션이 필요할 때 Vector를 사용해야 하는걸까?
**아니다 ArrayList를 그대로 사용하면 된다.**

**Vector**는 아주 오래전에 등장한 클래스로, 내부적으로 설계가 제대로 되지 않았을 때 등장 클래스로 이미 deprecated 되어 사용을 권장하지 않는다.
자바의 하위 호환성을 위해 유지되고 있을 뿐이다.
`Collections.synchronizedList()` 메서드를 통해 동기화 처리할 수 있으므로 ArrayList를 사용하는 게 적절하다.




[ArrayList vs Vector 동기화 & 성능 차이 비교](https://inpa.tistory.com/entry/JCF-%F0%9F%A7%B1-ArrayList-vs-Vector-%EB%8F%99%EA%B8%B0%ED%99%94-%EC%B0%A8%EC%9D%B4-%EC%9D%B4%ED%95%B4%ED%95%98%EA%B8%B0)
