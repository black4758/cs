# AOP 정리

## 1. AOP란?

AOP는 Aspect Oriented Programming의 약자이며, 관점 지향 프로그래밍이라고 한다.

여러 곳에서 반복되는 공통 기능을 핵심 비즈니스 로직과 분리해서 따로 관리하는 개념이다.

면접 답변:

> AOP는 관점 지향 프로그래밍으로, 로깅, 트랜잭션, 보안처럼 여러 곳에서 반복되는 공통 관심사를 핵심 비즈니스 로직과 분리해서 모듈화하는 개념입니다. 이를 통해 중복 코드를 줄이고 핵심 로직에 집중할 수 있습니다.

## 2. AOP를 사용하는 이유

비즈니스 로직을 작성하다 보면 여러 메서드에서 반복되는 기능이 생긴다.

예시:

- 로깅
- 트랜잭션
- 보안
- 실행 시간 측정
- 예외 처리

이런 코드를 모든 메서드에 직접 작성하면 중복 코드가 많아지고 핵심 로직이 지저분해진다.

예를 들어 AOP를 사용하지 않으면 다음처럼 각 메서드마다 로그 코드를 직접 작성해야 한다.

```java
public void join() {
    System.out.println("메서드 시작");

    // 회원가입 로직

    System.out.println("메서드 종료");
}
```

AOP를 사용하면 로그 기능을 따로 분리하고, 필요한 메서드에 공통으로 적용할 수 있다.

정리:

```text
핵심 관심사 → 회원가입, 주문 생성, 결제 같은 비즈니스 로직
공통 관심사 → 로깅, 트랜잭션, 보안 같은 반복 기능
```

## 3. AOP 핵심 용어

### Aspect

공통 기능을 모아둔 클래스이다.

예를 들어 로그를 남기는 `LogAspect` 클래스가 Aspect이다.

### Advice

실제로 실행되는 공통 기능이다.

예를 들어 메서드 실행 전후에 로그를 출력하는 코드가 Advice이다.

### Pointcut

공통 기능을 어디에 적용할지 정하는 조건이다.

예를 들어 `com.example.service` 패키지 안의 모든 메서드에 적용하겠다는 조건을 Pointcut이라고 한다.

### JoinPoint

AOP가 적용될 수 있는 지점이다.

Spring AOP에서는 보통 메서드 실행 지점이 JoinPoint가 된다.

### Weaving

핵심 로직에 공통 기능을 적용하는 과정이다.

## 4. Advice 종류

### @Before

대상 메서드 실행 전에 동작한다.

```java
@Before("execution(* com.example.service.*.*(..))")
public void before() {
    System.out.println("메서드 실행 전");
}
```

### @After

대상 메서드 실행 후에 동작한다.

```java
@After("execution(* com.example.service.*.*(..))")
public void after() {
    System.out.println("메서드 실행 후");
}
```

### @AfterReturning

대상 메서드가 정상적으로 반환된 후 동작한다.

```java
@AfterReturning("execution(* com.example.service.*.*(..))")
public void afterReturning() {
    System.out.println("정상 반환 후");
}
```

### @AfterThrowing

대상 메서드에서 예외가 발생했을 때 동작한다.

```java
@AfterThrowing("execution(* com.example.service.*.*(..))")
public void afterThrowing() {
    System.out.println("예외 발생");
}
```

### @Around

대상 메서드 실행 전후를 모두 감쌀 수 있다.

가장 강력하고 자주 사용되는 방식이다.

```java
@Around("execution(* com.example.service.*.*(..))")
public Object around(ProceedingJoinPoint joinPoint) throws Throwable {
    System.out.println("메서드 실행 전");

    Object result = joinPoint.proceed();

    System.out.println("메서드 실행 후");

    return result;
}
```

`joinPoint.proceed()`는 실제 대상 메서드를 실행하는 코드이다.

## 5. AOP 예시 코드

`LogAspect.java`

```java
@Aspect
@Component
public class LogAspect {

    @Around("execution(* com.example.service.*.*(..))")
    public Object log(ProceedingJoinPoint joinPoint) throws Throwable {
        System.out.println("메서드 시작: " + joinPoint.getSignature().getName());

        Object result = joinPoint.proceed();

        System.out.println("메서드 종료: " + joinPoint.getSignature().getName());

        return result;
    }
}
```

어노테이션 설명:

- `@Aspect`: 이 클래스가 AOP Aspect 클래스임을 나타낸다.
- `@Component`: Spring Bean으로 등록한다.
- `@Around`: 대상 메서드 실행 전후에 공통 기능을 적용한다.

코드 설명:

```java
@Around("execution(* com.example.service.*.*(..))")
```

`com.example.service` 패키지 안의 모든 클래스, 모든 메서드에 적용한다는 의미이다.

```java
joinPoint.getSignature().getName()
```

실행되는 메서드 이름을 가져온다.

```java
joinPoint.proceed()
```

실제 대상 메서드를 실행한다.

흐름:

```text
Service 메서드 호출
→ AOP가 먼저 가로챔
→ "메서드 시작" 출력
→ 실제 Service 메서드 실행
→ "메서드 종료" 출력
→ 결과 반환
```

## 6. AOP 적용 위치

AOP 클래스는 보통 `aop` 패키지에 만든다.

예시:

```text
src/main/java/com/example/demo
├── aop
│   └── LogAspect.java
├── member
│   ├── controller
│   ├── service
│   └── repository
└── config
```

또는 공통 기능을 모아두는 `global/aop` 패키지에 둘 수도 있다.

```text
src/main/java/com/example/demo/global/aop/LogAspect.java
```

## 7. AOP와 트랜잭션

Spring의 `@Transactional`도 AOP 기반으로 동작한다.

예를 들어 다음 메서드가 있다고 하자.

```java
@Transactional
public void join(MemberCreateRequest request) {
    Member member = Member.create(
        request.getEmail(),
        request.getPassword(),
        request.getName()
    );

    memberRepository.save(member);
}
```

Spring은 이 메서드 실행 전후에 트랜잭션 처리를 적용한다.

흐름:

```text
Service 메서드 호출
→ 트랜잭션 시작
→ 실제 비즈니스 로직 실행
→ 예외 없으면 commit
→ 예외 발생하면 rollback
```

즉 개발자가 직접 모든 메서드에 트랜잭션 시작, 커밋, 롤백 코드를 작성하지 않아도 된다.

면접 답변:

> Spring의 `@Transactional`은 AOP를 기반으로 동작합니다. 메서드 실행 전 트랜잭션을 시작하고, 메서드가 정상 종료되면 커밋하며, 예외가 발생하면 롤백합니다. 그래서 트랜잭션 처리 코드를 비즈니스 로직과 분리할 수 있습니다.

## 8. AOP 사용 시 주의할 점

### 1. 내부 메서드 호출에는 적용되지 않을 수 있다

Spring AOP는 프록시 기반으로 동작한다.

그래서 같은 클래스 내부에서 메서드를 직접 호출하면 AOP가 적용되지 않을 수 있다.

```java
public void methodA() {
    methodB();
}

@Transactional
public void methodB() {
}
```

위처럼 같은 클래스 안에서 `methodB()`를 직접 호출하면 프록시를 거치지 않아 AOP가 적용되지 않을 수 있다.

### 2. 너무 넓은 Pointcut은 조심해야 한다

Pointcut 범위를 너무 넓게 잡으면 원하지 않는 메서드에도 AOP가 적용될 수 있다.

```java
@Around("execution(* com.example..*(..))")
```

이런 방식은 프로젝트 전체에 적용될 수 있으므로 주의해야 한다.

## 9. 한 번에 외우는 면접 답변

> AOP는 관점 지향 프로그래밍으로, 로깅, 트랜잭션, 보안처럼 여러 곳에서 반복되는 공통 관심사를 핵심 비즈니스 로직과 분리하는 개념입니다. Spring AOP는 주로 프록시 기반으로 동작하며, Aspect에는 공통 기능을 작성하고 Pointcut으로 적용 대상을 지정합니다. 예를 들어 `@Around`를 사용하면 대상 메서드 실행 전후에 공통 로직을 적용할 수 있습니다. Spring의 `@Transactional`도 AOP 기반으로 동작해서 트랜잭션 시작, 커밋, 롤백 처리를 비즈니스 로직과 분리해줍니다.
