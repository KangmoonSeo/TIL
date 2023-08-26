## 스프링 부트와 AWS로 혼자 구현하는 웹 서비스


### 패키지명 이름 규약
- 일반적으로 패키지명은 웹사이트 주소의 역순으로 함
    - 예시. 도메인 네임 `admin.jojoldu.com`
        - 패키지명 `com.jojoldu.admin`
        

### Request, Response에서의 Entity 사용
- Entity 클래스를 Request/Response 클래스로 직접 사용하면 안 됨
    - 데이터베이스와 맞닿은 핵심 클래스기 때문
    - Entity의 스펙이 변경되었을 때 api 사용자, 공급자 양 쪽 모두 곤란함
    - 대신에, **Dto**를 Request로 받아내고, Long을 return하는 등 코드를 변경함

### DTO
- Data Transfer Object, **계층 간 데이터 교환을 위한 객체**
- Request와 Response만을 위한 클래스, Dto를 변경해도 Entity 스펙에 영향을 끼치지 않음
- 번외. Dtos
    - Dto들이 존재하는 영역을 의미함


### MockMvc, TestRestTemplate 비교

1. `MockMvc` 테스트 환경
    - **컨트롤러 단위 테스트**
    ```java
    @WebMvcTest(controllers = HelloController.class)
    class HelloControllerTest {
        @Autowired private MockMvc mvc;
        // @MockBean private HelloService helloService;
        @Test
        void 테스트() throws Exception {
            // given
            String hello = "hello";

            // when
            ResultActions resultActions = mvc.perform(get("/hello"));

            // then
            resultActions
                    .andExpect(status().isOk())
                    .andExpect(content().string(hello));
        }
    }
    ```
    - 특징은 다음과 같음
        1. `@WebMvcTest(HelloController.class)`
            - **컨트롤러 단위 테스트**임
            - 컨트롤러 레이어만 슬라이스 테스트 할 수 있음
                - 단위 테스트를 진행할 컨트롤러 클래스를 지정할 수 있음
                    - 지정하지 않으면 모든 컨트롤러를 스캔함
        2. `@Autowired private MockMvc mvc`
            - url에 대한 perform 메서드로 요청을 보내고, `ResultActions`로 응답을 받아 검증함
        3. `@MockBean private HelloService helloSerivce`
            - 서비스 컨테이너는 `@MockBean` 애노테이션을 붙여 호출 가능
        4. `ResultActions.andExpect` 메서드로 검증
            - `andExpect` 메서드의 인수로 `ResultMatcher` 객체를 넣어야 함
                - 예시. `status().isOk()`, `content().string("text")`, `content().contentType("text/plain;charset=UTF-8")`
            - `Assertions.assertThrows` 에 대응하는 메서드는 없으므로, 직접 Exception을 핸들링함 
2. `TestRestTemplate` 테스트 환경
    - **스프링 통합 테스트**
    ```java
    @SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
    class HelloControllerTest {
        @LocalServerPort private int port;
        @Autowired private TestRestTemplate restTemplate; 

        @Test
        void 테스트() {
            // given
            String hello = "hello";
            String url = "http://localhost:" + port + "/hello";
            
            // when
            ResponseEntity<String> responseEntity = restTemplate.getForEntity(url, String.class);

            // then
            assertThat(responseEntity.getStatusCode()).isEqualTo(HttpStatus.OK);
            assertThat(responseEntity.getBody()).isEqualTo(hello);
        }

    }
    ```
    - 특징은 다음과 같음
        1. `@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)`
            - **스프링 통합 테스트**임
            - 단위 테스트와 비교했을 때 
        2. `@LocalServerPort private int port`
            - 포트를 지정해야 함, `@SpringBootTest`의 `webEnvironment` 속성과 연동
        3. `@Autowired private TestRestTemplate restTemplate`
            - `restTemplate`를 거쳐 request를 보내고, response 받아옴
        4. `Assertions.assertThat` 스태틱 메서드로 검증
            - `assertThat` 메서드로 테스트의 가시성을 높일 수 있음 
            - Exception이 throw 되지 않음


