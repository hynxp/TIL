즐겨보는 개발 오픈채팅에서 아래와 같은 질문이 올라왔다.

> "스프링에서 redis 를 사용해보는중인데 @Cacheable 을 내부 메서드에서 사용하게 되면 캐싱이 안되네요. 해결책으로 보통 어떤식으로 사용하나요..방법은 여러가지라고는 하는데 주로 어떤식으로 하
> 는지 궁금합니다!"

방장님께서는 **트랜잭션 원리**와 **Selfinvoke**를 알면 왜 그런지 알 수 있다고 하셔서 찾아보았다.

결론적으로는 Spring에서 `@Cacheable` 어노테이션을 사용하면 결과를 캐싱해 성능을 높일 수 있다. 
하지만 이 어노테이션을 **같은 클래스 안에서 다른 메서드를 호출하며 사용할 경우**, 예상과 다르게 **캐시가 적용되지 않는 상황**이 발생한다.

왜 그런지 알아보자.


## 캐시가 적용되지 않는 예시
먼저 다음 코드를 보자.
```java
@Service
public class MyService {

    public void outerMethod() {
        System.out.println("outerMethod 실행");
        innerMethod(); // 내부 메서드 호출
    }

    @Cacheable("myCache")
    public String innerMethod() {
        System.out.println("innerMethod 실행");
        return "결과";
    }
}
```
`outerMethod()`를 여러 번 호출해도 `innerMethod()`가 매번 실행된다.  
즉, `@Cacheable`이 적용되지 않는다.

**🤔 왜 그럴까?**

## 프록시 기반 AOP 구조 이해하기
스프링의 `@Cacheable`은 , 프록시(Proxy)라는 기술을 기반으로 동작한다.

### 프록시란?
스프링은 `@Cacheable`이 붙은 메서드를 **대리 객체(프록시 객체)**로 감싸서 대신 실행시킨다.  
프록시는 다음과 같은 일을 한다.

1. 메서드 실행 전에 캐시 확인
2. 캐시에 값이 있으면 메서드 실행 없이 바로 반환
3. 값이 없으면 메서드 실행 후 결과를 캐시에 저장

즉, **스프링이 만든 프록시 객체를 통해 메서드가 호출될 때만** 캐시 기능이 작동한다.


## Self-invocation이란?
`self-invocation`은 **자기 자신의 메서드를 자기 자신이 호출하는 것**을 말한다.

```java
public void outerMethod() {
    innerMethod(); // 이게 self-invocation
}
```
이처럼 같은 클래스 안에서 직접 호출하면 **프록시를 거치지 않고**, `this.innerMethod()`가 실행된다.

이 때는 프록시가 끼어들 수 없기 때문에, `@Cacheable`이 붙어 있어도 캐시는 전혀 작동하지 않는다.

## 그래서 왜 안 되는가?

|상황|캐시 동작 여부|설명|
|---|---|---|
|외부에서 `@Cacheable` 메서드를 호출|✅ 작동함|프록시를 거치기 때문|
|같은 클래스 안에서 직접 호출|❌ 안 작동함|프록시를 거치지 않고 `this`로 호출됨|


## 해결 방법은?
### 1. 캐시 메서드를 다른 클래스로 분리하기
가장 흔하고 안정적인 방법은 **`@Cacheable`이 붙은 메서드를 다른 클래스(빈)으로 분리**하는 것이다.
이 방식이 가장 명확하고, 스프링 문서나 실무에서도 가장 많이 추천된다.
```java
@Service
public class CacheService {
    @Cacheable("myCache")
    public String getCachedData(String key) {
        System.out.println("캐시 안에 없음 → 실행");
        return "데이터:" + key;
    }
}

@Service
public class MyService {

    private final CacheService cacheService;

    public MyService(CacheService cacheService) {
        this.cacheService = cacheService;
    }

    public void doSomething() {
        String data = cacheService.getCachedData("abc"); // 외부 호출이므로 캐시 적용됨
        System.out.println("받은 데이터: " + data);
    }
}
```
`CacheService`는 별도 클래스로 분리되어 있으므로, `getCachedData()` 호출 시 **스프링이 자동으로 프록시 객체를 사용하게 된다.**  
결과적으로 캐시가 잘 적용된다.

#### 왜 이 방식은 프록시를 타게 되는가?
스프링은 애플리케이션 실행 시 `@Service`, `@Component` 같은 클래스를 스캔하고,  
그 안에 `@Cacheable`, `@Transactional` 등의 어노테이션이 있으면 **프록시 객체를 만들어 스프링 빈으로 등록**한다.

`MyService`가 `CacheService`를 DI 받아 호출하는 구조에서는  `CacheService`가 스프링 컨테이너에 등록된 **프록시 객체**이기 때문에,  
메서드 호출 시 **스프링이 프록시를 통해 캐시 처리를 가로챌 수 있게 되는 것**이다.


### 2. ApplicationContext를 이용해 자기 자신을 프록시로 불러오기
조금 트릭이지만, 자기 자신의 프록시 객체를 ApplicationContext에서 직접 가져와 호출할 수도 있다.

```java
@Component
public class MyService implements ApplicationContextAware {

    private ApplicationContext context;

    @Override
    public void setApplicationContext(ApplicationContext ctx) {
        this.context = ctx;
    }

    public void outerMethod() {
        MyService proxy = context.getBean(MyService.class);
        proxy.innerMethod(); // 프록시 객체로 호출 → 캐시 적용됨
    }

    @Cacheable("myCache")
    public String innerMethod() {
        System.out.println("캐시 없음 → 실행");
        return "결과";
    }
}
```
이 방식은 스프링 내부 구조를 잘 알고 있어야 하기 때문에 **권장되진 않지만**, 내부에서 반드시 호출해야 할 경우 한 가지 대안이 될 수 있다.


