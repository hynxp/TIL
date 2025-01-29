## RestTemplate
`RestTemplate`은 스프링에서 HTTP 요청을 처리하기 위해 가장 오래 사용된 클래스이다. 간단한 API 호출을 처리하는 데 유용하며 동기 방식으로 동작한다.

사용법이 간단하고 직관적이며, 동기 방식이라 요청 흐름을 따라가기가 쉽다. 
하지만 스프링 5부터 점차 사용이 줄어들고 있다. 
처음에는 **RestTemplate이 deprecated될 것이며, WebClient로 대체된다**는 이야기가 있었지만, 스프링 팀은 **Maintenance Mode**로 유지한다고 공식 발표했다. 
이후 Spring Boot 3.1에서 `RestClient`가 등장하면서 이 말도 사라졌다고 한다.

다만, 비동기 처리를 지원하지 않아 고성능이 필요한 애플리케이션에는 적합하지 않다.


## OkHttpClient
Spring에 종속적이지 않은 HTTP 클라이언트 라이브러리로, 동기 및 비동기 처리 방식을 선택적으로 사용할 수 있다.
```
implementation 'com.squareup.okhttp3:okhttp'
```
위 의존성을 추가하여 사용할 수 있으며, `HttpURLConnection`이나 `HttpClient`보다 훨씬 직관적이다. 하지만 Spring Framework 6.1 및 Spring Boot 3.2부터 `OkHttp3`가 더 이상 지원되지 않는다. 
최신 스프링 환경을 고려하고 있다면 사용에 주의가 필요하다!


## WebClient
`WebClient`는 스프링 5에서 등장한 HTTP 요청 처리를 위한 비동기 방식의 클라이언트이다. 리액티브 프로그래밍 모델을 기반으로 동작하며, 비동기 요청 및 스트리밍 처리에 적합하다.

과거에는 `RestTemplate` 대안으로 많이 사용되었으나, **WebClient를 사용하려면 spring-webflux 의존성을 추가해야 한다**는 점이 단점으로 꼽혔다. 
리액티브 환경을 구축하지 않는 프로젝트에서는 다소 부담이 될 수 있다.


## RestClient
`RestClient`는 Spring Boot 3.1에서 도입된 새로운 HTTP 클라이언트로, `WebClient`의 단점을 보완하기 위해 등장했다. 서블릿 기반으로 설계되어 리액티브 환경 없이도 사용할 수 있다. 동기 및 비동기 요청을 모두 지원하며, 최신 Spring Boot 환경에 최적화되어 있다.
```java
RestClient restClient = RestClient.create(myCustomRestTemplate);
```
또한 기존에 사용하던 `RestTemplate`을 파라미터로 주입하여 쉽게 전환할 수 있다는 점이 큰 장점이다.

## 각 방식의 예시

### RestTemplate
기존에 나는 카카오 서버의 API를 사용해 토큰을 가져오는 방식으로 `RestTemplate`을 사용했다.
```java
@Component  
public class KakaoOAuthClient {  
  
    public KakaoTokenResponse getToken(String code) {  
        RestTemplate restTemplate = new RestTemplate();  
  
        HttpHeaders headers = new HttpHeaders();  
        headers.setContentType(MediaType.APPLICATION_FORM_URLENCODED);  
        headers.add(HttpHeaders.ACCEPT, MediaType.APPLICATION_JSON_VALUE);  
  
        MultiValueMap<String, String> params = new LinkedMultiValueMap<>();  
        params.add("grant_type", "authorization_code");  
        params.add("client_id", clientId);  
        params.add("redirect_uri", redirectUri);  
        params.add("code", code);  
  
        HttpEntity<MultiValueMap<String, String>> request = new HttpEntity<>(params, headers);  
        ResponseEntity<KakaoTokenResponse> response = restTemplate.exchange(  
                ACCESS_TOKEN_URL, HttpMethod.POST, request, KakaoTokenResponse.class  
        );  
  
        return response.getBody();  
    }  
}
```
토비님의 영상을 보면 성능적인 큰 차이는 없어서 `RestTemplate`를 계속 사용해도 무방하다고 한다.
하지만 체이닝 방식을 사용하면 좀 더 직관적일 것 같다.

이 코드 중 `getToken()`을 각 방식으로 바꿔서 비교해 보자.

### WebClient
```java
public KakaoTokenResponse getToken(String code) {
    return webClient.post()
            .uri(ACCESS_TOKEN_URL)
            .header(HttpHeaders.CONTENT_TYPE, MediaType.APPLICATION_FORM_URLENCODED_VALUE)
            .bodyValue(Map.of(
                    "grant_type", "authorization_code",
                    "client_id", clientId,
                    "redirect_uri", redirectUri,
                    "code", code
            ))
            .retrieve()
            .bodyToMono(KakaoTokenResponse.class)
            .block(); // 동기 처리
}
```
`bodyValue`를 활용해 요청 데이터를 간결하게 설정하고 체이닝 방식으로 코드를 직관적으로 작성할 수 있다.

### RestClient
```java
public KakaoTokenResponse getToken(String code) {
    return restClient.post()
            .uri(ACCESS_TOKEN_URL)
            .header(HttpHeaders.CONTENT_TYPE, MediaType.APPLICATION_FORM_URLENCODED_VALUE)
            .body(Map.of(
                    "grant_type", "authorization_code",
                    "client_id", clientId,
                    "redirect_uri", redirectUri,
                    "code", code
            ))
            .retrieve(KakaoTokenResponse.class);
}

```
`RestClient`는 `WebClient` 기반으로 동작하지만, 추가적인 의존성이 없으며 사용법이 더 간단하고 직관적이다. WebFlux 의존성을 피하고 싶은 경우 적합한 대안이다.


## 요약
- **RestTemplate**: 간단한 동기 요청, 기존 코드 유지가 필요할 때 적합하다.
- **WebClient**: 비동기 처리, 리액티브 프로그래밍이 필요한 환경에 적합하다.
- **RestClient**: 최신 스프링 환경에서 간단하고 직관적인 HTTP 요청 처리가 필요할 때 권장된다.

`WebFlux` 의존성 추가가 부담스럽다면, 최신 프로젝트에서는 `RestClient`를 사용하는 것이 더 나은 선택일 것 같다.
고로 나는 RestClient로 변경하기로 했다!




참고
[RestClient 알아보기 (RestTemplate이 Deprecated 된다고요?)](https://poalim.tistory.com/59)
[토비님 유튜브](https://youtu.be/S4W3cJOuLrU?si=jeJw2jpg7HU5dwzw)