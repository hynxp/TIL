## 🤔 Spring Boot Actuator란?
Spring Boot Actuator는 2014년 4월 스프링 부트가 릴리스될 때 함께 제공되었던 스프링의 모듈 중 하나로, 애플리케이션의 상태를 모니터링하고 관리할 수 있도록 다양한 엔드포인트를 제공하는 기능이다.

주로 헬스 체크, 메트릭, 환경 정보, 로깅, 트레이싱 같은 기능을 제공하며, 이를 통해 Spring Boot 애플리케이션을 모니터링하고 관리하는 데 도움을 준다.
HTTP 엔드 포인트 또는 JMX Bean을 사용하여 상호 작용할 수 있다.

## 언제 사용될까?
|기능|설명|
|---|---|
|**헬스 체크**|애플리케이션이 정상적으로 실행 중인지 확인|
|**메트릭**|CPU, 메모리 사용량, 요청 처리량 등 성능 정보 확인|
|**환경 정보**|애플리케이션의 실행 환경(환경 변수, 설정 값) 확인|
|**로깅**|로그 레벨을 조회 및 동적으로 변경|
|**트레이싱**|요청이 애플리케이션에서 어떻게 처리되는지 추적|

### 1. 헬스 체크 (Health Check)
헬스 체크는 **애플리케이션이 정상적으로 동작하는지 확인하는 기능**이다.

- 애플리케이션의 **CPU, 메모리, 요청 처리량** 등을 모니터링하려고 할 때
- 성능 튜닝을 위해 현재 시스템 리소스 사용량을 확인할 때
- 장애 발생 시, 어떤 리소스가 부족한지 확인할 때

### 2. 메트릭 (Metrics)
메트릭은 **애플리케이션의 성능과 상태를 측정하는 데이터**를 의미한다.

- 서버가 정상적으로 동작하는지 확인하려고 할 때
- 배포 후 애플리케이션이 문제없이 실행 중인지 체크할 때
- 로드 밸런서가 트래픽을 정상적인 서버로만 보내도록 설정할 때

### 3. 환경 정보 (Environment)
애플리케이션이 실행되는 **환경 변수, 설정 값** 등을 확인할 수 있다.

- 애플리케이션이 **어떤 환경에서 실행 중인지 확인할 때**
- **DB 연결 정보, 포트, 프로파일 (dev, prod) 등**을 확인하고 싶을 때
- **환경 변수 값을 동적으로 변경하고 싶을 때**

### 4. 로깅 (Logging)
로깅은 **애플리케이션에서 발생하는 이벤트(로그)를 기록하는 기능**이다.

- 애플리케이션이 **정상적으로 실행되는지 로그를 보고 싶을 때**
- 특정 클래스나 패키지의 **로그 레벨을 동적으로 변경하고 싶을 때**
- 장애 발생 시, **어떤 오류가 발생했는지 로그를 보고 분석할 때**



## 사용법
### 의존성 추가
Actuator를 사용하려면 `spring-boot-starter-actuator` 의존성을 추가해야 한다.

#### maven
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

#### gradle
```yml
implementation("org.springframework.boot:spring-boot-starter-actuator")
```


### 활성화
`application.properties` 또는 `application.yml`에서 Actuator를 활성화할 수 있다.
```properties
management.endpoints.web.exposure.include=*
```
위 설정을 추가하면 모든 Actuator 엔드포인트가 활성화된다.


## 엔드포인트 종류

| HTTP 메서드       | 경로                   | 설명                                                                 | 디폴트 활성화 |
|------------------|----------------------|------------------------------------------------------------------|-------------|
| GET             | /auditevents         | 호출된 감사(audit) 이벤트 리포트를 생성한다.                           | No          |
| GET             | /beans               | 스프링 애플리케이션 컨텍스트의 모든 빈을 알려준다.                        | No          |
| GET             | /conditions          | 성공 또는 실패했던 자동-구성 조건의 내역을 생성한다.                      | No          |
| GET             | /configprop          | 모든 구성 속성을 현재 값과 같이 알려준다.                              | No          |
| GET, POST, DELETE | /env              | 스프링 애플리케이션에 사용할 수 있는 모든 속성 근원과 이 근원들의 속성을 알려준다. | No          |
| GET             | /env/{toMatch}       | 특정 환경 속성의 값을 알려준다.                                   | No          |
| GET             | /health              | 애플리케이션의 건강 상태 정보를 반환한다.                           | Yes         |
| GET             | /heapdump            | 힙(heap) 덤프를 다운로드한다.                                   | No          |
| GET             | /httptrace           | 가장 최근의 100개 요청에 대한 추적 기록을 생성한다.                    | No          |
| GET             | /info                | 개발자가 정의한 애플리케이션에 관한 정보를 반환한다.                     | Yes         |
| GET             | /loggers             | 애플리케이션의 패키지 리스트(각 패키지의 로깅 레벨 포함)를 생성한다.         | No          |
| GET, POST       | /loggers/{name}      | 지정된 로거의 로깅 레벨(구성된 로깅 레벨과 유효 로깅 레벨 모두)을 반환한다. 유효 로깅 레벨은 HTTP POST 요청으로 설정될 수 있다. | No          |
| GET             | /mappings            | 모든 HTTP 매핑과 이 매핑들을 처리하는 핸들러 메서드들의 내역을 제공한다.   | No          |
| GET             | /metrics             | 모든 메트릭 리스트를 반환한다.                                   | No          |
| GET             | /metrics/{name}      | 지정된 메트릭의 값을 반환한다.                                | No          |
| GET             | /scheduledtasks      | 스케줄링된 모든 태스크의 내역을 제공한다.                          | No          |
| GET             | /threaddump          | 모든 애플리케이션 스레드의 내역을 반환한다.                        | No          |


## Spring Security로 보호
Spring Security를 사용하면 Actuator 엔드포인트에 인증 및 권한을 설정할 수 있다.
```java
@Configuration
public class SecurityConfig {

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/actuator/health", "/actuator/info").permitAll()  // 특정 엔드포인트는 공개
                .requestMatchers("/actuator/**").hasRole("ADMIN")  // 그 외는 ADMIN 권한 필요
            )
            .httpBasic();  // HTTP 기본 인증 활성화
        return http.build();
    }
}
```
위 설정을 적용하면 `/actuator/health`와 `/actuator/info`는 **모든 사용자 접근 가능**,  
그 외 `/actuator/**` 엔드포인트는 **ADMIN 역할을 가진 사용자만 접근 가능**하다.


## 커스텀 엔드포인트 만들기
또한 사용자 정의 엔드포인트를 만들수도 있다.
```java
@Component
@Endpoint(id = "custom")
public class CustomEndpoint {

    @ReadOperation
    public String customEndpoint() {
        return "사용자 정의 Actuator 엔드포인트";
    }
}
```
위 코드를 추가하면 `/actuator/custom` 엔드포인트가 생성된다.
- `@Endpoint(id = "custom")` → 엔드포인트의 ID를 `custom`으로 설정
- `@ReadOperation` → GET 요청을 처리하는 메서드


## Spring Boot Actuator을 사용해 모니터링 구축해보기
[[모니터링&부하테스트]]





참고
[스프링 부트 - 액추에이터](https://velog.io/@zenon8485/%EC%8A%A4%ED%94%84%EB%A7%81-%EB%B6%80%ED%8A%B8-%EC%95%A1%EC%B6%94%EC%97%90%EC%9D%B4%ED%84%B0)
[Spring Boot Actuator의 헬스체크 살펴보기](https://toss.tech/article/how-to-work-health-check-in-spring-boot-actuator)
[Spring Boot Actuator](https://www.baeldung.com/spring-boot-actuators)