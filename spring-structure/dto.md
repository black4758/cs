# DTO 정리

## 1. DTO란?

DTO는 Data Transfer Object의 약자이다.

계층 간 데이터를 전달하기 위해 사용하는 객체이다.

Spring REST API에서는 보통 DTO를 두 가지로 나누어 사용한다.

- Request DTO: 클라이언트 요청 데이터를 받는 객체
- Response DTO: 서버 응답 데이터를 담는 객체

면접 답변:

> DTO는 Data Transfer Object의 약자로, 계층 간 데이터를 전달하기 위한 객체입니다. Spring REST API에서는 클라이언트 요청을 받는 Request DTO와 서버 응답을 내려주는 Response DTO로 나누어 사용하는 경우가 많습니다. DTO를 사용하면 Entity를 직접 외부에 노출하지 않고, API에 필요한 데이터만 주고받을 수 있습니다.

## 2. DTO를 사용하는 이유

DTO를 사용하는 이유:

- Entity를 외부에 직접 노출하지 않기 위해
- 요청과 응답 형식을 분리하기 위해
- API에 필요한 데이터만 주고받기 위해
- Request DTO에서 입력값 검증을 하기 위해
- Entity 변경이 API 응답 구조에 바로 영향을 주지 않게 하기 위해

면접 답변:

> DTO를 사용하면 Entity를 직접 외부에 노출하지 않고, API 요청과 응답에 필요한 데이터만 주고받을 수 있습니다. 또한 Entity 구조가 변경되어도 API 스펙에 주는 영향을 줄일 수 있습니다.

## 3. Request DTO

Request DTO는 클라이언트가 보낸 요청 데이터를 받는 객체이다.

예를 들어 클라이언트가 아래 JSON을 보낸다고 하자.

```json
{
  "name": "kim",
  "email": "kim@test.com"
}
```

이 요청을 받는 DTO는 다음과 같이 만들 수 있다.

```java
import lombok.Getter;
import lombok.NoArgsConstructor;

@Getter
@NoArgsConstructor
public class MemberCreateRequest {

    private String name;

    private String email;
}
```

컨트롤러에서 사용:

```java
@PostMapping("/members")
public void createMember(@RequestBody MemberCreateRequest request) {
    memberService.create(request.getName(), request.getEmail());
}
```

### Request DTO에서 자주 쓰는 Lombok

```java
@Getter
@NoArgsConstructor
```

### @Getter

컨트롤러나 서비스에서 요청 DTO의 값을 꺼내기 위해 사용한다.

```java
request.getName();
request.getEmail();
```

### @NoArgsConstructor

파라미터가 없는 기본 생성자를 만들어준다.

Spring은 JSON 요청을 Java 객체로 변환할 때 Jackson을 사용한다. Jackson이 Request DTO 객체를 만들고 값을 넣기 위해 기본 생성자가 필요할 수 있다.

흐름:

```text
JSON 요청
→ Jackson
→ 기본 생성자로 Request DTO 객체 생성
→ JSON 값을 DTO 필드에 매핑
→ Controller 파라미터로 전달
```

면접 답변:

> Request DTO는 클라이언트의 JSON 요청을 받기 위한 객체입니다. Spring은 Jackson을 사용해 JSON을 Java 객체로 변환하는데, 이때 기본 생성자가 필요할 수 있어 `@NoArgsConstructor`를 자주 사용합니다. 그리고 요청 값을 꺼내기 위해 `@Getter`를 사용합니다.

### Request DTO에 Builder를 잘 쓰지 않는 이유

Request DTO는 개발자가 직접 생성하는 객체가 아니라, 클라이언트 요청 JSON을 Jackson이 변환해서 만들어주는 객체이다.

그래서 보통 아래처럼 직접 만들 일이 거의 없다.

```java
MemberCreateRequest request = MemberCreateRequest.builder()
    .name("kim")
    .email("kim@test.com")
    .build();
```

정리:

- Request DTO는 클라이언트 요청을 받는 객체이다.
- 보통 Jackson이 생성하고 값을 채워준다.
- 개발자가 직접 Builder로 생성할 일이 적다.
- 그래서 Request DTO에는 Builder가 필수는 아니다.

면접 답변:

> Request DTO는 클라이언트 요청 데이터를 바인딩하기 위한 객체라서 보통 개발자가 직접 생성하지 않고 Jackson이 값을 채워줍니다. 그래서 Builder가 꼭 필요하지 않습니다.

## 4. Response DTO

Response DTO는 서버가 클라이언트에게 응답할 데이터를 담는 객체이다.

Entity를 그대로 반환하지 않고, 필요한 값만 Response DTO에 담아 응답한다.

예시:

```java
import lombok.AllArgsConstructor;
import lombok.Getter;

@Getter
@AllArgsConstructor
public class MemberResponse {

    private Long id;

    private String name;

    private String email;

    public static MemberResponse from(Member member) {
        return new MemberResponse(
            member.getId(),
            member.getName(),
            member.getEmail()
        );
    }
}
```

컨트롤러에서 사용:

```java
@GetMapping("/members/{id}")
public MemberResponse getMember(@PathVariable Long id) {
    Member member = memberService.findById(id);
    return MemberResponse.from(member);
}
```

### Response DTO에서 자주 쓰는 Lombok

```java
@Getter
@AllArgsConstructor
```

### @Getter

Response DTO의 값을 JSON으로 변환할 때 필요하다.

Spring은 Response DTO를 JSON으로 변환할 때 getter를 통해 값을 읽는다.

### @AllArgsConstructor

모든 필드를 받는 생성자를 만들어준다.

Response DTO는 서버가 직접 만들어서 반환하는 객체이기 때문에 생성자를 통해 값을 넣어 만들 수 있다.

```java
new MemberResponse(member.getId(), member.getName(), member.getEmail());
```

### Response DTO에 @NoArgsConstructor를 보통 안 쓰는 이유

Response DTO는 서버가 직접 생성해서 클라이언트에게 반환하는 객체이다.

즉, JSON을 Java 객체로 변환하는 것이 아니라 Java 객체를 JSON으로 변환하는 방향이다.

그래서 Request DTO처럼 Jackson이 빈 객체를 먼저 만들어야 하는 상황이 보통 없다.

정리:

- Request DTO: JSON → Java 객체
- Response DTO: Java 객체 → JSON

면접 답변:

> Response DTO는 서버에서 직접 생성해서 반환하는 객체이기 때문에 보통 기본 생성자가 필요하지 않습니다. 반면 Request DTO는 클라이언트의 JSON 요청을 Jackson이 Java 객체로 역직렬화해야 하므로 기본 생성자가 필요할 수 있습니다.

## 5. from() 정적 팩토리 메서드

Response DTO에서는 Entity를 DTO로 변환하기 위해 `from()` 정적 팩토리 메서드를 자주 사용한다.

```java
public static MemberResponse from(Member member) {
    return new MemberResponse(
        member.getId(),
        member.getName(),
        member.getEmail()
    );
}
```

사용 예시:

```java
Member member = memberService.findById(id);
MemberResponse response = MemberResponse.from(member);
```

`from()`을 사용하면 어떤 객체로부터 Response DTO를 만드는지 의도가 명확해진다.

```text
Member로부터 MemberResponse를 만든다
```

면접 답변:

> Response DTO는 Entity를 DTO로 변환하는 경우가 많기 때문에 `from()` 같은 정적 팩토리 메서드를 자주 사용합니다. `from()`을 사용하면 어떤 객체로부터 DTO를 생성하는지 의도가 명확해집니다.

## 6. Response DTO와 Builder

Response DTO에서도 Builder를 사용할 수 있다.

```java
import lombok.Builder;
import lombok.Getter;

@Getter
@Builder
public class MemberResponse {

    private Long id;

    private String name;

    private String email;
}
```

사용 예시:

```java
MemberResponse response = MemberResponse.builder()
    .id(member.getId())
    .name(member.getName())
    .email(member.getEmail())
    .build();
```

하지만 단순히 Entity를 Response DTO로 변환하는 경우에는 `from()`이 더 명확한 경우가 많다.

정리:

- 단순 Entity → Response DTO 변환: `from()` 사용
- 필드가 많거나 선택적으로 조립할 값이 많음: Builder 사용 가능
- Request DTO: Builder 필요성이 낮음

면접 답변:

> Builder는 DTO에서도 사용할 수 있지만 필수는 아닙니다. 단순히 Entity를 Response DTO로 변환하는 경우에는 `from()` 정적 팩토리 메서드가 의도를 더 명확하게 표현할 수 있고, 필드가 많거나 선택적으로 값을 조립해야 할 때는 Builder를 사용할 수 있습니다.

## 7. 한 번에 외우는 면접 답변

> DTO는 계층 간 데이터를 전달하기 위한 객체입니다. REST API에서는 보통 Request DTO와 Response DTO로 나누어 사용합니다. Request DTO는 클라이언트의 JSON 요청을 받는 객체라서 Jackson이 객체로 변환할 수 있도록 `@NoArgsConstructor`를 자주 사용하고, 값을 꺼내기 위해 `@Getter`를 사용합니다. Response DTO는 서버가 직접 만들어 응답하는 객체라서 `@AllArgsConstructor`와 `from()` 정적 팩토리 메서드를 자주 사용합니다. DTO를 사용하면 Entity를 직접 외부에 노출하지 않고 API에 필요한 데이터만 주고받을 수 있습니다.
