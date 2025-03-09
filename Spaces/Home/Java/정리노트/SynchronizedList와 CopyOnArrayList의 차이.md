Java에서는 멀티스레드 환경에서 안전하게 리스트(List)를 사용할 수 있도록 **SynchronizedList**와 **CopyOnWriteArrayList** 두 가지 방법을 제공한다.
이 방법들이 어떤 건지 알아보고 비교까지 해보자.


## SynchronizedList란?
먼저 코드에 있는 주석을 해석해 보았다.
![[Pasted image 20250309025356.png|500]]

> 이 메서드는 **주어진 리스트를 동기화된(스레드 안전한) 리스트로 감싸서 반환한다.**  
> 동기화된 접근을 보장하려면, **반드시 반환된 리스트를 통해서만 원본 리스트에 접근해야 한다.**
> 
> 중요한 주의사항: 반복자 사용 시 동기화 필요
> 반환된 리스트를 `Iterator`, `Spliterator`, 또는 `Stream`을 사용하여 순회할 때는 반드시 수동으로 동기화해야 한다.  
> 이를 따르지 않으면 **비결정적인 동작(non-deterministic behavior)** 이 발생할 수 있다.
```java
List list = Collections.synchronizedList(new ArrayList());
    ...
synchronized (list) {
    Iterator i = list.iterator(); // 반드시 synchronized 블록 안에서 사용해야 함
    while (i.hasNext()) {
        foo(i.next());
    }
}
```
> 위 코드에서 `Iterator`를 사용할 때 반드시 `synchronized (list) {}` 블록 안에서 호출해야 한다.
> `synchronized (list) {}` 블록 없이 `Iterator`를 사용하면 동기화가 보장되지 않아 **ConcurrentModificationException이 발생할 가능성이 있다.**
> 만약 원본 리스트가 **직렬화 가능(Serializable)** 하다면, `synchronizedList()`를 사용하여 생성한 동기화 리스트도 직렬화 가능하다.

위 해석에서 나와있듯이 `SynchronizedList`는 **주어진 리스트를 동기화된(스레드 안전한) 리스트로 감싸서 반환하는** 컬렉션이다.


### 동작 방식
`Collections.synchronizedList()` 메서드는 내부적으로 모든 작업에 `synchronized` 키워드를 사용하여 락을 획득하고 실행한다.
```java
public boolean equals(Object o) {  
    if (this == o)  
        return true;  
    synchronized (mutex) {return list.equals(o);}  
}  
public int hashCode() {  
    synchronized (mutex) {return list.hashCode();}  
}  
  
public E get(int index) {  
    synchronized (mutex) {return list.get(index);}  
}  
public E set(int index, E element) {  
    synchronized (mutex) {return list.set(index, element);}  
}  
public void add(int index, E element) {  
    synchronized (mutex) {list.add(index, element);}  
}  
public E remove(int index) {  
    synchronized (mutex) {return list.remove(index);}  
}  
  
public int indexOf(Object o) {  
    synchronized (mutex) {return list.indexOf(o);}  
}  
public int lastIndexOf(Object o) {  
    synchronized (mutex) {return list.lastIndexOf(o);}  
}  
  
public boolean addAll(int index, Collection<? extends E> c) {  
    synchronized (mutex) {return list.addAll(index, c);}  
}
```
리스트를 새로 생성하지 않기 때문에 생성 비용이 크지 않다.



## CopyOnWriteArrayList란?
이것도 코드 주석을 해석해 보자.
![[Pasted image 20250309030532.png|500]]
> `ArrayList`의 스레드 안전한 변형으로, 모든 변경 작업(add, set 등)이  기본 배열의 새로운 복사본을 생성하여 수행된다.
> 일반적으로 이러한 방식은 비용이 많이 들지만, 탐색 연산이 변경 연산보다 훨씬 많은 경우 효율적일 수 있다. 또한, 탐색을 동기화하고 싶지 않거나 동기화할 수 없는 상황에서 동시에 여러 스레드가 간섭하는 것을 방지해야 하는 경우에도 유용하다.
> "스냅샷(snapshot)" 스타일의 반복자(iterator) 메서드는 반복자가 생성된 시점의 배열 상태를 참조한다. 이 배열은 반복자의 생명 주기 동안 변경되지 않으므로 간섭이 불가능하며, **반복자는 ConcurrentModificationException을 발생시키지 않는다.**
> 다만, 반복자가 생성된 이후 리스트에 추가, 제거, 변경이 이루어지더라도 반복자는 이를 반영하지 않는다.
> 반복자 자체에서 요소를 변경하는 연산(remove, set, add)은 지원되지 않으며, 해당 메서드를 호출하면 UnsupportedOperationException이 발생한다.
> 
> null을 포함한 모든 요소가 허용된다.
> 
> 메모리 일관성 효과: 다른 동시성 컬렉션과 마찬가지로, 한 스레드에서 객체를CopyOnWriteArrayList에 추가하기 전에 수행된 작업은 happen-before 관계를 통해 다른 스레드에서 해당 요소를 접근하거나 제거하는 이후의 작업보다 앞서 실행된다.
> 


### 동작 방식
![[Pasted image 20250309031828.png|400]]
![[Pasted image 20250309031944.png|400]]
`CopyOnWriteArrayList`는 **쓰기 작업(`add()`, `remove()`)이 발생할 때마다 내부 배열을 복사하여 새로운 배열을 만든다.**  
때문에 리스트 크기가 크다면 **배열 복사 비용(overhead)** 이 발생한다.

> 읽기 작업(Iteration)은 기존 배열을 그대로 사용하므로 별도의 동기화 없이 안전하게 수행할 수 있다.



## 성능 비교
`synchronizedList`와 `CopyOnWriteArrayList`의 생성(create), 추가(add), 조회(get)에 대해 성능 테스트를 한 블로그를 보았다.

### create
```java
// synchronizedList
public static <T> List<T> synchronizedList(List<T> list) {  
    return (list instanceof RandomAccess ?  
            new SynchronizedRandomAccessList<>(list) :  
            new SynchronizedList<>(list));  
}

// CopyOnWriteArrayList
public CopyOnWriteArrayList(E[] toCopyIn) {  
    setArray(Arrays.copyOf(toCopyIn, toCopyIn.length, Object[].class));  
}
```

500만 사이즈의 리스트로 생성에 소용된 시간을 비교해 보니
![[Pasted image 20250309032336.png|400]]
6ms로 아주 크진 않다고 한다.


### add
```java
// synchronizedList
public void add(int index, E element) {  
    synchronized (mutex) {list.add(index, element);}  
}  

// CopyOnWriteArrayList
public void add(int index, E element) {  
    synchronized (lock) {  
		// ...
        int numMoved = len - index;  
        if (numMoved == 0)  
            newElements = Arrays.copyOf(es, len + 1);  
        else {  
            newElements = new Object[len + 1];  
            System.arraycopy(es, 0, newElements, 0, index);  
            System.arraycopy(es, index, newElements, index + 1,  
                             numMoved);  
        }  
        // ... 
    }  
}
```
`synchronized`키워드는 두 리스트 모두 사용하지만 `CopyOnWriteArrayList`는 배열 복사라는 오버헤드가 추가로 발생한다.

![[Pasted image 20250309032403.png|400]]
50만 번의 배열 복사 작업을 테스트해봤을 때 `CopyOnWriteArrayList`가 훨씬 느린 것을 볼 수 있다.


### get
```java
// synchronizedList
public E get(int index) {  
    synchronized (mutex) {return list.get(index);}  
}

// CopyOnWriteArrayList
public E get(int index) {  
    return elementAt(getArray(), index);  
}
```
`synchronizedList`는 모든 작업에 synchronized가 걸려있기 때문에 락을 획득한 스레드가 순서대로 get 메서드를 호출한다.

반면 `CopyOnWriteArrayList`는 동기화 장치가 없다.
병렬성을 높이기 위해 리스트의 불변객체를 외부에 제공하기 때문에 락 없이 빠르게 리스트를 순회할 수 있다.

![[Pasted image 20250309033130.png]]
테스트 결과 `CopyOnWriteArrayList`가 2배 이상 빠르다고 한다.

주의할 것은 iterator()가 아닌 반복문으로 원본 리스트를 순회한다면 로직에 따라 동기화 작업이 추가로 필요할 수 있다.
iterator 객체가 생성될 시점의 리스트 상태를 스냅숏으로 반환하기 때문에 잠금 없는 순회를 제공하는 것이다.




참고
[동기화된 리스트를 빠르게 조회하기 - synchronizedList와 CopyOnWriteArrayList 성능 비교해봅시다](CopyOnWriteArrayList)