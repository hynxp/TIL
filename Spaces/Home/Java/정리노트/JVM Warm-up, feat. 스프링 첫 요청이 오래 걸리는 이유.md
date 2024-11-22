
## 원인

웹 애플리케이션을 실행한 후 초기 요청이 지연되는 현상을 이해하려면, 자바 프로그램이 실행되는 방식을 깊이 살펴볼 필요가 있다.

![[IMG-20241122112027219.png]]
C, C++과 같은 컴파일 언어는 소스 코드를 기계어로 직접 변환하여 최적화된 성능을 제공하나 빌드 환경에 종속적이라는 단점이 있다. 즉, 플랫폼이 바뀌면 재컴파일이 필요하다.

자바는 이러한 플랫폼 종속적인 문제를 해결하고자 JVM을 도입하였다.
![[IMG-20241122112027345.png]]
자바는 자바 코드는 컴파일 시 바이트 코드로 변환되고, 실행 시 JVM이 이를 기계어로 변환하여 실행한다. 이러한 구조 덕분에 Java는 플랫폼에 종속되지 않게 되었지만, 이 과정에서 변환 작업이 추가되므로 성능에 영향을 미칠 수 있다.

### 1. 클래스 로더
JVM에서 클래스 로더(Class Loader)는 바이트 코드를 메모리에 로드하는 역할을 한다. 이때 클래스 로더는 일반적으로 **Lazy Loading** 방식을 채택한다. Lazy Loading이란 애플리케이션 실행 시 모든 클래스를 즉시 로드하지 않고, 특정 클래스가 필요해질 때 이를 로드하는 방식이다.

이 Lazy Loading이 첫번째 이유이다. **배포 직후에는 대부분의 클래스들이 한번도 사용되지 않았으므로 클래스 로더에 의해 메모리에 적재되지 않은 상태**이다. 따라서 첫 요청이 발생하면 클래스 로더는 그제서야 해당 클래스들을 메모리에 로드하는 작업을 수행하게 되고, 이 과정에서 지연이 발생하게 된다. 그재서야 클래스 로더가 부랴부랴 클래스를 메모리에 적재한다. 

### 2. JIT 컴파일러
바이트 코드를 기계어로 변환하는 작업은 JVM 성능에 중요한 영향을 미친다. JVM은 이를 해결하고자  [[JIT(Just-In-Time) 컴파일러]]를 도입하였다.

기존에는 인터프리터 방식을 통해 코드를 한줄 한줄 해석해 가면서 읽어 속도가 느리다는 단점이 있었다. 하지만 JIT 컴파일러는 **바이트 코드를 실행 중에 분석하고, 반복적으로 호출되는 코드(핫스팟)를 기계어로 변환하여 실행 속도를 최적화**한다.

JIT 컴파일러는 실행 중 프로파일링을 수행하여 코드의 실행 빈도, 메서드 호출 횟수, 루프 반복 횟수 등을 분석한다. 이를 통해 자주 사용되는 핫스팟 코드를 식별하고 이를 기계어로 변환한 후 **코드 캐시(Code Cache)**에 저장한다. 이후 동일한 코드 실행 시 캐시를 활용하여 성능을 극대화할 수 있다.

> 참고로 오라클에서는 JIT 컴파일을 Hotspot 이라고 부른다고 한다.


## JVM Warm-up 이란?

하지만 JIT 컴파일러를 사용하더라도 애플리케이션을 처음 시작했다면 아무런 코드도 실행되지 않았기 때문에 캐시에 아무것도 없다. 이때 매우 많은 요청이 한 번에 애플리케이션 서버로 들어오면 어떻게 될까? 
애플리케이션 초기 상태에서는 클래스 로딩이 완료되지 않았고, JIT 컴파일러에 의해 기계어로 변환된 코드도 없다. 이러한 상황에서 대량의 요청이 애플리케이션 서버로 들어오면 처리 속도가 급격히 느려질 수 있다. 이를 **Cold-Start**라고 부른다.

어떻게 해결하면 될까?
**자주 호출될 것으로 예상되는** 로직을 강제로 호출하여 기계어를 캐싱해 두는 작업을 하면 된다. 이를 `warm-up` 이라고 한다. 

## Warm-up 수행 방법
트래픽이 많은 서비스라면 `warm-up` 작업은 반드시 고려되어야 한다.
그래서 warm-up은 어떻게 할 수 있을까?
스프링 애플리케이션이 실행된 후에 핵심 로직들을 강제로 호출시켜 `warm-up` 하면 된다.

예를 들어 아래와 같은 방법을 활용해 볼 수 있다.

### 1.**주요 로직을 미리 호출**
Spring Boot에서는 `ApplicationRunner` 인터페이스를 활용하여 애플리케이션 초기화 시 특정 로직을 강제로 호출할 수 있다. 자주 사용될 메서드를 미리 실행하여 JVM의 Warm-up을 유도할 수 있다.

```java
@Component
public class WarmupTest implements ApplicationRunner {

    @Override
    public void run(final ApplicationArguments args) throws Exception {
        try {
            // 자주 사용되는 메서드 호출
        } catch (Exception e) {
            // ..
        }
    }
}
```

### 2. Java Microbenchmark Harness(JMH) 활용
보다 정교한 Warm-up이 필요할 경우, `JMH`를 활용할 수 있다. 
`JMH`는 JVM 워밍업을 자동화하기 위해 가장 널리 알려진 도구 중 하나로, JVM의 워밍업 상태를 모니터링하면서  특정 코드를 반복적으로 실행하여 최적화 주기를 유도할 수 있다.

```xml
<dependency>
    <groupId>org.openjdk.jmh</groupId>
    <artifactId>jmh-core</artifactId>
    <version>1.37</version>
</dependency>
<dependency>
    <groupId>org.openjdk.jmh</groupId>
    <artifactId>jmh-generator-annprocess</artifactId>
    <version>1.37</version>
</dependency>
```
위와 같이 의존성을 추가하고 JMH 메서드를 입력할 수 있다.
```java
@Benchmark
public void warmUpBenchmark() {
    // 반복 실행할 코드 작성
}
```


## Warm-up의 효과
Beihang University의 연구에 따르면, 대규모 데이터 처리 시스템(Hadoop, Spark)에서 JVM Warm-up이 적용되었을 때 처리 속도가 크게 개선되었다.
![[IMG-20241122112027733.png|300]]
`HotTub`은 JVM이 warm-up된 환경을 말하는데, 보다시피 속도 향상이 크게 발생한 것을 알 수 있다.
대규모 시스템도 이런데 비교적 작은 읽기 작업을 수행했을 때 차이는 훨씬 클 것으로 예상된다.😮



## Warm-up을 수행할 때 고려해 볼 트레이드오프
Warm-up 작업은 애플리케이션 성능을 최적화하는 데 유용하지만, 몇 가지 트레이드오프를 동반한다.

1. Warm-up 작업 중 오류가 발생하면 실제 기능은 정상적으로 작동하더라도 배포가 중단될 위험이 있다.  
2. 외부 API 호출을 포함한 Warm-up 작업은 외부 시스템에 대량의 요청을 발생시켜 DDoS공격과 비슷한 상황이 발생할 수 있다.  
3. 애플리케이션의 새로운 버전을 배포할 때 등의 애플리케이션 구동 시간이 늘어날 수 있다.





참고
[스프링 첫 요청이 처리되는데 오래 걸리는 이유(서블릿 초기화와 JIT 컴파일러)와 해결 방법(웜업, Warm-Up)](https://mangkyu.tistory.com/232)
[How to Warm Up the JVM - Baeldung](https://www.baeldung.com/java-jvm-warmup)
[스프링 애플리케이션 배포 직후 발생하는 Latency의 원인과 이를 해결하기 위한 JVM Warm-up](https://hudi.blog/jvm-warm-up/)
[JVM Warm-up 이란](https://junuuu.tistory.com/830)


