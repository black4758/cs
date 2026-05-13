# Service 정리

## 1. Service란?

Service는 비즈니스 로직을 처리하는 계층이다.

Controller는 클라이언트의 요청과 응답을 담당하고, Repository는 DB 접근을 담당한다. Service는 그 사이에서 실제 비즈니스 규칙을 처리한다.

예를 들어 회원가입 기능에서는 다음과 같은 로직을 Service에서 처리한다.

```text
이메일 중복 확인
→ Member 엔티티 생성
→ Repository를 통해 저장
```

면접 답변:

> Service는 비즈니스 로직을 처리하는 계층입니다. Controller는 요청과 응답을 담당하고, Repository는 DB 접근을 담당하며, Service는 그 사이에서 실제 비즈니스 규칙을 처리합니다.

## 2. Service 예시 코드

```java
@Service
@RequiredArgsConstructor
@Transactional(readOnly = true)
public class MemberService {

    private final MemberRepository memberRepository;

    @Transactional
    public void join(MemberCreateRequest request) {
        if (memberRepository.existsByEmail(request.getEmail())) {
            throw new IllegalArgumentException("이미 존재하는 이메일입니다.");
        }

        Member member = new Member(request.getName(), request.getEmail());
        memberRepository.save(member);
    }

    public MemberResponse findMember(Long id) {
        Member member = memberRepository.findById(id)
            .orElseThrow();

        return MemberResponse.from(member);
    }

    @Transactional
    public void changeName(Long id, String name) {
        Member member = memberRepository.findById(id)
            .orElseThrow();

        member.changeName(name);
    }
}
```

## 3. Service에서 자주 쓰는 어노테이션

### @Service

`@Service`는 해당 클래스가 Service 계층임을 나타내는 어노테이션이다.

내부적으로 `@Component`를 포함하고 있기 때문에 Spring Bean으로 등록된다.

```java
@Service
public class MemberService {
}
```

면접 답변:

> `@Service`는 비즈니스 로직을 담당하는 Service 계층 클래스에 사용하는 어노테이션입니다. 내부적으로 `@Component`를 포함하고 있어 Spring Bean으로 등록됩니다.

### @RequiredArgsConstructor

`@RequiredArgsConstructor`는 Lombok 어노테이션이다.

`final`이 붙은 필드를 파라미터로 받는 생성자를 자동으로 만들어준다.

```java
private final MemberRepository memberRepository;
```

위 필드가 있으면 Lombok이 아래 생성자를 자동으로 만든다.

```java
public MemberService(MemberRepository memberRepository) {
    this.memberRepository = memberRepository;
}
```

Spring은 생성자를 통해 `MemberRepository` Bean을 자동으로 주입한다.

면접 답변:

> `@RequiredArgsConstructor`는 final 필드에 대한 생성자를 자동으로 만들어주는 Lombok 어노테이션입니다. Service에서는 Repository 같은 의존성을 생성자 주입으로 받기 위해 자주 사용합니다.

### @Transactional(readOnly = true)

`@Transactional(readOnly = true)`는 조회 전용 트랜잭션이다.

Service 클래스 위에 붙이면 기본적으로 모든 메서드가 조회 전용 트랜잭션으로 동작한다.

```java
@Service
@RequiredArgsConstructor
@Transactional(readOnly = true)
public class MemberService {
}
```

조회 메서드는 데이터를 변경하지 않기 때문에 readOnly를 적용할 수 있다.

```java
public MemberResponse findMember(Long id) {
    Member member = memberRepository.findById(id)
        .orElseThrow();

    return MemberResponse.from(member);
}
```

사용 이유:

- 조회 메서드라는 의도를 명확히 표현한다.
- JPA의 불필요한 변경 감지 비용을 줄일 수 있다.
- 조회와 변경 작업을 구분하기 쉽다.
- 클래스 전체에 기본 설정을 걸어 중복을 줄일 수 있다.

면접 답변:

> Service 클래스에 `@Transactional(readOnly = true)`를 붙이는 이유는 기본 트랜잭션을 조회 전용으로 설정하기 위해서입니다. 조회 메서드는 데이터를 변경하지 않기 때문에 readOnly를 적용해 의도를 명확히 하고, JPA의 불필요한 변경 감지 비용을 줄일 수 있습니다.

### @Transactional

`@Transactional`은 저장, 수정, 삭제처럼 데이터 변경이 필요한 메서드에 사용한다.

클래스 위에 `@Transactional(readOnly = true)`가 있어도, 메서드 위에 일반 `@Transactional`을 붙이면 해당 메서드는 변경 가능한 트랜잭션으로 동작한다.

```java
@Transactional
public void join(MemberCreateRequest request) {
    Member member = new Member(request.getName(), request.getEmail());
    memberRepository.save(member);
}
```

```java
@Transactional
public void changeName(Long id, String name) {
    Member member = memberRepository.findById(id)
        .orElseThrow();

    member.changeName(name);
}
```

면접 답변:

> 저장, 수정, 삭제처럼 데이터를 변경하는 메서드에는 일반 `@Transactional`을 사용합니다. 클래스에 `readOnly = true`가 설정되어 있어도 메서드에 일반 `@Transactional`을 붙이면 해당 설정을 덮어써서 변경 가능한 트랜잭션으로 실행됩니다.

## 4. 생성자 주입

Service에서 Repository를 사용할 때는 보통 생성자 주입을 사용한다.

Lombok을 쓰면 `@RequiredArgsConstructor`로 생성자를 자동 생성할 수 있다.

```java
@Service
@RequiredArgsConstructor
public class MemberService {

    private final MemberRepository memberRepository;
}
```

Lombok을 사용하지 않으면 생성자를 직접 작성한다.

```java
@Service
public class MemberService {

    private final MemberRepository memberRepository;

    public MemberService(MemberRepository memberRepository) {
        this.memberRepository = memberRepository;
    }
}
```

Spring은 생성자가 하나만 있으면 `@Autowired` 없이도 해당 생성자를 통해 Bean을 자동 주입한다.

면접 답변:

> Service에서 Repository를 주입할 때는 보통 생성자 주입을 사용합니다. 생성자 주입은 의존성이 명확하고 테스트하기 좋습니다. Lombok의 `@RequiredArgsConstructor`를 사용하면 final 필드 기반 생성자를 자동으로 만들 수 있습니다.

## 5. join() 메서드 설명

```java
@Transactional
public void join(MemberCreateRequest request) {
    if (memberRepository.existsByEmail(request.getEmail())) {
        throw new IllegalArgumentException("이미 존재하는 이메일입니다.");
    }

    Member member = new Member(request.getName(), request.getEmail());
    memberRepository.save(member);
}
```

`join()`은 회원가입 로직이다.

처리 흐름:

```text
이메일 중복 확인
→ 중복이면 예외 발생
→ 중복이 아니면 Member 엔티티 생성
→ Repository로 저장
```

저장 작업이므로 일반 `@Transactional`을 붙인다.

## 6. Service에서 Entity 생성 방식

Service에서 회원가입 같은 저장 로직을 처리할 때는 Request DTO의 값을 꺼내 Entity를 생성한 뒤 Repository로 저장한다.

대표적인 Entity 생성 방식은 세 가지가 있다.

### 1. 생성자 방식

가장 단순한 방식이다.

Entity:

```java
@Entity
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class Member {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;

    private String email;

    public Member(String name, String email) {
        this.name = name;
        this.email = email;
    }
}
```

Service:

```java
Member member = new Member(request.getName(), request.getEmail());
memberRepository.save(member);
```

필드가 적고 생성 규칙이 단순하면 생성자 방식이 가장 쉽다.

### 2. 정적 팩토리 메서드 create()

Entity 안에 `create()` 같은 정적 팩토리 메서드를 만들어 사용하는 방식이다.

Entity:

```java
@Entity
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class Member {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;

    private String email;

    private Member(String name, String email) {
        this.name = name;
        this.email = email;
    }

    public static Member create(String name, String email) {
        return new Member(name, email);
    }
}
```

Service:

```java
Member member = Member.create(request.getName(), request.getEmail());
memberRepository.save(member);
```

`new Member(...)`보다 `Member.create(...)`가 "회원을 생성한다"는 의미를 더 잘 드러낼 수 있다.

면접 답변:

> 정적 팩토리 메서드 `create()`는 보통 Entity 내부에 작성합니다. 생성자를 직접 호출하는 대신 `Member.create()`처럼 의미 있는 이름으로 객체를 생성할 수 있고, 생성 과정을 한 곳에서 관리할 수 있습니다.

### 3. Builder 방식

필드가 많거나 선택적으로 넣을 값이 많을 때 사용할 수 있다.

Entity에서는 클래스 위에 `@Builder`를 붙이기보다 필요한 필드만 받는 생성자 위에 붙이는 방식이 안전하다.

Entity:

```java
@Entity
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class Member {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;

    private String email;

    @Builder
    private Member(String name, String email) {
        this.name = name;
        this.email = email;
    }
}
```

Service:

```java
Member member = Member.builder()
    .name(request.getName())
    .email(request.getEmail())
    .build();

memberRepository.save(member);
```

이 방식은 `id`가 Builder에 포함되지 않는다. `id`는 보통 DB가 자동 생성해야 하는 값이기 때문에 외부에서 넣지 않는 것이 좋다.

### 실무 기준 정리

```text
단순 생성: 생성자
도메인 의미 있음: create()
필드가 많거나 선택값이 많음: 생성자 위 @Builder
```

회원가입처럼 도메인 생성 의미가 있는 경우에는 `Member.create(...)` 방식이 읽기 좋다.

면접 답변:

> 실무에서는 팀 컨벤션에 따라 다르지만, 단순한 객체 생성은 생성자를 사용하고, 도메인 생성 의미를 명확히 하고 싶을 때는 `create()` 같은 정적 팩토리 메서드를 사용합니다. 필드가 많거나 선택값이 많을 때는 Builder를 사용할 수 있지만, Entity에서는 id 같은 DB 관리 필드가 외부에서 설정되지 않도록 생성자 위에 Builder를 붙이는 방식을 선호합니다.

## 7. findMember() 메서드 설명

```java
public MemberResponse findMember(Long id) {
    Member member = memberRepository.findById(id)
        .orElseThrow();

    return MemberResponse.from(member);
}
```

`findMember()`는 회원 조회 로직이다.

조회만 하기 때문에 클래스 위의 `@Transactional(readOnly = true)`가 적용된다.

처리 흐름:

```text
id로 Member 조회
→ 없으면 예외 발생
→ Member 엔티티를 MemberResponse DTO로 변환
→ 응답 반환
```

## 8. changeName() 메서드 설명

```java
@Transactional
public void changeName(Long id, String name) {
    Member member = memberRepository.findById(id)
        .orElseThrow();

    member.changeName(name);
}
```

`changeName()`은 회원 이름 수정 로직이다.

수정 작업이므로 일반 `@Transactional`을 붙인다.

여기서 `memberRepository.save(member)`를 다시 호출하지 않아도 된다.

이유는 JPA의 변경 감지 때문이다.

```text
@Transactional 시작
→ findById로 영속 상태 Member 조회
→ member.changeName(name)
→ 트랜잭션 종료
→ JPA 변경 감지
→ update 쿼리 실행
```

면접 답변:

> JPA에서는 트랜잭션 안에서 조회한 영속 상태의 엔티티 값을 변경하면, 트랜잭션 커밋 시점에 변경 감지가 일어나 자동으로 update 쿼리가 실행됩니다. 그래서 수정 작업에서는 보통 `save()`를 다시 호출하지 않아도 됩니다.

## 9. 한 번에 외우는 면접 답변

> Service는 비즈니스 로직을 처리하는 계층입니다. Controller는 요청과 응답을 담당하고, Repository는 DB 접근을 담당하며, Service는 그 사이에서 실제 비즈니스 규칙을 처리합니다. Service에서는 보통 `@Service`, `@RequiredArgsConstructor`, `@Transactional`을 자주 사용합니다. `@Service`는 Bean 등록, `@RequiredArgsConstructor`는 생성자 주입, `@Transactional`은 작업 단위를 트랜잭션으로 묶기 위해 사용합니다. 조회 메서드는 `readOnly = true`를 사용하고, 저장, 수정, 삭제 메서드는 일반 `@Transactional`을 사용합니다.
