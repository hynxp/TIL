애플리케이션 테스트는 주로 **단위 테스트(Unit Test)**, **통합 테스트(Integration Test)**, 그리고 **E2E 테스트(End-to-End Test)**로 나눌 수 있다. 각각의 테스트 유형과 그 특징을 알아보자.


## 단위 테스트 (Unit Test)
단위 테스트는 애플리케이션의 가장 작은 단위인 메서드나 클래스 단위로 동작을 검증한다.

### 테스트 대상
특정 클래스나 메서드를 대상으로 작성한다. 예를 들어, 컨트롤러 계층의 테스트는 `@WebMvcTest`를 사용하며, JPA 레포지토리 테스트는 `@DataJpaTest`를 활용한다.

### 테스트 범위
유효성 검사, 경계값 처리 등 다양한 케이스를 다룬다. 이 과정에서 private 메서드는 직접 테스트하지 않는다. 대신 private 메서드를 호출하는 public 메서드를 통해 테스트 케이스를 검증하면 충분하다.

### 예제 코드
```java
@WebMvcTest(MyController.class)
class MyControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @Test
    void shouldReturnHello() throws Exception {
        mockMvc.perform(get("/hello"))
               .andExpect(status().isOk())
               .andExpect(content().string("Hello, World!"));
    }
}
```


## 통합 테스트 (Integration Test)
통합 테스트는 여러 계층이 함께 동작하는 시나리오를 검증한다.

### 사용 어노테이션
`@SpringBootTest`를 사용한다. 이 어노테이션은 애플리케이션의 전체 스프링 컨텍스트를 로드하므로, 단위 테스트보다 상대적으로 느리다.

### 적용 사례
상품 생성 후 수정 여부를 확인하는 로직처럼 여러 컴포넌트 간의 상호작용을 확인할 때 주로 사용된다. 외부 API 호출과 같은 복합적인 케이스도 통합 테스트에 포함된다.

### 속도와 효율성
`@WebMvcTest`는 웹 계층 관련 빈만 로드하여 속도가 빠르다. 반면 `@SpringBootTest`는 모든 빈을 로드하므로 테스트 속도가 느려지는 단점이 있다.

### 예제 코드
```java
@SpringBootTest
class ProductIntegrationTest {

    @Autowired
    private ProductService productService;

    @Test
    void shouldCreateAndUpdateProduct() {
        // 상품 생성
        Product product = productService.createProduct("New Product");

        // 상품 수정
        product.setName("Updated Product");
        Product updatedProduct = productService.updateProduct(product);

        // 검증
        assertEquals("Updated Product", updatedProduct.getName());
    }
}

```


## E2E 테스트 (End-to-End Test)
E2E 테스트는 사용자의 실제 시나리오를 모방해 애플리케이션의 전체 흐름을 테스트한다.

### 사용 도구
브라우저 자동화 도구(Selenium)나 API 테스트 도구를 활용할 수 있다. IntelliJ의 HTTP 클라이언트 기능도 API 기반의 E2E 테스트에 유용하다.

### 테스트 범위
애플리케이션의 UI, API, 데이터베이스 등 모든 계층을 포함한 전체적인 동작을 검증한다.


## 추가 팁 - JPA 설정
### JPA 설정 관리
JPA 사용 시, `@EnableJpaAuditing` 설정은 별도의 `JpaConfig` 클래스를 만들어 관리하는 것이 일반적이다. 이는 애플리케이션의 주요 설정과 분리하여 가독성과 유지보수를 높이기 위함이다.

```java
@Configuration
@EnableJpaAuditing
public class JpaConfig {
}
```