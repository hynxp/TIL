## @Autowired를 사용하는 방식
`@Autowired`는 Spring Context에서 관리하는 실제 Bean을 테스트에 주입한다. 즉, 테스트에서 **Spring Boot 애플리케이션의 실제 구성 요소**를 사용한다.

### 특징
#### Spring Context 필요
`@SpringBootTest`를 통해 전체 Spring Application Context를 로드한다.

#### 실제 동작 검증
애플리케이션이 실제 실행될 때와 동일한 동작을 확인한다.

#### 통합 테스트 수행
데이터베이스 연동, JPA 동작, Bean 간의 상호작용을 포함한 테스트를 수행한다.


### 장점
#### 1. 실제 환경에서 동작을 확인할 수 있다
데이터베이스와 Repository가 제대로 연동되는지 확인한다.

#### 2. Spring Configuration 검증이 가능하다
Bean 주입과 설정이 정상적으로 작동하는지 확인할 수 있다.


### 단점
#### 1. 테스트 속도가 느리다
Spring Context를 로드하므로 초기화 시간이 길어진다.

#### 2. 테스트 간 독립성이 떨어질 수 있다
데이터베이스 상태가 공유되거나, 초기화되지 않으면 데이터 충돌이 발생할 가능성이 있다.



## Mock을 사용하는 방식
`Mock`은 테스트에서 실제 객체 대신 **가짜 객체**를 사용하여 의존성을 대체하는 방식이다. 주로 **Mockito**와 같은 라이브러리를 사용한다.

### 특징
#### Spring Context 불필요
테스트에서 Spring Boot 애플리케이션 컨텍스트를 로드하지 않는다.

#### 가짜 동작을 시뮬레이션한다
Repository, 외부 서비스 등의 의존성을 Mock으로 대체해 특정 동작을 검증한다.

#### 단위 테스트를 수행한다
특정 클래스나 메서드의 동작만 독립적으로 확인할 수 있다.


### 장점
#### 1. 테스트 속도가 빠르다
실제 데이터베이스나 네트워크 호출이 없으므로 테스트 실행이 빠르다.

#### 2. 독립적인 테스트 작성이 가능하다
외부 의존성을 제거하여 테스트를 독립적으로 실행할 수 있다.

#### 3. 예외 처리 시뮬레이션이 용이하다
Mock 객체를 통해 특정 상황(예외 발생)을 쉽게 재현할 수 있다.


### 단점
#### 1. 실제 동작을 확인할 수 없다
Repository와 JPA의 실제 동작을 검증할 수 없다.

#### 2. Mock 설정이 필요하다
Mock 객체의 동작을 미리 정의해야 하므로 설정 과정이 복잡해질 수 있다.



## 3. 두 방식의 코드 비교

### 1) `@Autowired`를 사용하는 테스트 코드
```java
@SpringBootTest
@Transactional
class MemberServiceTest {

    @Autowired
    private MemberService memberService;

    @Autowired
    private MemberRepository memberRepository;

    @Test
    @DisplayName("신규 사용자 저장 테스트")
    void saveMember() {
        Member member = new Member(3L, SocialType.KAKAO, "신영만", "https://user3.com");

        MemberResponse memberResponse = memberService.saveOrUpdate(member);

        Optional<Member> savedMember = memberRepository.findById(memberResponse.getId());
        assertThat(savedMember).isPresent();
    }
}
```

### 2) `Mock`을 사용하는 테스트 코드
```java
@ExtendWith(MockitoExtension.class)
class MemberServiceTest {

    @Mock
    private MemberRepository memberRepository;

    @InjectMocks
    private MemberService memberService;

    @Test
    @DisplayName("신규 사용자 저장 테스트")
    void saveMember() {
        Member member = new Member(3L, SocialType.KAKAO, "신영만", "https://user3.com");

        Mockito.when(memberRepository.save(member)).thenReturn(member);

        MemberResponse memberResponse = memberService.saveOrUpdate(member);

        Mockito.verify(memberRepository).save(member);
        assertThat(memberResponse.getId()).isEqualTo(3L);
    }
}
```


## 주요 차이점

|**특징**|**`@Autowired` 사용**|**`Mock` 사용**|
|---|---|---|
|**의존성 관리**|Spring Context에서 실제 Bean을 주입한다|Mockito를 사용해 가짜 객체(Mock)를 주입한다|
|**테스트 범위**|통합 테스트 (Spring Configuration, DB 연동 포함)|단위 테스트 (특정 클래스/메서드의 동작만 검증)|
|**속도**|느리다 (Spring Context 초기화 필요)|빠르다 (Context 불필요, DB 호출 없음)|
|**외부 의존성 필요 여부**|필요하다 (데이터베이스, 외부 API 등)|필요 없다 (모든 의존성을 Mock으로 대체 가능)|
|**실제 동작 확인 여부**|가능하다 (JPA, 트랜잭션, Bean 동작 검증)|불가능하다 (Mock 동작만 검증)|
|**테스트 작성 복잡도**|낮다 (Spring Boot 기본 설정 활용)|높다 (Mock 동작을 미리 정의해야 함)|


## 5. 언제 무엇을 사용해야 하는가?

#### `@Autowired`를 사용하는 경우
- **통합 테스트**를 수행할 때 적합하다.
- Spring Context, JPA, 트랜잭션, 데이터베이스 연동 등을 포함한 실제 동작을 검증하고자 할 때 사용한다.

#### `Mock`을 사용하는 경우
- **단위 테스트**를 수행할 때 적합하다.
- 특정 클래스(예: `MemberService`)의 로직만 테스트하고자 할 때.
- 데이터베이스나 외부 API 호출을 제거하여 독립적인 테스트를 작성하고자 할 때 사용한다.