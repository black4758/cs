# Controller 정리

## 1. Controller란?

Controller는 클라이언트의 HTTP 요청을 받는 계층이다.

REST API 기준으로 Controller는 요청 URL과 HTTP Method를 받아서, 필요한 요청 데이터를 DTO로 받고, Service를 호출한 뒤 응답을 반환한다.

흐름:

```text
요청 받기
→ 요청 데이터 꺼내기
→ Service 호출
→ 응답 반환
```

면접 답변:

> Controller는 클라이언트의 HTTP 요청을 받는 계층입니다. REST API에서는 요청 데이터를 DTO로 받고, Service를 호출한 뒤 처리 결과를 Response DTO로 반환하는 역할을 합니다.

## 2. Controller 예시 코드

```java
@RestController
@RequiredArgsConstructor
@RequestMapping("/members")
public class MemberController {

    private final MemberService memberService;

    @PostMapping
    public void join(@RequestBody MemberCreateRequest request) {
        memberService.join(request);
    }

    @GetMapping("/{id}")
    public MemberResponse findMember(@PathVariable Long id) {
        return memberService.findMember(id);
    }

    @PatchMapping("/{id}/name")
    public void changeName(
        @PathVariable Long id,
        @RequestBody MemberNameUpdateRequest request
    ) {
        memberService.changeName(id, request.getName());
    }

    @DeleteMapping("/{id}")
    public void deleteMember(@PathVariable Long id) {
        memberService.deleteMember(id);
    }

    @GetMapping
    public List<MemberResponse> findMembers(@RequestParam String name) {
        return memberService.findMembersByName(name);
    }
}
```

## 3. Controller에서 자주 쓰는 어노테이션

### @RestController

REST API 컨트롤러를 만들 때 사용한다.

`@RestController`는 `@Controller`와 `@ResponseBody`를 합친 어노테이션이다.

```text
@RestController = @Controller + @ResponseBody
```

메서드 반환값이 View 이름이 아니라 HTTP Response Body에 직접 들어간다. 보통 JSON 형태로 응답된다.

면접 답변:

> `@RestController`는 REST API 컨트롤러를 만들 때 사용하는 어노테이션입니다. `@Controller`와 `@ResponseBody`를 합친 것으로, 메서드 반환값이 View 이름이 아니라 HTTP Response Body에 직접 담깁니다.

### @RequiredArgsConstructor

Service를 생성자 주입하기 위해 자주 사용한다.

```java
private final MemberService memberService;
```

`final` 필드에 대한 생성자를 Lombok이 자동으로 만들어주고, Spring이 해당 생성자를 통해 Service Bean을 주입한다.

면접 답변:

> `@RequiredArgsConstructor`는 final 필드에 대한 생성자를 자동으로 만들어주는 Lombok 어노테이션입니다. Controller에서는 Service를 생성자 주입하기 위해 자주 사용합니다.

### @RequestMapping

컨트롤러의 공통 URL 경로를 지정할 때 사용한다.

```java
@RequestMapping("/members")
public class MemberController {
}
```

위 코드에서는 `MemberController` 안의 모든 API가 `/members`로 시작한다.

## 4. HTTP Method 매핑

### @PostMapping

생성 요청을 처리할 때 주로 사용한다.

```java
@PostMapping
public void join(@RequestBody MemberCreateRequest request) {
    memberService.join(request);
}
```

요청 예시:

```http
POST /members
```

### @GetMapping

조회 요청을 처리할 때 사용한다.

```java
@GetMapping("/{id}")
public MemberResponse findMember(@PathVariable Long id) {
    return memberService.findMember(id);
}
```

요청 예시:

```http
GET /members/1
```

### @PatchMapping

일부 수정 요청을 처리할 때 사용한다.

```java
@PatchMapping("/{id}/name")
public void changeName(
    @PathVariable Long id,
    @RequestBody MemberNameUpdateRequest request
) {
    memberService.changeName(id, request.getName());
}
```

요청 예시:

```http
PATCH /members/1/name
```

### @DeleteMapping

삭제 요청을 처리할 때 사용한다.

```java
@DeleteMapping("/{id}")
public void deleteMember(@PathVariable Long id) {
    memberService.deleteMember(id);
}
```

요청 예시:

```http
DELETE /members/1
```

## 5. 요청 데이터를 받는 어노테이션

### @RequestBody

요청 body의 JSON 데이터를 DTO로 받을 때 사용한다.

```java
@PostMapping
public void join(@RequestBody MemberCreateRequest request) {
    memberService.join(request);
}
```

요청 예시:

```json
{
  "name": "kim",
  "email": "kim@test.com"
}
```

정리:

```text
@RequestBody → JSON body를 DTO로 받음
```

### @PathVariable

URL 경로에 있는 값을 받을 때 사용한다.

```java
@GetMapping("/{id}")
public MemberResponse findMember(@PathVariable Long id) {
    return memberService.findMember(id);
}
```

요청 예시:

```http
GET /members/1
```

여기서 `1`이 `id`로 들어온다.

정리:

```text
@PathVariable → URL 경로 값
```

### @RequestParam

쿼리 파라미터 값을 받을 때 사용한다.

```java
@GetMapping
public List<MemberResponse> findMembers(@RequestParam String name) {
    return memberService.findMembersByName(name);
}
```

요청 예시:

```http
GET /members?name=kim
```

여기서 `kim`이 `name`으로 들어온다.

정리:

```text
@RequestParam → 쿼리 파라미터
```

### @RequestHeader

HTTP Header 값을 받을 때 사용한다.

예를 들어 Authorization 헤더에서 토큰을 받을 수 있다.

```java
@GetMapping("/me")
public MemberResponse myInfo(
    @RequestHeader("Authorization") String authorization
) {
    String token = authorization.replace("Bearer ", "");
    return memberService.findMyInfo(token);
}
```

요청 예시:

```http
GET /members/me
Authorization: Bearer eyJhbGciOi...
```

정리:

```text
@RequestHeader → HTTP Header 값
```

## 6. 인증된 사용자 정보 받기

Spring Security를 사용하면 컨트롤러에서 `@AuthenticationPrincipal`로 현재 인증된 사용자 정보를 받을 수 있다.

```java
@GetMapping("/me")
public MemberResponse myInfo(
    @AuthenticationPrincipal CustomUserDetails userDetails
) {
    Long memberId = userDetails.getId();

    return memberService.findMember(memberId);
}
```

여기서 들어오는 값은 토큰 자체가 아니라, 인증 필터가 토큰을 검증한 뒤 `SecurityContext`에 저장한 사용자 정보 객체이다.

흐름:

```text
클라이언트 요청
Authorization: Bearer JWT토큰
→ JWT 인증 필터가 토큰 추출
→ 토큰 검증
→ 사용자 정보 추출
→ CustomUserDetails 객체 생성
→ SecurityContext에 저장
→ Controller에서 @AuthenticationPrincipal로 받음
```

면접 답변:

> `@AuthenticationPrincipal`은 Spring Security에서 현재 인증된 사용자 정보를 컨트롤러 메서드 파라미터로 주입받을 때 사용하는 어노테이션입니다. JWT 필터나 로그인 과정에서 인증이 완료되면 사용자 정보가 SecurityContext에 저장되고, 컨트롤러에서는 `@AuthenticationPrincipal`을 통해 그 정보를 바로 사용할 수 있습니다.

## 7. Controller에서 주의할 점

Controller에는 비즈니스 로직을 많이 넣지 않는 것이 좋다.

Controller는 요청과 응답을 담당하고, 실제 비즈니스 로직은 Service에서 처리한다.

예를 들어 다음 로직은 Service에서 처리하는 것이 좋다.

- 이메일 중복 검사
- 회원 생성
- 이름 변경
- 회원 삭제
- 비밀번호 검증
- 토큰 발급

면접 답변:

> Controller는 요청과 응답을 담당하고, 비즈니스 로직은 Service에서 처리하는 것이 좋습니다. 이렇게 역할을 분리하면 코드의 책임이 명확해지고 유지보수가 쉬워집니다.

## 8. 한 번에 외우는 면접 답변

> Controller는 클라이언트의 HTTP 요청을 받는 계층입니다. REST API에서는 `@RestController`를 사용해 JSON 응답을 반환하고, 요청 데이터는 `@RequestBody`, `@PathVariable`, `@RequestParam`, `@RequestHeader` 등을 통해 받습니다. Controller는 직접 비즈니스 로직을 처리하기보다는 Service를 호출하고, 처리 결과를 Response DTO로 반환하는 역할을 합니다. 인증된 사용자 정보가 필요한 경우에는 Spring Security의 `@AuthenticationPrincipal`을 사용할 수 있습니다.
