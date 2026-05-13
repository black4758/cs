# Spring MVC REST 요청 흐름

## 1. REST API 기준 요청 흐름

Spring MVC에서 REST API 요청은 보통 다음 흐름으로 처리된다.

```text
Client
→ DispatcherServlet
→ HandlerMapping
→ HandlerAdapter
→ RestController
→ Service
→ Repository
→ HttpMessageConverter
→ JSON Response
```

## 2. 각 과정 설명

### 1. Client

브라우저, 프론트엔드 애플리케이션, 모바일 앱, Postman 같은 클라이언트가 서버에 HTTP 요청을 보낸다.

예시:

```http
GET /members
```

### 2. DispatcherServlet

Spring MVC의 프론트 컨트롤러이다.

클라이언트의 모든 요청을 가장 먼저 받아서, 요청을 처리할 수 있는 컨트롤러를 찾는 과정으로 넘긴다.

면접 답변:

> DispatcherServlet은 Spring MVC에서 모든 요청을 가장 먼저 받는 프론트 컨트롤러입니다. 요청을 직접 처리하기보다는 적절한 컨트롤러를 찾아 실행하도록 전체 흐름을 제어합니다.

### 3. HandlerMapping

요청 URL과 HTTP Method에 맞는 컨트롤러 메서드를 찾아준다.

예를 들어 아래 요청이 들어오면:

```http
GET /members
```

다음 메서드를 찾는다.

```java
@GetMapping("/members")
public List<Member> members() {
    return memberService.findAll();
}
```

면접 답변:

> HandlerMapping은 요청 URL과 HTTP Method를 기준으로 어떤 컨트롤러 메서드가 실행되어야 하는지 찾아주는 역할을 합니다.

### 4. HandlerAdapter

HandlerMapping이 찾은 컨트롤러 메서드를 실제로 실행해준다.

Spring MVC에는 다양한 형태의 핸들러가 있을 수 있기 때문에, HandlerAdapter가 중간에서 실행 방식을 맞춰준다.

면접에서는 깊게 설명하지 않아도 되지만, 요청 흐름을 자세히 말할 때 넣으면 좋다.

면접 답변:

> HandlerAdapter는 HandlerMapping이 찾은 컨트롤러 메서드를 실제로 실행해주는 어댑터 역할을 합니다.

### 5. RestController

요청을 받아 Service를 호출하고, 처리 결과를 반환한다.

`@RestController`는 `@Controller`와 `@ResponseBody`를 합친 어노테이션이다.

```text
@RestController = @Controller + @ResponseBody
```

예시:

```java
@RestController
public class MemberController {

    private final MemberService memberService;

    public MemberController(MemberService memberService) {
        this.memberService = memberService;
    }

    @GetMapping("/members")
    public List<Member> members() {
        return memberService.findAll();
    }
}
```

면접 답변:

> RestController는 REST API 요청을 처리하는 컨트롤러입니다. Service를 호출해서 비즈니스 로직을 처리하고, 반환값을 HTTP Response Body에 담아 응답합니다.

### 6. Service

비즈니스 로직을 처리하는 계층이다.

예를 들어 회원 조회, 회원 가입, 주문 생성, 결제 처리 같은 핵심 로직을 담당한다.

예시:

```java
@Service
public class MemberService {

    private final MemberRepository memberRepository;

    public MemberService(MemberRepository memberRepository) {
        this.memberRepository = memberRepository;
    }

    public List<Member> findAll() {
        return memberRepository.findAll();
    }
}
```

면접 답변:

> Service는 비즈니스 로직을 처리하는 계층입니다. Controller가 요청을 받으면 Service를 호출하고, Service는 필요한 로직을 수행한 뒤 Repository를 통해 데이터에 접근합니다.

### 7. Repository

데이터베이스에 접근하는 계층이다.

DB에서 데이터를 조회, 저장, 수정, 삭제하는 역할을 담당한다.

예시:

```java
@Repository
public interface MemberRepository extends JpaRepository<Member, Long> {
}
```

면접 답변:

> Repository는 데이터 접근 계층입니다. DB와 관련된 조회, 저장, 수정, 삭제 작업을 담당합니다.

### 8. HttpMessageConverter

Controller가 반환한 객체를 HTTP 응답 body에 들어갈 형태로 변환한다.

REST API에서는 보통 Java 객체를 JSON으로 변환한다.

예를 들어 컨트롤러가 아래 객체를 반환하면:

```java
new Member("kim")
```

응답은 보통 JSON으로 변환된다.

```json
{
  "name": "kim"
}
```

면접 답변:

> HttpMessageConverter는 Controller의 반환 객체를 JSON 같은 HTTP 메시지 형태로 변환하는 역할을 합니다. REST API에서는 ViewResolver 대신 HttpMessageConverter가 사용되어 객체를 JSON으로 변환해 응답합니다.

### 9. JSON Response

최종적으로 클라이언트에게 JSON 형태의 HTTP 응답이 반환된다.

예시:

```json
[
  {
    "id": 1,
    "name": "kim"
  },
  {
    "id": 2,
    "name": "lee"
  }
]
```

## 3. MVC 방식과 REST 방식의 차이

### MVC View 방식

```text
Client
→ DispatcherServlet
→ HandlerMapping
→ Controller
→ Service
→ Repository
→ Model
→ ViewResolver
→ View
→ HTML Response
```

MVC View 방식은 서버가 HTML 화면을 만들어서 응답한다.

### REST API 방식

```text
Client
→ DispatcherServlet
→ HandlerMapping
→ RestController
→ Service
→ Repository
→ HttpMessageConverter
→ JSON Response
```

REST API 방식은 서버가 화면을 만들지 않고 JSON 데이터를 응답한다.

## 4. 한 번에 외우는 면접 답변

> Spring MVC에서 REST API 요청이 들어오면 먼저 DispatcherServlet이 요청을 받습니다. DispatcherServlet은 HandlerMapping을 통해 요청 URL과 HTTP Method에 맞는 컨트롤러 메서드를 찾고, HandlerAdapter가 해당 메서드를 실행합니다. RestController는 Service를 호출해 비즈니스 로직을 처리하고, Service는 Repository를 통해 데이터에 접근합니다. 이후 Controller가 반환한 객체는 HttpMessageConverter를 통해 JSON으로 변환되어 HTTP Response Body에 담겨 클라이언트에게 응답됩니다.

짧게 외우기:

```text
요청
→ DispatcherServlet
→ HandlerMapping
→ RestController
→ Service
→ Repository
→ HttpMessageConverter
→ JSON 응답
```
