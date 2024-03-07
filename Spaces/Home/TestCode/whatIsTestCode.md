# 테스트 코드

**먼저 짚고 갈 것은 TDD와 단위 테스트는 다르다.**

- TDD
    - 테스트가 주도하는 개발을 이야기 하는 것
    - 테스트 코드를 먼저 작성하는 것부터 시작
  

- 단위 테스트
    - TDD의 첫 번째 단계인 기능 단위의 테스트 코드를 작성하는 것
    - TDD와 달리 테스트 코드를 꼭 먼저 작성해야 하는것도 아니고, 리팩토링도 포함되지 않음
    - 순수하게 테스트 코드만 작성하는 것

<br>

## 💡테스트 코드는 왜 작성하는가

나도 현재 api 관련 개발할 때

1. 서버 올리고
2. 포스트맨으로 http 요청하고
3. 결과보고
4. 고치고

위 과정을 반복했다. 이게 다 테스트코드가 없어서 직접 확인해야 하기 때문이다.

추가로 기존에 잘되던 기능에 문제가 있는지 테스트가 가능하다.

나도 신입 때 회원가입 필수 값에 생년월일을 뺐더니 생년월일이 필수값인 기능들에서 오류가 줄줄이 터졌었던 기억이…

<br>

## 💡테스트 코드 작성을 도와주는 프레임워크

- JUnit - Java
- DBUnit - DB
- CppUnit - C++
- NUnit - .net

<br>

## 💡예제
```java
@RunWith(SpringRunner.class)
@WebMvcTest(controllers = HelloController.class)
public class HelloControllerTest {

    @Autowired
    private MockMvc mvc;

    @Test
    public void hello가_리턴된다() throws Exception {
        String hello = "hello";

        mvc.perform(get("/hello"))
                .andExpect(status().isOk())
                .andExpect(content().string(hello));
    }

    @Test
    public void helloDto가_리턴된다() throws Exception {
        String name = "hello";
        int amount = 1000;

        mvc.perform(
                    get("/hello/dto")
                            .param("name", name)
                            .param("amount", String.valueOf(amount)))
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.name", is(name)))
                .andExpect(jsonPath("$.amount", is(amount)));
    }
}
```

- **@RunWith(SpringRunner.class)**
    - 테스트를 진행할 때 JUnit에 내장된 실행자 외에 다른 실행자를 실행시킴
    - 여기서는 SpringRunner라는 스프링 실행자를 사용
    - 즉, 스프링 부트 테스트와  JUnit 사이에 연결자 역할

- **@WebMvcTest**
    - 여러 스프링 테스트 어노테이션 중, Web(Spring MVC)에 집중할 수 있는 어노테이션
    - @Controller, @ControllerAdvice 등을 사용
    - @Service, @Repository, @Repository 등은 사용할 수 없음

- **MockMvc**
    - 웹 API를 테스트할 때 사용
    - 이 클래스를 통해 HTTP GET, POST 등에 대한 API 테스트가 가능
        - mvc.perform(get(”/hello”))

- **.andExpect()**
    - mvc.perform의 결과를 검증
    - status().isOk()
        - HTTP Header의 Status를 검증
    - content().string(hello)
        - 응답 본문의 내용을 검증

- **jsonPath**
    - JSON 응답값을 필드별로 검증할 수 있는 메소드
    - $를 기준으로 필드명을 명시한다.