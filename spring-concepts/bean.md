# Bean 정리

## 1. Bean이란?

Bean은 Spring 컨테이너가 생성하고 관리하는 객체를 의미한다.

일반 Java 객체는 개발자가 직접 `new` 키워드로 생성한다.

```java
OrderService orderService = new OrderService();
```

하지만 Spring에서는 객체를 개발자가 직접 생성하지 않고, Spring 컨테이너가 객체를 생성하고 관리한다.

```java
@Service
public class OrderService {
}
```

이처럼 Spring 컨테이너가 생성하고 관리하는 객체를 Bean이라고 한다.

면접 답변:

> Bean은 Spring 컨테이너가 생성하고 관리하는 객체입니다. 개발자가 직접 `new`로 객체를 생성하는 것이 아니라, Spring이 객체를 생성하고 의존성을 주입하며 생명주기까지 관리합니다.

정리:

- 객체: Java에서 생성된 인스턴스
- Bean: Spring 컨테이너가 관리하는 객체

즉, 모든 Bean은 객체지만 모든 객체가 Bean은 아니다.

## 2. Bean 생명주기

Bean 생명주기는 Spring 컨테이너가 Bean을 생성하고, 의존성을 주입하고, 초기화한 뒤, 사용하다가 종료 시점에 소멸시키는 흐름을 말한다.

Bean 생명주기 흐름:

```text
Spring 컨테이너 생성
→ Bean 객체 생성
→ 의존성 주입
→ 초기화 콜백
→ Bean 사용
→ 소멸 전 콜백
→ Spring 컨테이너 종료
```

각 단계 설명:

1. Spring 컨테이너 생성
   - Spring 애플리케이션이 실행되면서 컨테이너가 생성된다.

2. Bean 객체 생성
   - `@Component`, `@Service`, `@Bean` 등으로 등록된 Bean 객체를 생성한다.

3. 의존성 주입
   - 생성된 Bean에 필요한 의존 객체를 주입한다.

4. 초기화 콜백
   - Bean이 사용되기 전에 필요한 초기화 작업을 수행한다.
   - 예: `@PostConstruct`

5. Bean 사용
   - 애플리케이션에서 Bean을 사용한다.

6. 소멸 전 콜백
   - 컨테이너가 종료되기 전에 정리 작업을 수행한다.
   - 예: `@PreDestroy`

예시:

```java
@Component
public class MyBean {

    @PostConstruct
    public void init() {
        System.out.println("Bean 초기화");
    }

    @PreDestroy
    public void destroy() {
        System.out.println("Bean 소멸 전 정리");
    }
}
```

면접 답변:

> Bean 생명주기는 Spring 컨테이너가 Bean을 생성하고, 의존성을 주입하고, 초기화한 뒤 사용하다가, 컨테이너 종료 시점에 소멸시키는 과정입니다. 대표적으로 초기화 콜백에는 `@PostConstruct`, 소멸 전 콜백에는 `@PreDestroy`를 사용할 수 있습니다.

## 3. Bean 등록 어노테이션

Spring에서 Bean을 등록하는 대표적인 방법은 컴포넌트 스캔 방식과 수동 등록 방식이 있다.

컴포넌트 스캔 방식:

- `@Component`
- `@Service`
- `@Repository`
- `@Controller`
- `@RestController`

수동 등록 방식:

- `@Configuration` + `@Bean`

## 4. @Component

`@Component`는 가장 기본적인 Bean 등록 어노테이션이다.

특정 계층의 의미가 강하지 않은 일반적인 객체를 Bean으로 등록할 때 사용한다.

예시:

```java
@Component
public class EmailSender {
}
```

사용 예시:

- 공통 유틸 클래스
- 메일 발송 클래스
- 파일 업로드 클래스
- 외부 API 호출 클래스

면접 답변:

> `@Component`는 Spring Bean으로 등록하기 위한 가장 기본적인 어노테이션입니다. 특정 계층에 속하지 않는 일반적인 클래스를 Bean으로 등록할 때 사용합니다.

## 5. @Service

`@Service`는 비즈니스 로직을 처리하는 서비스 계층 클래스에 사용한다.

예시:

```java
@Service
public class OrderService {
}
```

사용 예시:

- 회원 가입 처리
- 주문 생성
- 결제 처리
- 게시글 작성

면접 답변:

> `@Service`는 비즈니스 로직을 담당하는 서비스 계층 클래스에 사용하는 어노테이션입니다. 내부적으로는 `@Component`를 포함하고 있어서 Spring Bean으로 등록됩니다.

## 6. @Repository

`@Repository`는 데이터 접근 계층 클래스에 사용한다.

주로 DB에 접근하는 DAO나 Repository 클래스에 붙인다.

예시:

```java
@Repository
public class OrderRepository {
}
```

사용 예시:

- 회원 조회
- 주문 저장
- 게시글 삭제
- DB CRUD 처리

특징:

- 내부적으로 `@Component`를 포함하고 있어 Bean으로 등록된다.
- DB 관련 예외를 Spring의 `DataAccessException`으로 변환해준다.

면접 답변:

> `@Repository`는 데이터 접근 계층에 사용하는 어노테이션입니다. DB와 관련된 CRUD 작업을 담당하는 클래스에 사용하며, DB 관련 예외를 Spring의 `DataAccessException`으로 변환해주는 기능도 있습니다.

## 7. @Controller

`@Controller`는 Spring MVC에서 웹 요청을 처리하는 클래스에 사용한다.

주로 View를 반환하는 컨트롤러에서 사용한다.

예시:

```java
@Controller
public class PageController {

    @GetMapping("/home")
    public String home() {
        return "home";
    }
}
```

여기서 `"home"`은 문자열 데이터가 아니라 `home.html` 같은 View 이름으로 해석된다.

면접 답변:

> `@Controller`는 Spring MVC에서 클라이언트 요청을 처리하는 컨트롤러 클래스에 사용하는 어노테이션입니다. 주로 View 이름을 반환해서 HTML 화면을 응답할 때 사용합니다.

## 8. @RestController

`@RestController`는 REST API를 만들 때 사용하는 컨트롤러 어노테이션이다.

`@RestController`는 `@Controller`와 `@ResponseBody`를 합친 것이다.

```text
@RestController = @Controller + @ResponseBody
```

예시:

```java
@RestController
public class UserApiController {

    @GetMapping("/users")
    public List<String> users() {
        return List.of("kim", "lee");
    }
}
```

반환값은 View 이름이 아니라 HTTP Response Body에 직접 들어간다. 보통 JSON 형태로 응답된다.

면접 답변:

> `@RestController`는 REST API 컨트롤러를 만들 때 사용하는 어노테이션입니다. `@Controller`와 `@ResponseBody`를 합친 것으로, 메서드 반환값이 View 이름이 아니라 HTTP Response Body에 직접 담깁니다.

## 9. @Configuration + @Bean

`@Configuration`과 `@Bean`은 Bean을 수동으로 등록할 때 사용한다.

`@Configuration`은 설정 클래스라는 의미이고, `@Bean`은 해당 메서드가 반환하는 객체를 Spring Bean으로 등록한다는 의미이다.

예시:

```java
@Configuration
public class AppConfig {

    @Bean
    public ObjectMapper objectMapper() {
        return new ObjectMapper();
    }
}
```

이 경우 `objectMapper()` 메서드가 반환하는 `ObjectMapper` 객체가 Spring Bean으로 등록된다.

사용 예시:

- 외부 라이브러리 객체를 Bean으로 등록할 때
- 객체 생성 과정을 직접 제어해야 할 때
- 생성자에 특별한 설정값을 넣어야 할 때
- 여러 구현체 중 하나를 선택해서 Bean으로 등록할 때

면접 답변:

> `@Configuration`과 `@Bean`은 개발자가 직접 생성 과정을 제어해서 Bean을 등록할 때 사용합니다. 주로 외부 라이브러리 객체처럼 클래스에 직접 `@Component`를 붙일 수 없거나, 복잡한 설정이 필요한 객체를 Bean으로 등록할 때 사용합니다.

## 10. 한 번에 외우는 면접 답변

> Bean은 Spring 컨테이너가 생성하고 관리하는 객체입니다. Spring은 Bean을 생성하고, 의존성을 주입하고, 초기화한 뒤 사용하다가 컨테이너 종료 시점에 소멸시킵니다. Bean은 `@Component`, `@Service`, `@Repository`, `@Controller`, `@RestController` 같은 어노테이션을 통해 자동 등록할 수 있고, `@Configuration`과 `@Bean`을 사용해 수동으로 등록할 수도 있습니다. `@Service`는 비즈니스 로직, `@Repository`는 DB 접근, `@Controller`는 View 반환, `@RestController`는 JSON API 응답에 사용합니다.
