## String Constant Pool이란?
![[IMG-20241116201537333.png]]
우선 **Contant Pool**이란  클래스 내에 사용되는 상수들을 담아놓은공간이다.
클래스 파일(.class)안에 테이블 형태로 들어가있다.

이 Constant Pool에서 문자열 리터럴만을 별도로 관리하는 공간이 **String Pool(String Constant Pool)**이다.

![[String Constant Pool.excalidraw]]

```java
String str1 = "hello";
```
`str1`을 리럴로 선언하게되면 JVM은 String Pool이라는 곳에 "hello" 값과 동일한 문자열이 있는지 확인한다.
존재하지 않으면 String pool에 새롭게 추가한다.

```java
String str1 = "hello";
String str2 = "hello";
```
이어 "hello"라는 값을 가진 문자열을 리터럴로 다시 생성하면 String Pool에 저장되어 있던 "hello"객체를 참조하여 재사용한다. (`str2` = `str1`;)


```java
String str3 = new ("hello"); // 별도의 Heap 메모리에 저장 
```
하지만 `new` 키워드를 사용하면, 같은 문자열이라도 항상 새로운 객체가 힙(heap) 메모리에 생성된다.

때문에 문자열을 사용할 때는 메모리 관점에서 리터럴로 생성하는 게 좋다.

또 **동적으로 생성되는 문자열(DB에서 조회해 오는 String, 파일에서 읽어들인 String)의 경우에도 String Pool에 저장되지 않는다.**
그러므로 동일한 문자열이어도 `==` 로 비교하면 결과는 false가 나오므로 문자열 비교를 위해서는 `equals`(동등연산)으로 비교해야한다.

> [!NOTE]
> Java 7 버전부터 String Pool이 PermGen 영역에서 Heap 영역으로 옮겨지게 되면서 String pool에서도 GC가 수행되게 되어 메모리 부족 오류 위험 또한 줄어들게 되었다. 
>  이로 인해 String Cons~~t~~ant Pool은 Runtime에서 많은 양의 String 객체를 생성하게 되더라도 효율적인 메모리 관리가 가능해졌다.


## String Constant Pool과 Runtime Constant Pool

String Constant Pool은 힙 영역, Runtime Constant Pool은 메서드 영역에 저장된다고 알고 있었다.
String Constant Pool은 Runtime Constant Pool의 하위 개념인데 왜 둘의 저장 위치가 다를까?라는 궁금증이 생겼다. (GC 공부하다가 왜 여기까지 나왔는지 모르겠지만)

### Runtime Constant Pool
자바에서 **Runtime Constant Pool**은 클래스의 **정적 데이터**(클래스 메타데이터) 즉,
클래스 및 메서드 레벨의 다양한 상수(숫자 상수, 메서드 참조, 필드 참조 등)를 저장하는 곳이다.
각 클래스와 인터페이스마다 고유의 Runtime Constant Pool이 존재하며, JVM은 이를 통해 클래스 로드, 메서드 호출, 필드 접근 등의 작업을 수행한다.


### 왜 영역이 다른가?

#### Runtime Constant Pool
`런타임 상수 풀`은  [[자바 코드의 메모리 영역(스택&힙)#메서드 영역(Method Area)|메서드 영역]](Java 7까지는 PermGen, Java 8부터는 Metaspace)에 저장된다. 
 런타임 상수 풀은 클래스별로 존재하는데, 클래스 및 인터페이스의 상수 뿐만 아니라 메서드와 필드에 대한 모든 레퍼런스에 대한 정보를 가지고 있다.
 
#### String Constant Pool
String 객체는 기존 객체를 재사용하지 않고 생성할 때마다 새로운 객체가 생성되는 [[15. String#Immutable(불변성), StringBuffer와 StringBuilder|불변 객체]]이다.
보통 문자열은 동적으로 생성되는 경우가 많고, 필요하지 않은 문자열을 가비지 컬렉션을 통해 정리할 수 있기 때문에 `문자열 상수 풀`을 힙 영역에 따로 두고 재사용하는 것이다.

#### 요약하자면
String Constant Pool은 Runtime Constant Pool의 논리적 하위 개념이지만, JVM의 메모리 관리 최적화를 위해 물리적으로 서로 다른 영역에 저장된다! 






참고
[런타임 데이터 영역(Runtime Data Area)에 대해](https://velog.io/@ddangle/Java-%EB%9F%B0%ED%83%80%EC%9E%84-%EB%8D%B0%EC%9D%B4%ED%84%B0-%EC%98%81%EC%97%ADRuntime-Data-Area%EC%97%90-%EB%8C%80%ED%95%B4)






