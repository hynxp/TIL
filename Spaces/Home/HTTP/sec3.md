# **HTTP 기본**

## **HTTP**

: HyperText Transfer Protocol

이제는 HTTP 메시지에 모든 것을 전송한다.

HTML ,TExt, 이미지, 음성, 영상, 파일, JSON, XML 등..

### **TCP/UDP와 HTTP**

TCP기반 : HTTP/1.1, HTTP/2

UDP기반 : HTTP/3

HTTP 프로토콜을 사용하면 간접적으로 TCP를 사용하는게 된다.

TCP의 3 way handshake의 과정과 메커니즘이 복잡해서 최적화해서 나온게  HTTP/3이다.

![https://blog.kakaocdn.net/dn/bpkNQW/btsCjVBR3H2/zArK5rxVvq4ST6jDoMkU00/img.png](https://blog.kakaocdn.net/dn/bpkNQW/btsCjVBR3H2/zArK5rxVvq4ST6jDoMkU00/img.png)

개발자 도구에서 확인할 수 있다.

### **HTTP의 특징**

### **무상태 프로토콜(stateless)**

HTTP의 중요한 특징 중에 하나는 무상태 프로토콜을 지향한다는 것이다.

서버가 클라이언트의 상태를 유지하지않는다.

여기서 "상태(state)"란 클라이언트와 서버 간의 통신 상황, 정보를 말한다.

예를 들어 사용자의 로그인 "상태"를 유지하는 방법으로 쿠키나 세션을 사용할 수 있는데 이는 서버에서

사용자의 상태를 유지하고있다.

하지만 HTTP 프로토콜을 사용한 JWT 토큰이나, OAuth 프로토콜로 stateless하게 사용자의 로그인을 처리할 수 있다.

### **비 연결성(connectionless)**

클라이언트와 서버간의 연결을 유지하지 않고, 연결을 요청하고 응답을 받으면 연결을 즉시 끊는다.

서버 자원을 매우 효율적으로 사용할 수 있다.

**단점**

TCP/IP 연결을 새로 맺어야 하기 때문에 3 way handshake 과정의 시간이 추가된다.

ex) 다음 페이지 클릭 시 새로운 요청을 보내기 때문에

HTML 뿐만 아니라 자바스크립트, CSS, 이미지 등 수많은 자원이 매번 함께 다운로드 된다.

### **HTTP 메시지**

요청 메시지, 응답 메시지의 구조

![https://blog.kakaocdn.net/dn/cFcwcb/btsCjV2Z3wS/9M0m8SXkmiQ15XA8iEdZB1/img.png](https://blog.kakaocdn.net/dn/cFcwcb/btsCjV2Z3wS/9M0m8SXkmiQ15XA8iEdZB1/img.png)

### **요청 메시지**

```
GET /search?q=hello&hl=ko HTTP/1.1
Host: www.google.com
```

`GET`

HTTP 메서드로 서버가 수행해야 할 동작을 지정한다.

`/search?q=hello&hl=ko`

절대 경로로 표시된 요청 대상이다.

`HTTP/1.1`

HTTP의 버전이다.

### **응답 메시지**

```
HTTP/1.1 200 OK
Content-Type: text/html;charset=UTF-8
Content-Length: 3423
<html>
 <body>...</body>
</html>
```

HTTP 버전과 상태 코드, 데이터의 타입, 길이 등 많은 정보가 포함된다.

### **예시**

- 네이버 메인페이지의 헤더정보

![https://blog.kakaocdn.net/dn/b9xjtb/btsCiJaAZRU/4BkKdxzL0UEAkXeYM9UfKK/img.png](https://blog.kakaocdn.net/dn/b9xjtb/btsCiJaAZRU/4BkKdxzL0UEAkXeYM9UfKK/img.png)

API를 설계할 때는 리소스를 식별해야 한다.

리소스와 리소스를 어떻게 할건지 행위를 분리해야 한다.

이 분리를 위해 HTTP메서드를 사용한다.

GET

POST

PUT

PATCH

DELETE