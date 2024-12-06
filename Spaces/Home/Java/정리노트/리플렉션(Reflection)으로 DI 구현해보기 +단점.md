자바의신을 읽던 중 리플렉션 파트가 나왔다.
특히 스프링 프레임워크의 DI(Dependency Injection)나 어노테이션 처리에 사용되고, 자사 라이브러리를 만들 때 사용한다고 한다.
아직 한번도 사용해본 적은 없어서 DI 구현을 통해 이해해보고자 한다!


## @TestAutowired 구현하기
리플렉션의 활용을 이해하기 위해 간단한 `@TestAutowired` 어노테이션을 만들어보자. 이 어노테이션은 클래스의 필드에 붙여 의존성을 자동으로 주입하는 기능을 한다.

### 1. @TestAutowired 어노테이션 정의
먼저 `@TestAutowired` 어노테이션을 정의한다.
이 어노테이션은 필드에만 사용할 수 있고, 런타임에도 유지되어야 하므로 `@Retention`과 `@Target`을 설정한다.
```java
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)  
public @interface TestAutowired {  
```
특정 클래스의 멤버필드에 `@TestAutowired` 어노테이션이 붙어있을 경우 해당 리플렉션을 통해 객체를 생성한다.


### 2. 의존성을 주입하는 클래스 작성
그리고 해당 어노테이션이 붙은 필드를 주입하는 클래스를 작성한다.
```java
public class DependencyInjector {
    public static void injectDependencies(Object target) {
        Class<?> clazz = target.getClass();
        
        for (Field field : clazz.getDeclaredFields()) {
            if (field.isAnnotationPresent(AutoWired.class)) {
                field.setAccessible(true); // private 필드 접근 허용
                try {
                    Object dependency = field.getType().newInstance();
                    field.set(target, dependency); // 객체를 필드에 주입
                } catch (Exception e) {
                    throw new RuntimeException("Dependency injection failed: " + e.getMessage(), e);
                }
            }
        }
    }
}
```


### 3. 테스트 클래스 작성
`Service`와 `Controller` 클래스를 만들어 실제로 DI(Dependency Injection)가 작동하는지 확인한다.
```java
class Service {  
    public void serve() {  
        System.out.println("DI가 됐어유");  
    }  
}  
  
class Controller {  
    @AutoWired  
    private Service service;  
  
    public void handleRequest() {  
        service.serve();  
    }  
}  
  
public class Main {  
    public static void main(String[] args) {  
        Controller controller = new Controller();  
  
        DependencyInjector.injectDependencies(controller);  
  
        controller.handleRequest();  
    }  
```


### 실행 결과
```
DI가 됐어유
```
결과를 보면 `Controller` 클래스에 `Service` 클래스가 정상적으로 주입되어 `serve()`메서드가 실행된 것을 확인할 수 있다.


## 리플렉션의 단점
하지만 리플렉션의 단점도 분명한데, 대표적으로 2가지 단점이 있다.

### 일반 메서드 보다 성능이 떨어진다.
리플렉션은 일반적인 메서드 호출보다 성능이 느리다.
런타임에 동적으로 클래스를 탐색하고 메서드를 호출하기 때문이다.
즉, 해당 클래스의 타입이 맞는지, 생성자가 존재하는지 등의 validation(유효성 검증) 과정을 **런타임 시 처리해야하기 때문에 성능이 떨어**진다.

다음 코드는 리플렉션 호출과 일반적인 메서드 호출의 성능을 비교하는 예제이다.
```java
public class ReflectionTest {  
    public void normalMethod() {  
        // 단순 메서드 호출  
    }  
  
    public static void main(String[] args) throws Exception {  
        ReflectionTest test = new ReflectionTest();  
  
        long start = System.nanoTime();  
        for (int i = 0; i < 1_000_000; i++) {  
            test.normalMethod();  
        }  
        long end = System.nanoTime();  
        System.out.println("Normal method time: " + (end - start) + " ns");  
  
        // Reflection 호출  
        start = System.nanoTime();  
        for (int i = 0; i < 1_000_000; i++) {  
            ReflectionTest.class.getMethod("normalMethod").invoke(test);  
        }  
        end = System.nanoTime();  
        System.out.println("Reflection method time: " + (end - start) + " ns");  
    }  
```

실행 결과 리플렉션 호출이 일반 메서드 호출보다 훨씬 느리다는 것을 알 수 있다.
```
Normal method time: 2437500 ns
Reflection method time: 63187083 ns
```

따라서 성능이 중요한 코드에서는 리플렉션을 남용하지 않아야 한다.


### 안전성 문제
또한 리플렉션은 프로그램의 안정성을 저하시킬 수 있다. 
왜냐하면 개발자가 의도치 않은 방식으로 내부 구조에 접근할 수 있기 때문에, 예상치 못한 에러가 발생할 수 있다.





참고
[Reflection / 개념 / 예제 / 단점 / DI 프레임워크 구현](https://tlatmsrud.tistory.com/112)