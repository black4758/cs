# JPA 정리

## 1. JPA란?

JPA는 Java Persistence API의 약자이다.

Java 객체와 관계형 데이터베이스 테이블을 매핑해서, SQL을 직접 많이 작성하지 않고 객체 중심으로 DB를 다룰 수 있게 해주는 ORM 표준 기술이다.

예를 들어 DB에는 `members` 테이블이 있고, Java에는 `Member` 엔티티가 있다고 하자.

```java
@Entity
@Table(name = "members")
public class Member {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;

    private String email;
}
```

매핑 관계:

```text
Member 클래스 → members 테이블
id 필드 → id 컬럼
name 필드 → name 컬럼
email 필드 → email 컬럼
```

면접 답변:

> JPA는 Java Persistence API의 약자로, Java 객체와 관계형 데이터베이스 테이블을 매핑해서 객체 중심으로 데이터를 다룰 수 있게 해주는 ORM 표준 기술입니다. JPA를 사용하면 SQL을 직접 작성하는 대신 엔티티 객체를 저장하거나 조회하는 방식으로 DB 작업을 처리할 수 있습니다.

## 2. JPA를 쓰는 이유와 장점

JPA를 사용하는 가장 큰 이유는 SQL 중심 개발에서 벗어나 객체 중심으로 데이터를 다루기 위해서이다.

### 1. 반복적인 SQL 작성을 줄일 수 있다

JPA를 사용하면 기본적인 저장, 조회, 수정, 삭제 SQL을 직접 작성하지 않아도 된다.

```java
memberRepository.save(member);
memberRepository.findById(id);
memberRepository.findAll();
memberRepository.delete(member);
```

JPA가 내부적으로 필요한 SQL을 생성해 실행한다.

### 2. 객체 중심으로 개발할 수 있다

개발자는 테이블보다 Entity 객체를 중심으로 코드를 작성할 수 있다.

```java
Member member = new Member("kim", "kim@test.com");
memberRepository.save(member);
```

즉, SQL 결과를 직접 객체로 변환하는 작업을 줄일 수 있다.

### 3. 변경 감지를 사용할 수 있다

트랜잭션 안에서 조회한 엔티티의 값을 변경하면, JPA가 변경을 감지해서 자동으로 update 쿼리를 실행한다.

```java
@Transactional
public void changeName(Long id, String name) {
    Member member = memberRepository.findById(id)
        .orElseThrow();

    member.changeName(name);
}
```

수정할 때 `save()`를 다시 호출하지 않아도 되는 이유가 변경 감지 때문이다.

### 4. DB 독립성을 얻을 수 있다

JPA는 특정 DB에 직접 의존하는 SQL을 줄여준다.

예를 들어 MySQL, PostgreSQL, H2 등 DB가 달라도 JPA 방언 설정을 통해 SQL 차이를 어느 정도 처리할 수 있다.

### 5. 생산성과 유지보수성이 좋아진다

반복적인 CRUD 코드를 줄이고, Entity와 Repository 중심으로 코드를 작성할 수 있어 개발 생산성이 좋아진다.

또한 SQL이 여러 곳에 흩어지는 것을 줄이고, 객체 모델 중심으로 구조를 관리할 수 있다.

면접 답변:

> JPA를 사용하는 이유는 반복적인 SQL 작성을 줄이고 객체 중심으로 데이터를 다루기 위해서입니다. JPA는 기본 CRUD SQL을 자동으로 처리해주고, 영속성 컨텍스트를 통해 변경 감지 같은 기능을 제공합니다. 또한 특정 DB에 대한 의존을 줄이고 생산성과 유지보수성을 높일 수 있습니다.

## 3. ORM이란?

ORM은 Object Relational Mapping의 약자이다.

객체와 관계형 데이터베이스를 매핑하는 기술이다.

```text
Object → Java 객체
Relational → 관계형 데이터베이스
Mapping → 서로 연결
```

ORM을 사용하면 객체를 저장하는 것처럼 코드를 작성해도, 내부적으로는 SQL이 실행된다.

```java
memberRepository.save(member);
```

내부적으로 실행되는 SQL 예시:

```sql
insert into members (name, email) values (?, ?)
```

면접 답변:

> ORM은 객체와 관계형 데이터베이스를 매핑하는 기술입니다. 객체를 저장하거나 조회하는 방식으로 코드를 작성하면, ORM이 내부적으로 SQL을 생성해 데이터베이스와 통신합니다.

## 4. JPA, Hibernate, Spring Data JPA 차이

### JPA

JPA는 Java에서 ORM을 사용하기 위한 표준 인터페이스이다.

즉, JPA 자체는 구현체가 아니라 규칙이다.

### Hibernate

Hibernate는 JPA를 실제로 구현한 대표적인 구현체이다.

JPA라는 규칙을 실제로 동작하게 만들어주는 기술이다.

### Spring Data JPA

Spring Data JPA는 JPA를 더 편하게 사용할 수 있도록 도와주는 Spring 프로젝트이다.

`JpaRepository` 같은 인터페이스를 제공해서 기본 CRUD 메서드를 직접 구현하지 않아도 사용할 수 있게 해준다.

```java
public interface MemberRepository extends JpaRepository<Member, Long> {
}
```

정리:

```text
JPA → ORM 표준 인터페이스
Hibernate → JPA 구현체
Spring Data JPA → JPA를 편하게 쓰게 해주는 Spring 기술
```

면접 답변:

> JPA는 ORM을 위한 Java 표준 인터페이스이고, Hibernate는 JPA를 실제로 구현한 대표적인 구현체입니다. Spring Data JPA는 JPA를 더 편하게 사용할 수 있도록 Repository와 기본 CRUD 기능을 제공하는 Spring 프로젝트입니다.

## 5. Entity

Entity는 DB 테이블과 매핑되는 Java 객체이다.

```java
@Entity
@Table(name = "members")
public class Member {
}
```

`@Entity`를 붙이면 JPA가 관리하는 엔티티가 되고, `@Table`을 사용하면 매핑할 테이블 이름을 지정할 수 있다.

면접 답변:

> Entity는 JPA에서 데이터베이스 테이블과 매핑되는 Java 객체입니다. 클래스에는 `@Entity`를 붙이고, 실제 테이블명을 지정할 때는 `@Table`을 사용할 수 있습니다.

## 6. 영속성 컨텍스트

영속성 컨텍스트는 JPA가 엔티티를 관리하는 공간이다.

쉽게 말하면, DB에서 조회하거나 저장한 엔티티 객체를 JPA가 관리하는 1차 캐시 같은 공간이다.

```text
애플리케이션
→ EntityManager
→ 영속성 컨텍스트
→ DB
```

JPA는 영속성 컨텍스트 안에서 엔티티를 관리하며, 변경 감지나 1차 캐시 같은 기능을 제공한다.

면접 답변:

> 영속성 컨텍스트는 JPA가 엔티티를 관리하는 공간입니다. 엔티티를 저장하거나 조회하면 영속성 컨텍스트에서 관리되고, 이를 통해 1차 캐시, 동일성 보장, 변경 감지 같은 기능을 사용할 수 있습니다.

## 7. 변경 감지 Dirty Checking

변경 감지는 트랜잭션 안에서 조회한 엔티티의 값이 변경되면, JPA가 이를 감지해서 자동으로 update 쿼리를 실행하는 기능이다.

예시:

```java
@Transactional
public void changeName(Long id, String name) {
    Member member = memberRepository.findById(id)
        .orElseThrow();

    member.changeName(name);
}
```

위 코드에서는 `save()`를 다시 호출하지 않는다.

흐름:

```text
트랜잭션 시작
→ 엔티티 조회
→ 영속성 컨텍스트가 엔티티 관리
→ 엔티티 값 변경
→ 트랜잭션 커밋
→ 변경 감지
→ update 쿼리 실행
```

면접 답변:

> 변경 감지는 트랜잭션 안에서 조회한 영속 상태의 엔티티 값이 변경되면, 트랜잭션 커밋 시점에 JPA가 변경 내용을 감지해서 자동으로 update 쿼리를 실행하는 기능입니다. 그래서 수정 작업에서는 보통 `save()`를 다시 호출하지 않아도 됩니다.

## 8. 지연 로딩과 즉시 로딩

### 지연 로딩 Lazy Loading

지연 로딩은 연관된 엔티티를 실제로 사용할 때 조회하는 방식이다.

```java
@ManyToOne(fetch = FetchType.LAZY)
private Team team;
```

처음 Member를 조회할 때 Team을 바로 조회하지 않고, `member.getTeam()`처럼 실제로 사용할 때 Team을 조회한다.

### 즉시 로딩 Eager Loading

즉시 로딩은 엔티티를 조회할 때 연관된 엔티티도 함께 조회하는 방식이다.

```java
@ManyToOne(fetch = FetchType.EAGER)
private Team team;
```

즉시 로딩은 예상하지 못한 쿼리가 많이 나갈 수 있어서 실무에서는 지연 로딩을 기본으로 사용하는 경우가 많다.

면접 답변:

> 지연 로딩은 연관 엔티티를 실제 사용할 때 조회하는 방식이고, 즉시 로딩은 엔티티를 조회할 때 연관 엔티티까지 바로 조회하는 방식입니다. 즉시 로딩은 예상하지 못한 쿼리를 발생시킬 수 있어 실무에서는 지연 로딩을 기본으로 사용하는 경우가 많습니다.

## 9. N+1 문제

N+1 문제는 하나의 쿼리로 N개의 데이터를 조회한 뒤, 연관된 데이터를 조회하기 위해 추가 쿼리가 N번 더 발생하는 문제이다.

예를 들어 회원 목록을 조회한 뒤 각 회원의 팀 정보를 사용할 때, 회원 수만큼 팀 조회 쿼리가 추가로 나갈 수 있다.

```text
회원 목록 조회 1번
→ 각 회원의 팀 조회 N번
→ 총 1 + N번 쿼리 발생
```

해결 방법으로는 fetch join, EntityGraph, batch size 설정 등을 사용할 수 있다.

면접 답변:

> N+1 문제는 연관 관계가 있는 엔티티를 조회할 때, 처음 조회 쿼리 1번 이후 연관 데이터를 가져오기 위한 쿼리가 N번 추가로 발생하는 문제입니다. 보통 fetch join이나 EntityGraph, batch size 설정 등을 통해 해결할 수 있습니다.

## 10. JPQL

JPQL은 Java Persistence Query Language의 약자이다.

SQL과 비슷하지만, 테이블이 아니라 Entity 객체와 필드를 기준으로 작성한다.

```java
@Query("select m from Member m where m.email = :email")
Optional<Member> findByEmail(@Param("email") String email);
```

여기서 `Member`는 테이블명이 아니라 Entity 이름이고, `m.email`은 컬럼명이 아니라 Entity 필드명이다.

면접 답변:

> JPQL은 JPA에서 사용하는 객체 지향 쿼리 언어입니다. SQL처럼 보이지만 테이블과 컬럼이 아니라 Entity와 필드명을 기준으로 작성합니다.

## 11. 한 번에 외우는 면접 답변

> JPA는 Java 객체와 관계형 데이터베이스 테이블을 매핑해서 객체 중심으로 DB를 다룰 수 있게 해주는 ORM 표준 기술입니다. JPA를 사용하면 반복적인 SQL 작성을 줄이고, Entity 중심으로 데이터를 저장하고 조회할 수 있습니다. JPA는 표준 인터페이스이고, Hibernate는 대표적인 구현체이며, Spring Data JPA는 JPA를 편하게 사용할 수 있도록 Repository와 기본 CRUD 기능을 제공하는 Spring 프로젝트입니다. JPA의 핵심 개념으로는 Entity, 영속성 컨텍스트, 변경 감지, 지연 로딩, JPQL 등이 있습니다. 특히 트랜잭션 안에서 조회한 엔티티를 수정하면 변경 감지로 update 쿼리가 자동 실행됩니다.
