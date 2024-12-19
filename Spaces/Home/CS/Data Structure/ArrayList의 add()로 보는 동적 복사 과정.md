
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
기존 배열(크기가 10이던)은 더 이상 참조되지 않아 GC에 의해 제거된다.

### Arrays.copyOf()은 얕은 복사? 깊은 복사?
많은 블로그에서 깊은 복사의 방법으로 Arrays.copyOf()을 설명하지만, 이는 틀린 설명이다.

```java
public static <T,U> T[] copyOf(U[] original, int newLength, Class<? extends T[]> newType) {  
    @SuppressWarnings("unchecked")  
    T[] copy = ((Object)newType == (Object)Object[].class)  
        ? (T[]) new Object[newLength]  
        : (T[]) Array.newInstance(newType.getComponentType(), newLength);  
    System.arraycopy(original, 0, copy, 0,  
                     Math.min(original.length, newLength));  
    return copy;  
}

```

코드를 보면 `System.arraycopy()`라는 메서드를 수행하는데 이 메서드는 `original[i]`의 값을 `copy[i]`에 복사한다.

즉, 기본 타입 배열인 경우 배열의 각 요소 값을 복사하고, 참조 타입 배열인 경우 배열 요소가 참조하는 객체의 주소값(참조)을 복사한다.

기본 타입 배열은 값 자체를 복사하므로 원본과 독립적이라는 점에서 깊은 복사라고 생각할 수 있지만 기본 타입의 경우 객체 참조가 아니라 **값** 자체를 다루기 때문에, 얕은 복사가 동작하더라도 결과적으로 원본에 영향을 미치지 않아 **깊은 복사처럼 보이는것이다.**

