# Exception Handling 정리

## 1. 예외 처리를 하는 이유

예외 처리는 애플리케이션에서 문제가 발생했을 때 클라이언트에게 일관된 에러 응답을 주기 위해 사용한다.

예를 들어 회원을 조회했는데 회원이 없거나, 권한이 없는 사용자가 수정 요청을 보낼 수 있다.

이때 Controller마다 `try-catch`를 작성하면 코드가 반복되고 응답 형식이 제각각이 될 수 있다.

그래서 Spring에서는 보통 전역 예외 처리 방식을 사용한다.

흐름:

```text
Controller 요청
→ Service 로직 처리
→ 예외 발생
→ GlobalExceptionHandler가 예외를 잡음
→ ErrorResponse로 변환
→ 클라이언트에게 JSON 응답
```

면접 답변:

> 예외 처리는 애플리케이션에서 발생한 오류를 일관된 형식으로 클라이언트에게 응답하기 위해 사용합니다. Spring에서는 각 Controller마다 try-catch를 작성하기보다 `@RestControllerAdvice`와 `@ExceptionHandler`를 사용해 전역으로 예외를 처리하는 방식을 많이 사용합니다.

## 2. 패키지 구조

예외 처리는 특정 도메인에만 속하지 않고 프로젝트 전체에서 공통으로 사용된다.

그래서 보통 `global/exception` 또는 `common/exception` 패키지에 둔다.

```text
src/main/java/com/example/demo
├── global
│   └── exception
│       ├── ErrorCode.java
│       ├── ErrorResponse.java
│       ├── CustomException.java
│       └── GlobalExceptionHandler.java
├── member
├── auth
└── config
```

## 3. ErrorCode.java

`ErrorCode`는 에러 종류별 코드, 메시지, HTTP 상태를 enum으로 관리하는 클래스이다.

```java
@Getter
@AllArgsConstructor
public enum ErrorCode {

    BAD_REQUEST(HttpStatus.BAD_REQUEST, "BAD_REQUEST", "잘못된 요청입니다."),
    UNAUTHORIZED(HttpStatus.UNAUTHORIZED, "UNAUTHORIZED", "인증이 필요합니다."),
    FORBIDDEN(HttpStatus.FORBIDDEN, "FORBIDDEN", "접근 권한이 없습니다."),
    NOT_FOUND(HttpStatus.NOT_FOUND, "NOT_FOUND", "리소스를 찾을 수 없습니다."),
    INTERNAL_SERVER_ERROR(HttpStatus.INTERNAL_SERVER_ERROR, "SERVER_ERROR", "서버 내부 오류입니다.");

    private final HttpStatus status;
    private final String code;
    private final String message;
}
```

설명:

- `status`: HTTP 상태 코드이다. 예: 400, 401, 404
- `code`: 클라이언트가 구분할 에러 코드이다.
- `message`: 기본 에러 메시지이다.

사용 예시:

```java
throw new CustomException(ErrorCode.NOT_FOUND);
```

## 4. ErrorResponse.java

`ErrorResponse`는 클라이언트에게 내려줄 공통 에러 응답 DTO이다.

```java
@Getter
@AllArgsConstructor
public class ErrorResponse {

    private String code;
    private String message;

    public static ErrorResponse from(ErrorCode errorCode) {
        return new ErrorResponse(
            errorCode.getCode(),
            errorCode.getMessage()
        );
    }

    public static ErrorResponse of(ErrorCode errorCode, String message) {
        return new ErrorResponse(
            errorCode.getCode(),
            message
        );
    }
}
```

응답 예시:

```json
{
  "code": "NOT_FOUND",
  "message": "리소스를 찾을 수 없습니다."
}
```

## 5. CustomException.java

`CustomException`은 비즈니스 예외를 표현하기 위해 직접 만든 예외 클래스이다.

```java
@Getter
public class CustomException extends RuntimeException {

    private final ErrorCode errorCode;

    public CustomException(ErrorCode errorCode) {
        super(errorCode.getMessage());
        this.errorCode = errorCode;
    }

    public CustomException(ErrorCode errorCode, String message) {
        super(message);
        this.errorCode = errorCode;
    }
}
```

설명:

- `RuntimeException`을 상속한다.
- `ErrorCode`를 필드로 가진다.
- 기본 메시지는 `ErrorCode`의 메시지를 사용한다.
- 상황별로 직접 메시지를 넣을 수도 있다.

사용 예시:

```java
throw new CustomException(ErrorCode.NOT_FOUND);
```

```java
throw new CustomException(ErrorCode.FORBIDDEN, "수정 권한이 없습니다.");
```

## 6. GlobalExceptionHandler.java

`GlobalExceptionHandler`는 프로젝트 전체에서 발생한 예외를 한 곳에서 처리하는 클래스이다.

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(CustomException.class)
    public ResponseEntity<ErrorResponse> handleCustomException(CustomException e) {
        ErrorCode errorCode = e.getErrorCode();

        return ResponseEntity
            .status(errorCode.getStatus())
            .body(ErrorResponse.of(errorCode, e.getMessage()));
    }

    @ExceptionHandler(IllegalArgumentException.class)
    public ResponseEntity<ErrorResponse> handleIllegalArgumentException(
        IllegalArgumentException e
    ) {
        return ResponseEntity
            .badRequest()
            .body(ErrorResponse.of(ErrorCode.BAD_REQUEST, e.getMessage()));
    }

    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleException(Exception e) {
        return ResponseEntity
            .status(HttpStatus.INTERNAL_SERVER_ERROR)
            .body(ErrorResponse.from(ErrorCode.INTERNAL_SERVER_ERROR));
    }
}
```

어노테이션 설명:

- `@RestControllerAdvice`: 모든 Controller에서 발생한 예외를 전역으로 처리한다.
- `@ExceptionHandler`: 특정 예외 타입을 처리하는 메서드를 지정한다.

예외 처리 메서드 역할:

```text
CustomException
→ 우리가 의도적으로 던지는 비즈니스 예외 처리

IllegalArgumentException
→ 잘못된 요청 같은 기본 예외 처리

Exception
→ 예상하지 못한 모든 서버 예외 처리
```

### CustomException handler

```java
@ExceptionHandler(CustomException.class)
```

직접 정의한 비즈니스 예외를 처리한다.

예를 들어 회원이 없거나, 권한이 없거나, 이미 가입된 이메일인 경우처럼 개발자가 의도적으로 던진 예외를 처리한다.

```java
throw new CustomException(ErrorCode.NOT_FOUND, "존재하지 않는 회원입니다.");
```

### IllegalArgumentException handler

```java
@ExceptionHandler(IllegalArgumentException.class)
```

잘못된 요청이나 잘못된 인자 때문에 발생한 기본 예외를 처리한다.

아직 `CustomException`으로 바꾸지 않은 간단한 예외도 공통 에러 응답으로 변환할 수 있다.

```java
throw new IllegalArgumentException("이미 존재하는 이메일입니다.");
```

### Exception handler

```java
@ExceptionHandler(Exception.class)
```

위에서 처리하지 못한 예상치 못한 모든 예외를 처리한다.

예를 들어 `NullPointerException`, `NoSuchElementException` 같은 서버 내부 오류를 마지막으로 잡아 공통 500 응답으로 변환한다.

주의:

`Exception.class`는 모든 예외를 잡기 때문에 가장 마지막에 두는 것이 좋다.

## 7. Service에서 사용하는 방식

Service에서는 에러 응답을 직접 만들기보다 예외를 던진다.

예시:

```java
@Service
@RequiredArgsConstructor
public class MemberService {

    private final MemberRepository memberRepository;

    public MemberResponse findMember(Long id) {
        Member member = memberRepository.findById(id)
            .orElseThrow(() -> new CustomException(ErrorCode.NOT_FOUND, "존재하지 않는 회원입니다."));

        return MemberResponse.from(member);
    }

    public void checkOwner(Long ownerId, Long loginUserId) {
        if (!ownerId.equals(loginUserId)) {
            throw new CustomException(ErrorCode.FORBIDDEN, "수정 권한이 없습니다.");
        }
    }
}
```

흐름:

```text
회원 없음
→ CustomException 발생
→ GlobalExceptionHandler가 잡음
→ ErrorResponse 생성
→ 404 응답
```

## 8. Controller는 어떻게 달라지나?

Controller에서는 예외 처리를 직접 하지 않고 Service를 호출한다.

```java
@RestController
@RequiredArgsConstructor
@RequestMapping("/members")
public class MemberController {

    private final MemberService memberService;

    @GetMapping("/{id}")
    public MemberResponse findMember(@PathVariable Long id) {
        return memberService.findMember(id);
    }
}
```

Controller에 `try-catch`를 반복해서 작성하지 않아도 된다.

나쁜 예:

```java
@GetMapping("/{id}")
public ResponseEntity<?> findMember(@PathVariable Long id) {
    try {
        return ResponseEntity.ok(memberService.findMember(id));
    } catch (CustomException e) {
        return ResponseEntity.status(404).body(e.getMessage());
    }
}
```

좋은 예:

```java
@GetMapping("/{id}")
public MemberResponse findMember(@PathVariable Long id) {
    return memberService.findMember(id);
}
```

예외 응답은 `GlobalExceptionHandler`가 담당한다.

## 9. Optional과 예외 처리

Repository 조회 결과는 없을 수 있으므로 `Optional`을 자주 사용한다.

```java
Optional<Member> findByEmail(String email);
```

값이 없을 때는 `orElseThrow()`로 예외를 던질 수 있다.

```java
Member member = memberRepository.findByEmail(email)
    .orElseThrow(() -> new CustomException(ErrorCode.NOT_FOUND, "존재하지 않는 회원입니다."));
```

`Optional.get()`은 값이 없으면 `NoSuchElementException`이 발생하므로 조심해야 한다.

```java
Member member = memberRepository.findByEmail(email).get(); // 조심
```

## 10. HTTP Status 정리

자주 사용하는 HTTP 상태 코드는 다음과 같다.

```text
400 Bad Request
→ 잘못된 요청

401 Unauthorized
→ 인증되지 않음

403 Forbidden
→ 인증은 됐지만 권한 없음

404 Not Found
→ 리소스를 찾을 수 없음

500 Internal Server Error
→ 서버 내부 오류
```

## 11. 직접 에러 응답 반환 방식과 비교

직접 에러 응답을 반환하는 방식도 가능하다.

```java
if (!ownerId.equals(loginUserId)) {
    return ApiErrorResponse.of(ErrorCode.FORBIDDEN, "수정 권한이 없습니다.");
}
```

하지만 이 방식은 Service가 응답 형식을 직접 알게 된다.

전역 예외 처리 방식:

```java
if (!ownerId.equals(loginUserId)) {
    throw new CustomException(ErrorCode.FORBIDDEN, "수정 권한이 없습니다.");
}
```

장점:

- Service는 비즈니스 예외만 던진다.
- 응답 형식은 GlobalExceptionHandler에서 통일한다.
- Controller마다 try-catch를 반복하지 않아도 된다.
- 에러 응답 형식을 일관되게 유지할 수 있다.

면접 답변:

> 에러 응답을 Service에서 직접 반환할 수도 있지만, 그러면 Service가 응답 형식에 의존하게 됩니다. 보통은 Service에서 CustomException 같은 비즈니스 예외를 던지고, `@RestControllerAdvice`가 붙은 GlobalExceptionHandler에서 이를 공통 ErrorResponse로 변환하는 방식을 선호합니다.

## 12. 한 번에 외우는 면접 답변

> Spring에서는 예외 처리를 각 Controller에서 try-catch로 반복하기보다 `@RestControllerAdvice`와 `@ExceptionHandler`를 사용해 전역으로 처리합니다. Service에서 문제가 발생하면 `CustomException` 같은 비즈니스 예외를 던지고, GlobalExceptionHandler가 이를 잡아 ErrorCode와 ErrorResponse를 이용해 일관된 JSON 응답으로 변환합니다. 이를 통해 Controller는 요청과 응답 흐름에 집중하고, Service는 비즈니스 로직과 예외 발생에 집중할 수 있습니다.
