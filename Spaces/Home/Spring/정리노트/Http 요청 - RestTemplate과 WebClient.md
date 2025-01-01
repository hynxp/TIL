## RestTemplate
`RestTemplate`은 스프링에서 HTTP 요청을 처리하기 위해 가장 오래도록 사용된 클래스이다. 간단한 API 호출을 처리하는 데 유용하며 동기 방식으로 동작한다. 동기 방식이란 요청을 보내고 응답을 받을 때까지 쓰레드가 블로킹된다는 의미이다.

### 주요 메서드
- `getForObject(url, responseType)`: GET 요청을 보내고 객체를 응답받는다.
- `postForObject(url, request, responseType)`: POST 요청을 보내고 객체를 응답받는다.
- `exchange(url, method, requestEntity, responseType)`: 요청 메서드와 헤더, 요청 본문 등을 세부적으로 설정할 수 있다.

```java
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
```

### 장점
- 사용이 간단하고 직관적이다.
- 동기 방식이므로 흐름을 따라가기 쉽다.

### 단점
- 비동기 처리를 지원하지 않아 고성능이 필요한 애플리케이션에는 적합하지 않다.
- 스프링 5부터 점진적으로 사용이 줄어들고 있다.


## WebClient
`WebClient`는 스프링 5부터 등장한 HTTP 요청 처리를 위한 비동기 방식 도구이다. 리액티브 프로그래밍 모델을 기반으로 동작하며, 비동기 방식과 스트리밍 처리에 적합하다.

### 주요 메서드
- `get()`: GET 요청을 설정한다.
- `post()`: POST 요청을 설정한다.
- `uri(uri)`: 요청 URI를 설정한다.
- `retrieve()`: 요청을 전송하고 간단히 응답을 처리한다.
- `exchangeToMono()`와 `exchangeToFlux()`: 세부적인 요청 및 응답 설정을 처리한다.

```java
public KakaoTokenResponse getToken(String code) {
    WebClient webClient = WebClient.builder()
            .baseUrl(ACCESS_TOKEN_URL) // 기본 URL 설정
            .defaultHeader(HttpHeaders.CONTENT_TYPE, MediaType.APPLICATION_FORM_URLENCODED_VALUE)
            .defaultHeader(HttpHeaders.ACCEPT, MediaType.APPLICATION_JSON_VALUE)
            .build();

    MultiValueMap<String, String> params = new LinkedMultiValueMap<>();
    params.add("grant_type", "authorization_code");
    params.add("client_id", clientId);
    params.add("redirect_uri", redirectUri);
    params.add("code", code);

    KakaoTokenResponse response = webClient.post()
            .bodyValue(params) // 요청 본문 설정
            .retrieve() // 응답을 간단히 처리
            .bodyToMono(KakaoTokenResponse.class) // 응답을 Mono로 변환
            .block(); // 동기 방식으로 결과 반환

    return response;
}
```

### 장점
- 비동기 방식으로 동작해 고성능 시스템에 적합하다.
- 리액티브 스트림을 활용하여 대량의 데이터를 효율적으로 처리할 수 있다.
- 다양한 HTTP 요청을 세밀히 조정할 수 있다.

### 단점
- 초기 학습 곡선이 비교적 높다.
- 비동기 및 리액티브 방식에 익숙하지 않다면 디버깅이 어려울 수 있다.


## RestTemplate과 WebClient 비교

|특징|RestTemplate|WebClient|
|---|---|---|
|등장 시점|스프링 3|스프링 5|
|동기/비동기|동기|비동기 및 동기 모두 지원|
|리액티브 지원|미지원|지원|
|학습 난이도|낮음|다소 높음|
|권장 사용 시점|간단한 요청 처리|고성능 애플리케이션, 리액티브 환경|



## 어떤 것을 선택해야 할까?
- **간단한 요청 처리**: 동기 방식으로 동작하며 복잡하지 않은 HTTP 호출이라면 `RestTemplate`을 사용하는 것이 유효하다.
- **고성능 시스템**: 비동기 처리와 리액티브 스트림의 장점을 활용하려면 `WebClient`를 선택하는 것이 좋다.
- **새로운 프로젝트**: 스프링 5 이상에서는 `WebClient`를 사용하는 것이 권장된다.