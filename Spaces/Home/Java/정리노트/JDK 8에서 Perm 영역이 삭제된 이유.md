![[IMG-20241122112623375.png|500]]

JDK 8부터 자바는 기존의 PermGen(Permanent Generation) 영역을 제거하고, Native Memory를 사용하는 Metaspace를 도입했다.


## PermGen이란?
`PermGen`(이하 Perm) 영역은 JDK 7까지 존재했던 메모리 영역으로, 클래스 메타데이터와 런타임 중에 읽힌 클래스 및 메서드 정보 등을 저장하는 공간이었다. 

**Perm에 저장되는 데이터**
- 클래스 메타데이터(클래스 이름, 메서드 이름, 필드 이름 등)
- 런타임 상수(Constant Pool)
- Static 변수(JDK 8 이전)
- 메서드와 바이트코드

`Perm` 영역은 힙(Heap) 외부에 위치하며, JVM 실행 시 크기가 고정된다.

하지만 이 고정된 크기는 메모리 부족 문제를 자주 일으켰다. 동적으로 클래스들이 로드되고 Static 변수나 상수가 Perm 영역에 계속 쌓이면서 `OutOfMemoryError`이 자주 발생했다.

JDK 8에서는 이 문제를 해결하기 위해 `Perm`을 제거하고, `Perm` 영역에서 관리하던 것들을 `Metaspace`로 옮기게 되었다. 즉, 각종 메타 정보를 OS가 관리하는 영역으로 옮겨 `Perm` 영역의 사이즈 제한을 없앤 것이라 할 수 있다.

> 이 과정에서 Static 변수는 Heap 영역으로 옮겨져서 GC의 대상이 되었다.


## PermGen 제거 -> Metaspace 도입
Metaspace는 JVM의 새로운 클래스 메타데이터 저장소로, **클래스 로더가 런타임 중에 읽어온 정보**가 저장된다.

**Metaspace에 저장되는 데이터**
- 클래스 구조 정보(필드, 메서드, 바이트코드 등)
- 예외 정보
- Constant Pool
- Annotation

Perm영역이 Metaspace로 옮겨짐으로써 가장 중요한 차이는 Metaspace가 **Heap 메모리가 아닌 Native Memory**를 사용한다는 점이다.

Native Memory를 활용하는 Metaspace는 JVM 힙 메모리와 독립적이므로, 운영체제가 직접 관리한다. 즉, Metaspace의 크기는 런타임 중 동적으로 확장되기 때문에 개발자가 이를 별도로 관리할 필요가 없는 것이다.

### 이후의 변화
JDK 8부터 PermGen이 제거되고 **Metaspace**가 도입됨으로써 변화는 다음과 같다.

#### 1. Static 변수와 상수 풀 관리의 변화
기존 PermGen에서 관리되던 Static 변수와 상수(Constant Pool)는 이제 **Java Heap 영역**으로 옮겨져 GC의 대상이 되었다. 이를 통해 메모리 관리가 더 효율적으로 이루어졌으며, 메모리 누수 가능성도 줄어들었다.

#### 2. 클래스 메타데이터 관리의 유연성
Metaspace는 PermGen과 마찬가지로 클래스 메타데이터를 저장한다. 하지만 Native Memory를 활용하기 때문에 메모리 크기를 OS가 자동으로 조정할 수 있다.
즉, 개발자가 PermGen 크기(`-XX:PermSize`, `-XX:MaxPermSize`)를 지정할 필요가 없어졌다.

#### 3. OutOfMemoryError 문제 완화
PermGen의 고정 크기 제약이 없어지면서 메모리 부족으로 인한 `OutOfMemoryError` 문제가 발생할 가능성이 줄어들었다.




참고
[Perm 영역이 Metaspace로 바뀐 이유](https://jaehoney.tistory.com/177)
[JDK 8에서 Perm 영역은 왜 삭제됐을까](https://johngrib.github.io/wiki/java8-why-permgen-removed/)