
## add() 메서드 호출
```java
public boolean add(E e) {  
    modCount++;  
    add(e, elementData, size);  
    return true;  
}

private void add(E e, Object[] elementData, int s) {  
    if (s == elementData.length)  
        elementData = grow();  
    elementData[s] = e;  
    size = s + 1;  
}
```
처음 `add(E e)`메서드를 호출하게 되면 오버로딩된 `add(e, elementData, size)`를 호출한다.
이 메서드에는 추가할 데이터(`e`), 현재 배열(`elementData`), 배열의 현재 크기(`size`)가 전달된다.

현재 배열 크기(`elementData.length`)와 데이터 추가 후 배열 크기(`s`)를 비교한다. 
만약 배열이 가득 찬 경우 배열 크기를 확장하기 위해 `grow()` 메서드를 호출한다.

## grow()
```java
private Object[] grow() {  
    return grow(size + 1);  
}

private Object[] grow(int minCapacity) {  
    int oldCapacity = elementData.length;  // 현재 배열의 크기
    if (oldCapacity > 0 || elementData != DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {  
        int newCapacity = ArraysSupport.newLength(oldCapacity,  
                minCapacity - oldCapacity, oldCapacity >> 1);  
        return elementData = Arrays.copyOf(elementData, newCapacity);  
    } else {  
        return elementData = new Object[Math.max(DEFAULT_CAPACITY, minCapacity)];  
    }  
}
```
현재 배열의 크기를 가져와 변수 `oldCapacity`에 저장한다. 예를 들어, 현재 배열의 크기가 `10`이라면 `oldCapacity = 10`이 된다.

그 다음 `ArraysSupport.newLength` 메서드를 호출하여 oldCapacity의 절반을 더해 새로운 배열 크기(`newCapacity`) 15를 계산한다.

이후 `Arrays.copyOf`를 사용해 크기가 `15`인 배열을 생성하고 기존 데이터를 복사한다.
기존에 크기가 10이던 배열을 더 이상 참조되지 않아 GC에 의해 제거된다.

