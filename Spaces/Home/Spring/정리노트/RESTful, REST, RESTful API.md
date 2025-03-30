## REST란?
REST(Representational State Transfer)는 웹의 기존 기술과 HTTP 프로토콜을 그대로 활용하는 아키텍처 스타일이다. 자원을 이름으로 구분하여 해당 자원의 상태를 주고받는 방식을 따르며, 웹 서비스에서 API를 설계할 때 많이 사용된다.

## REST의 구성 요소
REST는 크게 **자원(Resource)**, **행위(Verb)**, **표현(Representation)** 세 가지 요소로 이루어진다.

|구성 요소|설명|
|---|---|
|자원(Resource)|URI(Uniform Resource Identifier)를 통해 접근하는 데이터|
|행위(Verb)|HTTP 메서드(GET, POST, PUT, DELETE 등)를 사용하여 자원 조작|
|표현(Representation)|JSON, XML 등의 형태로 데이터를 전달|

## REST의 원칙
RESTful한 시스템을 만들기 위해서는 다음의 원칙을 준수해야 한다.

### 1. 클라이언트-서버 구조
클라이언트와 서버가 서로 독립적으로 동작하며, 클라이언트는 서버에 요청을 보내고 응답을 받는다. 
클라이언트는 UI와 사용자 경험을 담당하고, 서버는 비즈니스 로직과 데이터 처리를 담당한다.

### 2. 무상태(Stateless)
서버는 클라이언트의 요청을 처리할 때 이전 상태를 저장하지 않는다. 
즉, 각 요청은 독립적이며, 필요한 모든 정보를 포함해야 한다. 이를 통해 확장성이 뛰어나고 서버의 부담이 줄어든다.

### 3. 캐시 가능(Cacheable)
서버의 응답은 캐시가 가능해야 한다. 이를 통해 불필요한 요청을 줄이고 성능을 향상시킬 수 있다. HTTP 헤더를 이용해 캐싱 가능한 리소스를 명시할 수 있다.

### 4. 계층화된 시스템(Layered System)
클라이언트는 서버와 직접 통신할 수도 있고, 프록시, 로드 밸런서, 게이트웨이 등을 거쳐 데이터를 받을 수도 있다. 이러한 계층 구조는 보안, 로드 밸런싱, 확장성에 유리하다.

### 5. 일관된 인터페이스(Uniform Interface)
API의 설계가 일관되어야 하며, 아래의 원칙을 따라야 한다.

- **자원의 식별**: URI를 통해 자원을 명확히 식별해야 한다.
- **표현과 자원의 분리**: 서버는 동일한 자원에 대해 JSON, XML 등 다양한 표현을 제공할 수 있어야 한다.
- **Self-descriptive 메시지**: 요청 자체만으로도 충분한 정보를 포함해야 한다.
- **HATEOAS(Hypermedia as the Engine of Application State)**: API 응답에 관련 링크를 포함하여 클라이언트가 이를 탐색할 수 있도록 해야 한다.

### 6. 코드 온 디맨드 (선택적)
필요한 경우, 서버에서 클라이언트로 실행 가능한 코드를 전송하여 동작할 수도 있다(JavaScript 코드 등). 하지만 대부분의 RESTful API에서는 사용되지 않는다.


## RESTful API 설계
RESTful API를 설계할 때는 자원 중심의 URL을 사용하고, HTTP 메서드를 적절히 활용해야 한다.

| HTTP 메서드 | 사용 목적    | 예제                |
| -------- | -------- | ----------------- |
| GET      | 자원 조회    | `GET /users/1`    |
| POST     | 자원 생성    | `POST /users`     |
| PUT      | 자원 전체 수정 | `PUT /users/1`    |
| PATCH    | 자원 일부 수정 | `PATCH /users/1`  |
| DELETE   | 자원 삭제    | `DELETE /users/1` |

### RESTful URL 설계 예제
좋은 RESTful API 설계는 직관적이며 이해하기 쉬운 URL을 사용해야 한다.

| 잘못된 예                  | 올바른 예             |
| ---------------------- | ----------------- |
| `GET /getUser?id=1`    | `GET /users/1`    |
| `POST /createUser`     | `POST /users`     |
| `PUT /updateUser/1`    | `PUT /users/1`    |
| `DELETE /removeUser/1` | `DELETE /users/1` |


## RESTful API 예제 (Java Spring Boot)
Spring Boot를 사용하여 간단한 RESTful API를 구현할 수 있다.
```java
@RestController
@RequestMapping("/users")
public class UserController {

    private final Map<Long, String> users = new HashMap<>();

    @GetMapping("/{id}")
    public ResponseEntity<String> getUser(@PathVariable Long id) {
        return users.containsKey(id) ? 
            ResponseEntity.ok(users.get(id)) : 
            ResponseEntity.notFound().build();
    }

    @PostMapping
    public ResponseEntity<String> createUser(@RequestParam String name) {
        long id = users.size() + 1;
        users.put(id, name);
        return ResponseEntity.status(HttpStatus.CREATED).body("User created with ID: " + id);
    }

    @PutMapping("/{id}")
    public ResponseEntity<String> updateUser(@PathVariable Long id, @RequestParam String name) {
        if (users.containsKey(id)) {
            users.put(id, name);
            return ResponseEntity.ok("User updated");
        }
        return ResponseEntity.notFound().build();
    }

    @DeleteMapping("/{id}")
    public ResponseEntity<String> deleteUser(@PathVariable Long id) {
        if (users.containsKey(id)) {
            users.remove(id);
            return ResponseEntity.ok("User deleted");
        }
        return ResponseEntity.notFound().build();
    }
}
```



## RESTful API의 장점과 단점

### 장점
- HTTP 기반으로 작동하여 클라이언트와 서버가 독립적으로 개발될 수 있음
- URL을 통해 직관적인 API 설계가 가능
- 확장성이 뛰어나며, 다양한 플랫폼과 호환 가능
- JSON, XML 등의 다양한 응답 포맷을 지원

### 단점
- 상태를 저장하지 않기 때문에 인증, 보안 처리가 다소 복잡함
- 오버페칭(Over-fetching)과 언더페칭(Under-fetching) 이슈 발생 가능
- API 설계에 대한 명확한 가이드가 필요