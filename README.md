# spring-web-application

## 1. Spring Initializer
Spring boot 을 사용해 프로젝트의 설정 및 생성.
- Gradle: Dependency 와 library
- Dependency: Spring Web, Thymeleaf(template engine)


## 2. 웹 기초- 백엔드 구동 방식
1. 정적 컨텐츠: src/resources/static 에서 html 을 그대로 띄워줌.
2. MVC 와 Template engine
    * Model: 값을 전달하는 매개체.
    * View: 받은 값들을 처리해 템플릿 엔진을 통해 HTML 로 변환(렌더) 후 브라우저에 반환.
    * Controller: 요청에 대한 로직을 검색해 있으면 해당 key, value 값을 model 에 담아서 viewResolver 에 넘겨줌.
3. API: 주로 어플리케이션에서 사용하는데, Json 형태로 값 return. \
   @ResponseBody 를 붙여서 값을 return 해서 'view' 를 통한 템플릿 엔진을 거치지 않고, (html 형식이 아닌)데이터 그대로 전송해줌.
   이 때, HttpMessageConverter 가 String 이면 그대로, 객체(Object)면 Json 으로 변환해서 반환.

웹 브라우저에서 localhost:8080/hello.html 와 같은 url 요청이 들어오면, 내장된 톰켓 서버가 우선 스프링 컨테이너로 요청 전달. \
스프링 컨테이너에 관련된 컨트롤러가 존재하면 해당 컨트롤러(MVC) 실행. \
없을 시엔, resources 패키지의 static/hello.html 스태틱 파일 반환.
   

## 3. 예제- 회원 관리 서비스 백엔드 개발
### 1. 비즈니스 요구사항 정리
* 데이터: ID, 이름
* 기능: 등록, 조회
* 아직 데이터 저장소가 선정되지 않음.
   #### * 웹 애플리케이션의 구조
   - 컨트롤러: 웹 MVC 의 컨트롤러.
   - 도메인: 비즈니스 도메인 객체. Ex) 회원, 주문, 쿠폰 등등 데이터베이스에 저장하고 관리됨.
   - 서비스: 도메인 객체의 핵심 비즈니스 로직.
   - 리포지토리: 데이터베이스에 접근해 도메인 객체를 저장하고 관리.
   #### * 클래스 의존관계
    데이터 저장소가 선정되지 않아 리포지토리를 interface 로 구현. 
    초기엔 빠르게 개발이 목적이기에 메모리 기반의 가벼운 데이터 저장소 사용.
    추후에 RDB, NoSQL 등이 선정되면 메모리 구현체 갈아끼우는 형식.
  
### 2. 회원 도메인과 레포지토리 생성
main 하위 패키지로 domain, repository 생성.
- 도메인: class Member \
  -> Long id, String name, getter, setter
- 레포지토리: interface MemberRepository, class MemoryMemberRepository \
  -> Member save, Optional<Member> findById(id), Optional<Member> findByName(name), List<Member> findAll(), clearStore()

### 3. 회원 레포지토리 테스트
test 하위 패키지로 repository 생성. 
1. 레포지토리에 구현한 함수들에 대한 테스트 케이스를 각각 작성.
2. 테스트 도구 중 하나인, Assertj 의 Assertions.assertThat 을 사용해 actual 이 expected 와 같은지 테스트.
3. 이 때, 전역 변수인 repository 가 각 테스트 케이스에서 서로 영향을 끼치면 안되기 때문에, @afterEach 로 매 테스트 후 실행 될 함수를 생성.

### 4. 회원 서비스 개발
main 하위 패키지로 service 생성.

실제 비즈니스가 돌아가는 로직에 따라 함수를 작성. \
주석으로 내용 설명하는 것이 바람직. \
Ex) 회원 가입: join(), 회원 조회: findMember()

### 5. 회원 서비스 테스트
test 하위 패키지로 service 생성.
1. 서비스에 구현한 함수들에 대한 테스트 케이스 작성.
2. 함수명은 기획자가 봤을 때도 어떤 상황인지 명확한 게 중요하기에, 한글로 해도 됨.
3. 맞는 상황 뿐 아니라, 예외 처리해주는 상황도 테스트 해야 함.
    Ex) assertThrows(Exception, lambda func), assertThat(message).isEqualTo()
4. 이 때, 전역 변수인 MemberService 에 선언되어 있는 MemberR Repository 가 같은 것임을 보장해주기 위해 DI(Dependency Injection)를 적용해줌.
5. 또한, @beforeEach 로 매 테스트 전에 실행될 함수를 생성해, 여기에서 새로운 repository 와 service 객체를 생성해줌.
'
## 4. 의존관계(DI)- 스프링 '빈'
### 1. 컴포넌트 스캔과 자동 의존관계 설정
- 서버가 구동될 때 '@Component' annotation 이 적힌 클래스는 스프링 컨테이너 안에 스프링 빈으로 등록되어 스프링이 관리한다.
- @Controller, @Service, @Repository 는 @Component 를 포함하기 때문에, 각 컨트롤러, 서비스, 레포지토리 클래스에 적어준다. 
- 컨트롤러 클래스에서 서비스를 선언, 서비스 클래스에서 레포지토리를 선언할 때 생성자(generator) 만들고 @Autowired 를 적어주면 위 세 가지 빈을 연결해준다. \
 -> DI(Dependency Injection): 의존관계 주입. \
  필드 주입, setter 주입, 생성자 주입 3가지 중 생성자 주입을 추천! \
  의존관계가 실행되고 나면 동적으로 변경해줘야 하는 경우가 없기 때문.
### 2. 코드로 직접 스프링 빈 등록
- SpringConfig 클래스를 프로젝트 하위에 만들고, '@Configuration' annotation 을 적어준다.
- 서비스, 레포지토리를 선언하고, '@Bean' annotation 을 적어준다.
- 컨트롤러는 1번의 컴포넌트 스캔 방식으로 유지한다. 

이 때, 정형화된 '컨트롤러', '서비스', '레포지토리' 같은 코드는 컴포넌트 스캔을 주로 사용. \
상황에 따라 구현 클래스를 변경해야 하는 경우엔 SpringConfig 로 스프링 빈 등록하는 방법 사용.

## 5. 웹 MVC 개발
### 1. 홈 화면
1. 첫 페이지(localhost:8080/)를 위해 resources 의 templates 패키지에 home.html 생성.
2. HomeController 생성해 @GetMapping("/")추가, return "home".

스프링 컨테이너에 컨트롤러가 있기 때문에, static 파일을 띄우지 않고 templates 의 해당 html 파일을 렌더링해 띄워줌.

### 2. 등록
1. resources/templates 에 members 디렉토리 생성후, createMemberForm.html 생성. \
   여기에 form 태그(action="/members/new" method="post")로 입력을 받을 수 있는 형태로 만들어줌.
2. MemberController 에 @GetMapping("/members/new"), return "members/createMemberForm"
3. MemberForm 클래스에 name 변수를 갖게 하나 만들고, \
   MemberController 에 @PostMapping("/members/new"), MemberForm 인자로 받고 이름을 Member 생성 후 입력. \
   MemberService 에 해당 Member 을 join 해주고, return "redirect:/"으로 홈 화면으로 돌아옴.

### 3. 조회
1. resources/templates 의 members 에 memberList.html 생성. \
   'table' 태그로 목록을 보여주는 형식. 'thread' 태그와 'tbody' 태그로 타임리프 엔진이 입력된 Model member 의 id 와 name 을 갖고오도록 해줌.
2. MemberController 에 @GetMapping("/members"), Model 을 인자로 받아 memberService 의 리스트를 "members" name 으로 addAttribute 해줌. return "members/memberList"

## 6. Database 접근
H2 데이터베이스 사용. \
순수 Jdbc -> JdbcTemplate -> JPA -> 데이터 JPA 단계로 발전하는 과정을 직접 경험해봄.

### 1. Jdbc
스프링의 장점이 바로, 인터페이스를 통한 다형성을 의존성 주입을 통해 스프링 컨테이너가 쉽게 제공해준다는 점이다. \
개방-폐쇄 원칙(OCP, Open-Closed Principle) 
 -> memoryRepository 를 jdbcRepository 로 구현체만 변경하고, SpringConfig 에 입력해주면 됨.

<b>테스트</b> \
-> 테스트의 경우엔 DI를 필드 주입으로 해줘도 된다. 새로 인자를 추가해줄 경우가 없기 때문. 따라서, @BeforeEach 를 해주지 않아도 됨. \
-> 테스트 케이스의 클래스에 @Transactional 어노테이션을 붙이면, 데이터베이스에 쿼리 날렸던 정보를 롤백해줌. 
따라서, @AfterEach 를 해줄 필요가 없음. \
-> @SpringBootTest 는 스프링 컨테이너와 테스트를 동시에 실행해줌. \
*실제 스프링을 띄워서 하는 통합 테스트 보다 , 각 함수마다 하는 단위 테스트가 더 좋은 경우가 많다.

### 2. JdbcTemplate


### 3. JPA

### 4. 데이터 JPA
