## 서블릿이란?
[[17. 서블릿]]

## DispatcherServlet이란?
DispatcherServlet은 서블릿의 일종으로, Spring MVC의 핵심 요소이다.
서블릿 컨테이너의 가장 앞단에서 HTTP 프로토콜로 들어오는 모든 요청을 먼저 받아 적합한 컨트롤러에 위임해주는 프론트 컨트롤러이다.

### 프론트 컨트롤러란?
프론트 컨트롤러(Front Controller)란 서블릿 컨테이너의 제일 앞에서 서버로 들어오는 클라이언트의 모든 요청을 받아서 처리해주는 컨트롤러이다.

### 장점
과거에는 모든 서블릿을 URL 매핑을 위해 web.xml에 등록해야 했지만, 이제는 DispatcherServlet이애플리케이션으로 들어오는 모든 요청을 핸들링해주고 공통 작업을 처리하면서 상당히 편해졌다.


## DispatcherServlet의 상속 구조
DispatcherServlet은 Servlet의 일종이다. 다음과 같은 상속 구조를 가진다.

```java
public class DispatcherServlet extends FrameworkServlet {
}

public abstract class FrameworkServlet extends HttpServletBean {
}

public abstract class HttpServletBean extends HttpServlet {
}
```
DispatcherServlet은 **HttpServlet**을 최종적으로 상속받으므로 Servlet의 모든 기능을 활용할 수 있다. 또한 "Dispatcher"라는 이름처럼 요청을 받아 적절한 컨트롤러(Handler)로 "배분"하는 역할을 한다.


## 동작 과정
클라이언트에서 웹 요청을 했을 때, DispatcherServlet에서 이루어지는 동작 흐름은 아래와 같다.
![[Pasted image 20250112015434.png]]

1. DispatcherServlet으로 클라이언트의 요청이 들어온다.(중간에 필터를 거친다)
2. 웹 요청을 `HandlerMapping`에 위임하여 해당 요청을 처리할 Handler(Controller)를 탐색한다.
3. 찾은 Handler를 실행할 수 있는 `HandlerAdapter`를 탐색한다.
4. 찾은 HandlerAdapter를 사용해서 Handler의 메소드를 실행한다. 
5. 이때, Handler의 반환값은 `Model`과 `View`이다.
6. View 이름을 `ViewResolver`에게 전달하고,
7. ViewResolver는 해당하는 View 객체를 반환한다.
8. DispatcherServlet은 `View`에게 Model을 전달하고 화면 표시를 요청한다. 이때, Model이 null이면 View를 그대로 사용한다. 반면, 값이 있으면 View에 Model 데이터를 렌더링한다.
9. 최종적으로 DispatcherServlet은 `View 결과(HttpServletResponse)`를 클라이언트에게 반환한다.

해당 흐름은 @Controller를 기준으로 설명했다. @RestController의 경우 6, 7 과정이 생략된다. 
즉, ViewResolver를 타지 않고 반환값에 알맞는 MessageConverter를 찾아 응답 본문을 작성한다.

[과정 자세히 이해하기 좋은 글](https://mangkyu.tistory.com/18)






참고
[DispatcherServlet - Part 1](https://tecoble.techcourse.co.kr/post/2021-06-25-dispatcherservlet-part-1/)
[DispatcherServlet - Part 2](https://tecoble.techcourse.co.kr/post/2021-07-15-dispatcherservlet-part-2/)
[디스패처 서블릿이란? (Dispatcher Servlet)](https://mozzi-devlog.tistory.com/8)
