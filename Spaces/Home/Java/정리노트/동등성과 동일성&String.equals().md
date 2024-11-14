## 동일성, ==

동일성은 동일하다는 뜻으로 두 개의 객체가 완전히 같은 경우를 의미한다. 
여기서 완전히 같다는 것은 두 객체의 주소값이 같아 동일하다고 봐도 무방하다는 말이다.

`==`와 `!=` 연산자는 기본 자료형에서만 값 비교를 위해 사용할 수 있다.
참조 자료형에서 사용해도 되지만 **참조 자료형에서 사용하면** 값을 비교하는 게 아니라 **주소값**을 비교한다.

```java
Parent parent1 = new Parent("hyun");
Parent parent2 = new Parent("hyun");

System.out.println(parent1 == parent2); //false
```
Parent 객체 안에 `String name` 이라는 변수에 같은 값을 넣고 비교해도 결과는 false다.
두 객체는 각자의 생성자를 사용하여 만들었기 때문에 주소값이 다르기 때문이다.

## ## 동등성, equals()

동등성은 동등하다는 뜻으로 두 개의 객체가 같은 정보를 갖고 있는 경우를 의미한다. 
동등성은 두 객체의 주소가 서로 다르더라도 내용만 같으면 동등하다고 이야기 할 수 있다. 

동일하면 동등하지만, 동등하다고 동일한 것은 아닌 것이다.

참조 자료형은 equals()를 오버라이딩하여 비교해야 한다.
그렇지 않으면 `==`처럼 hashCode()값을 비교하므로 equals()를 오버라이딩해 주소가 아닌 필드값을 비교하도록 재정의 해주어야 한다.

대표적인 예로 객체 타입인 String 클래스의 equals 오버라이딩을 보면 주소값이 아닌 문자열을 하나하나 비교한다.
```java
public boolean equals(Object anObject) {
    if (this == anObject) {         // 동일한 객체를 비교할 때 true를 반환
        return true;
    }
    if (anObject instanceof String) { // 비교 대상이 String 타입인지 확인
        String anotherString = (String) anObject;
        int n = value.length;
        if (n == anotherString.value.length) {  // 길이가 동일한지 확인
            char v1[] = value;
            char v2[] = anotherString.value;
            int i = 0;
            while (n-- != 0) {       // 각 문자를 하나씩 비교
                if (v1[i] != v2[i]) { // 문자 하나라도 다르면 false 반환
                    return false;
                }
                i++;
            }
            return true;              // 모든 문자가 동일할 경우 true 반환
        }
    }
    return false;                    // 타입이 다르거나 길이가 다른 경우 false 반환
}

```

원래 `equals()` 메서드는 객체의 주소값을 비교하는데, String의 `equals()`가 재정의 되어있어 문자열 값으로 비교할 수 있는 것이다.

## 오버라이딩할 때는 hashCode()도 같이 오버라이딩 해야 한다.

같은 클래스의 name 값이 "hyun"인 2개의 객체를 equals()로 비교했을 때 객체가 서로 같다고는 할 수 있지만,
그 값이 같다고 해서 그 객체의 주소 값이 서로 같지는 않기 때문이다.(각각 힙 영역에 저장되어 있기 때문에)
→ equals()결과가 true여도 hashCode() 값은 서로 다르기 때문이다.

```java
//기본 equals() 
public boolean equals(Object obj) {
    return (this == obj);
}
```
```java
@Override
public boolean equals(Object o) {
    if (this == o) return true;
    if (o == null || getClass() != o.getClass()) return false;
    Parent parent = (Parent) o;
    return Objects.equals(name, parent.name);
}

@Override
public int hashCode() {
    return Objects.hash(name);
}
```

두 메소드를 재정의 하지 않을시, 
hash 값을 사용하는 **Collection Framework(HashSet, HashMap, HashTable)** 을 사용할 때 문제가 발생한다고 한다. 

예를 들어 name 값이 "hyun"인 2개의 객체를 HashSet자료형에 넣는다면 중복 판별되어 하나의 데이터만 컬렉션에 들어간다고 예상하지만 예상과 다르게 2개가 들어간다.
이는 해시코드가 다르기 때문에 중복된 데이터가 컬렉션에 추가된 것이다.
참고 : [자바 equals / hashCode 오버라이딩 - 완벽 이해하기](https://inpa.tistory.com/entry/JAVA-%E2%98%95-equals-hashCode-%EB%A9%94%EC%84%9C%EB%93%9C-%EA%B0%9C%EB%85%90-%ED%99%9C%EC%9A%A9-%ED%8C%8C%ED%97%A4%EC%B9%98%EA%B8%B0)


### 객체의 고유값을 나타내는 hashCode()
hashCode() 메서드는 객체의 메모리 주소를 16진수로 리턴한다.
만약 어떤 두 개의 객체가 서로 동일하다면, hashCode() 값은 무조건 동일해야만 한다.
직접 구현할 일은 거의 없다.
오버라이딩 하여 문자열값이 같을 경우 같은 해시코드를 갖게 하도록 만들어주면 된다.
