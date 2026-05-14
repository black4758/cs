# Spring 정리

## 1. Spring이란?

Spring은 Java 기반의 백엔드 애플리케이션 프레임워크이다.

객체 생성, 의존성 관리, 트랜잭션 처리, 웹 요청 처리, 데이터 접근 같은 공통 기능을 프레임워크가 제공해 개발자가 비즈니스 로직에 집중할 수 있게 해준다.

면접 답변:

> Spring은 Java 기반의 애플리케이션 프레임워크입니다. IoC/DI를 통해 객체의 생성과 의존성 관리를 Spring 컨테이너가 담당하고, AOP를 통해 트랜잭션이나 로깅 같은 공통 관심사를 분리할 수 있습니다. 또한 PSA를 통해 특정 기술에 종속되지 않고 일관된 방식으로 여러 기술을 사용할 수 있게 해줍니다.

Spring의 핵심 개념:

- POJO: 순수한 자바 객체
- IoC/DI: 제어의 역전 / 의존성 주입
- AOP: 관점 지향 프로그래밍
- PSA: 일관된 서비스 추상화

## 2. Spring Boot의 3가지 특징

Spring Boot는 Spring을 더 쉽고 빠르게 사용할 수 있도록 도와주는 프레임워크이다.

Spring Boot의 대표적인 3가지 특징은 다음과 같다.

### 1. Auto Configuration

자동 설정을 의미한다.

Spring Boot는 프로젝트에 추가된 라이브러리와 설정 정보를 보고 필요한 Bean과 설정을 자동으로 구성해준다.

예를 들어 `spring-boot-starter-web`을 추가하면 웹 애플리케이션에 필요한 설정들이 자동으로 적용된다.

### 2. Starter

Starter는 자주 함께 사용하는 라이브러리들을 하나로 묶어둔 의존성 패키지이다.

예를 들어 `spring-boot-starter-web`을 추가하면 Spring MVC, Tomcat, Jackson 등 웹 개발에 필요한 라이브러리들이 함께 추가된다.

### 3. Embedded Server

내장 서버를 의미한다.

Spring Boot는 Tomcat 같은 WAS를 애플리케이션 내부에 포함할 수 있다. 그래서 별도의 서버를 설치하지 않아도 `jar` 파일을 실행하는 것만으로 애플리케이션을 실행할 수 있다.

면접 답변:

> Spring Boot는 자동 설정, Starter 의존성, 내장 서버를 통해 Spring 애플리케이션을 빠르게 개발하고 실행할 수 있게 해줍니다. 자동 설정은 필요한 설정을 자동으로 구성해주고, Starter는 관련 라이브러리들을 한 번에 가져오게 해주며, 내장 서버는 별도의 WAS 설치 없이 애플리케이션을 실행할 수 있게 해줍니다.

## 3. Spring과 Spring Boot의 차이

Spring은 Java 기반 애플리케이션을 만들기 위한 프레임워크이고, Spring Boot는 Spring을 더 편하게 사용할 수 있도록 도와주는 도구이다.

Spring은 설정이 비교적 많고, 서버 설정도 직접 해야 하는 경우가 많다. 반면 Spring Boot는 자동 설정, Starter, 내장 서버를 제공해서 설정과 실행 과정을 단순화한다.

면접 답변:

> Spring은 Java 기반 애플리케이션 프레임워크이고, Spring Boot는 Spring 애플리케이션을 더 쉽고 빠르게 만들 수 있도록 도와주는 프레임워크입니다. Spring Boot는 자동 설정, Starter 의존성, 내장 서버를 제공하기 때문에 복잡한 설정을 줄이고 빠르게 애플리케이션을 실행할 수 있습니다.

정리:

| 구분 | Spring | Spring Boot |
| --- | --- | --- |
| 목적 | Java 애플리케이션 개발 프레임워크 | Spring을 쉽게 사용하도록 도와주는 프레임워크 |
| 설정 | 직접 설정할 부분이 많음 | 자동 설정 제공 |
| 의존성 | 필요한 라이브러리를 직접 조합 | Starter로 한 번에 관리 |
| 서버 실행 | 별도 WAS 설정이 필요한 경우가 많음 | 내장 서버로 바로 실행 가능 |

## 4. PSA

PSA는 Portable Service Abstraction의 약자이다.

특정 기술에 직접 의존하지 않고, 일관된 방식으로 서비스를 사용할 수 있게 해주는 Spring의 서비스 추상화 개념이다.

예를 들어 트랜잭션 처리를 할 때 개발자는 `@Transactional`만 사용하면 된다.

```java
@Transactional
public void saveOrder() {
    // 주문 저장 로직
}
```

내부 구현은 JDBC, JPA, Hibernate 등으로 달라질 수 있지만 개발자는 같은 방식으로 트랜잭션을 처리할 수 있다.

면접 답변:

> PSA는 Portable Service Abstraction의 약자로, 특정 기술에 종속되지 않고 일관된 방식으로 사용할 수 있도록 Spring이 제공하는 서비스 추상화입니다. 예를 들어 `@Transactional`을 사용하면 JDBC, JPA 등 내부 기술이 달라도 같은 방식으로 트랜잭션을 처리할 수 있습니다.

PSA 예시:

- `@Transactional`
- Spring MVC
- Spring Data JPA
- Spring Cache

## 5. AOP

AOP는 Aspect Oriented Programming의 약자이며, 관점 지향 프로그래밍이라고 한다.

여러 곳에서 반복되는 공통 기능을 핵심 비즈니스 로직과 분리해서 따로 관리하는 개념이다.

예를 들어 다음과 같은 기능은 여러 메서드에서 반복될 수 있다.

- 로깅
- 트랜잭션
- 보안
- 실행 시간 측정
- 예외 처리

이런 공통 기능을 각 비즈니스 로직에 직접 작성하면 중복 코드가 많아지고 핵심 로직이 지저분해질 수 있다. AOP는 이런 공통 관심사를 분리해서 필요한 곳에 적용할 수 있게 해준다.

예시:

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

흐름:

```text
Service 메서드 호출
→ AOP가 먼저 가로챔
→ 공통 기능 실행
→ 실제 비즈니스 로직 실행
→ 공통 기능 실행
→ 응답 반환
```

면접 답변:

> AOP는 관점 지향 프로그래밍으로, 로깅, 트랜잭션, 보안처럼 여러 곳에서 반복되는 공통 관심사를 핵심 비즈니스 로직과 분리해서 모듈화하는 개념입니다. 이를 통해 중복 코드를 줄이고 핵심 로직에 집중할 수 있습니다.

용어 정리:

- Aspect: 공통 기능을 모아둔 클래스
- Advice: 실제로 실행되는 공통 기능
- Pointcut: 공통 기능을 어디에 적용할지 정하는 조건
- JoinPoint: AOP가 적용될 수 있는 지점

## 6. POJO

POJO는 Plain Old Java Object의 약자이다.

특정 프레임워크나 기술에 종속되지 않는 순수한 Java 객체를 의미한다.

Spring은 POJO 기반 개발을 지향한다. 즉, 비즈니스 로직을 특정 기술에 강하게 묶지 않고 순수한 Java 코드로 작성할 수 있게 도와준다.

예시:

```java
public class Member {
    private String name;
    private int age;

    public Member(String name, int age) {
        this.name = name;
        this.age = age;
    }
}
```

면접 답변:

> POJO는 특정 프레임워크나 기술에 종속되지 않는 순수한 Java 객체를 의미합니다. Spring은 POJO 기반 개발을 지원해서 비즈니스 로직이 특정 기술에 강하게 의존하지 않도록 하고, 테스트와 유지보수를 쉽게 만들어줍니다.

## 7. Spring Container

Spring Container는 Spring Bean을 생성하고 관리하는 핵심 객체이다.

개발자가 직접 `new`로 객체를 생성하고 의존성을 연결하는 것이 아니라, Spring Container가 객체를 생성하고 필요한 의존성을 주입해준다.

Spring Container가 하는 일:

- Bean 생성
- Bean 관리
- 의존성 주입
- Bean 생명주기 관리
- 필요한 Bean을 찾아 연결

예시:

```java
@Service
public class OrderService {

    private final OrderRepository orderRepository;

    public OrderService(OrderRepository orderRepository) {
        this.orderRepository = orderRepository;
    }
}
```

위 코드에서 `OrderService`는 `OrderRepository`를 직접 생성하지 않는다. Spring Container가 `OrderRepository` Bean을 찾아서 `OrderService`에 주입해준다.

면접 답변:

> Spring Container는 Spring Bean을 생성하고 관리하는 핵심 객체입니다. 객체의 생성, 의존성 주입, 초기화, 소멸 같은 생명주기를 관리하며, IoC/DI를 실제로 수행하는 역할을 합니다.

대표적인 Spring Container:

- BeanFactory: 가장 기본적인 컨테이너
- ApplicationContext: BeanFactory 기능에 이벤트, 국제화, 환경 설정 같은 기능이 추가된 컨테이너

실무에서는 대부분 `ApplicationContext`를 사용한다.

## 8. IoC/DI

### IoC

IoC는 Inversion of Control의 약자이며, 제어의 역전이라고 한다.

기존에는 개발자가 직접 객체를 생성하고 관리했다. 하지만 Spring에서는 객체의 생성과 생명주기 관리를 Spring 컨테이너가 담당한다.

즉, 객체에 대한 제어권이 개발자에게서 Spring 컨테이너로 넘어간 것이다.

### DI

DI는 Dependency Injection의 약자이며, 의존성 주입이라고 한다.

객체가 필요한 의존 객체를 직접 생성하지 않고 외부에서 주입받는 방식이다. Spring에서는 Spring 컨테이너가 필요한 객체를 주입해준다.

예시:

```java
@Service
public class OrderService {

    private final OrderRepository orderRepository;

    public OrderService(OrderRepository orderRepository) {
        this.orderRepository = orderRepository;
    }
}
```

`OrderService`가 `OrderRepository`를 직접 생성하지 않고 생성자를 통해 주입받고 있다.

면접 답변:

> IoC는 객체의 생성과 제어권을 개발자가 아니라 Spring 컨테이너가 가지는 것을 의미합니다. DI는 IoC를 구현하는 대표적인 방법으로, 객체가 필요한 의존성을 직접 생성하지 않고 외부에서 주입받는 방식입니다. 이를 통해 객체 간 결합도를 낮추고 테스트와 유지보수를 쉽게 할 수 있습니다.

## 9. 한 번에 외우는 면접 답변

> Spring은 Java 기반의 백엔드 애플리케이션 프레임워크입니다. 핵심 개념으로는 POJO, IoC/DI, AOP, PSA가 있습니다. POJO는 특정 기술에 종속되지 않는 순수 Java 객체를 의미하고, IoC/DI는 객체의 생성과 의존성 관리를 Spring 컨테이너가 담당하게 하는 개념입니다. AOP는 트랜잭션이나 로깅 같은 공통 관심사를 핵심 로직과 분리하는 것이고, PSA는 특정 기술에 종속되지 않고 일관된 방식으로 서비스를 사용할 수 있게 해주는 추상화입니다. Spring Boot는 이러한 Spring을 더 쉽게 사용할 수 있도록 자동 설정, Starter, 내장 서버를 제공하는 프레임워크입니다.
