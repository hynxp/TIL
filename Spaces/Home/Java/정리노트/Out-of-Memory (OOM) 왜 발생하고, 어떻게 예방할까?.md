Java 애플리케이션을 개발하다 보면 `OutOfMemoryError`라는 런타임 에러를 마주칠 때가 있다. JVM이 더 이상 메모리를 할당할 수 없을 때 발생하는 이 에러는 코드나 환경 문제를 나타내는 강력한 신호다. 이번 글에서는 OOM 에러가 발생하는 이유와 이를 예방하기 위한 방법을 알아보았다.

## Out-of-Memory 에러는 언제 발생할까?
OOM 에러는 [[자바 코드의 메모리 영역(스택&힙)|자바의 메모리 구조]]와 밀접한 관련이 있다. 
JVM은 `Heap, Stack, Metaspace` 등으로 나뉘어 메모리를 관리하는데 이 중 특정 영역이 고갈되면 문제가 생긴다. 
각각의 상황을 예제와 함께 살펴보자.


### 1. Heap 메모리가 부족한 경우
Heap 영역은 객체를 저장하는 공간이다. 프로그램이 너무 많은 객체를 생성하거나 메모리를 효율적으로 관리하지 못하면 고갈된다. 예를 들어, 아래와 같은 코드가 있을 때:

```java
public class HeapOOM {
    public static void main(String[] args) {
        List<String> list = new ArrayList<>();
        while (true) {
            list.add("가자아아아!");
        }
    }
}
```
위 코드는 끝없이 문자열을 `ArrayList`에 추가한다. 
결국 메모리가 부족해져 `java.lang.OutOfMemoryError: Java heap space` 에러가 발생한다. 즉, 대규모 데이터를 처리하거나 컬렉션 객체를 잘못 사용했을 때 흔히 나타난다.


### 2. GC Overhead Limit 초과
[[가비지 컬렉터(GC) feat.힙 영역|GC(Garbage Collector)]]가 메모리를 정리하려고 애써도 공간이 회수되지 않으면 OOM 에러 중  `GC overhead limit exceeded`가 발생한다. 이는 메모리가 부족해 GC가 대부분의 시간을 소모하고, 프로그램이 정상적으로 실행되지 않을 때 나타난다.
정확히는 GC가 수행되어 새로 확보된 메모리가 전체 메모리의 2%미만이면 발생한다.

예를 들어, 아래처럼 대량의 객체를 반복적으로 생성하면 GC가 이를 처리하지 못할 수 있다.

```java
public class GCOverheadOOM {
    public static void main(String[] args) {
        HashMap<Integer, String> map = new HashMap<>();
        for (int i = 0; i < Integer.MAX_VALUE; i++) {
            map.put(i, "Data " + i);
        }
    }
}
```
이 경우 JVM이 계속 GC만 실행하고도 메모리를 확보하지 못하니 OOM이 발생한다.


### 3. Metaspace 부족
Java 8부터는 PermGen 영역이 Metaspace로 대체되었는데, 이 공간에는 클래스 메타데이터가 저장된다. 동적으로 클래스를 많이 로드하거나 리플렉션을 남발하면 Metaspace가 부족해질 수 있다.

```java
public class MetaspaceOOM {
    public static void main(String[] args) throws Exception {
        while (true) {
            ClassPool pool = ClassPool.getDefault();
            pool.makeClass("Class" + System.nanoTime()).toClass();
        }
    }
}
```
위 코드는 무한히 새로운 클래스를 생성해 Metaspace를 고갈시킨다. 
이는 동적 클래스 로딩을 사용하는 프레임워크(Spring, Hibernate 등)에서 잘못된 설계로 인해 발생할 수 있다.


### 4. Direct Memory 부족
NIO 기반 애플리케이션은 Heap이 아닌 Direct Memory를 사용하는데, 이를 과도하게 할당하면 문제가 생긴다. JVM 옵션으로 설정한 Direct Memory 크기를 초과하면 에러가 발생한다.

```java
public class DirectMemoryOOM {
    public static void main(String[] args) {
        while (true) {
            ByteBuffer.allocateDirect(1024 * 1024);
        }
    }
}
```
이 코드는 Direct Memory를 계속 할당하며, `java.lang.OutOfMemoryError: Direct buffer memory`를 유발한다.



## OOM 에러를 예방하는 방법
OOM은 코드와 환경의 문제를 개선하면 충분히 예방할 수 있다. 아래는 실제로 적용 가능한 해결책들이다.

### 1. JVM 메모리 크기 조정
첫 번째 방법으로는 JVM의 메모리 크기를 적절히 설정하는 것이다. 
예를 들어 Heap 메모리가 부족한 경우 다음과 같이 JVM 옵션을 추가해 메모리를 늘릴 수 있다.

```bash
java -Xms512m -Xmx4g MyApp
```
여기서 `-Xms`는 초기 Heap 크기, `-Xmx`는 최대 Heap 크기를 의미한다. 

필요하다면 Metaspace나 Direct Memory 크기도 `-XX:MaxMetaspaceSize`나 `-XX:MaxDirectMemorySize` 옵션으로 조정할 수 있다.


### 2. 메모리 누수 방지
메모리 누수는 OOM의 주요 원인 중 하나다. 
예를 들어 사용하지 않는 객체를 계속 참조하고 있다면 GC가 이를 회수하지 못한다. 이를 해결하려면 다음을 염두에 두자

- 컬렉션에서 사용 후 객체를 제거한다.
- 전역 변수로 객체를 참조하지 않도록 한다.

또한, 도구를 사용해 메모리 누수를 점검할 수 있다. **Eclipse Memory Analyzer (MAT)나 VisualVM**을 활용하면 메모리 누수가 발생한 객체를 쉽게 파악할 수 있다.


### 3. 데이터 처리 최적화
대규모 데이터를 처리할 때는 한 번에 메모리에 모두 로드하지 않고, 스트리밍 방식으로 처리하자. 
예를 들어, 아래 코드는 파일을 한 줄씩 읽어 메모리 사용을 최소화한다.

```java
public class FileReaderExample {
    public static void main(String[] args) throws Exception {
        try (BufferedReader reader = new BufferedReader(new FileReader("largeFile.txt"))) {
            String line;
            while ((line = reader.readLine()) != null) {
                // 한 줄씩 처리
                System.out.println(line);
            }
        }
    }
}
```


### 4. GC 튜닝과 모니터링
GC를 효과적으로 활용하기 위해 GC 옵션을 설정하고, 로그를 활성화해 문제를 분석할 수 있다.

단적인 예시로 각 애플리케이션에 최적화된 GC로 세팅해볼 수 있다.
```bash
java -Xlog:gc -XX:+UseG1GC MyApp
```

G1GC는 대규모 Heap을 효과적으로 관리하며, 현대 애플리케이션에 적합하다. 
또한, **Prometheus**나 **Grafana** 같은 도구로 JVM 메모리 사용량을 모니터링하면 OOM 발생 가능성을 사전에 파악할 수 있다.






참고
[Out Of Memory 오류](https://www.nextree.co.kr/p3878/)