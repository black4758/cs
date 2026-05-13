# Repository 정리

## 1. Repository란?

Repository는 데이터베이스에 접근하는 계층이다.

DB와 관련된 조회, 저장, 수정, 삭제 작업을 담당한다.

Spring Data JPA에서는 보통 `JpaRepository`를 상속한 인터페이스로 만든다.

```java
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;

@Repository
public interface MemberRepository extends JpaRepository<Member, Long> {
}
```

의미:

- `MemberRepository`: Member 엔티티의 DB 작업을 담당하는 Repository
- `JpaRepository<Member, Long>`: Member 엔티티를 다루고, 기본키 타입은 Long
- `@Repository`: 데이터 접근 계층 Bean으로 등록

면접 답변:

> Repository는 데이터 접근 계층으로, DB와 관련된 조회, 저장, 수정, 삭제 작업을 담당합니다. Spring Data JPA에서는 `JpaRepository`를 상속한 인터페이스를 만들면 기본적인 CRUD 메서드를 사용할 수 있습니다.

## 2. JpaRepository 기본 메서드

`JpaRepository`를 상속하면 기본 CRUD 메서드를 사용할 수 있다.

```java
memberRepository.save(member);        // 저장
memberRepository.findById(id);        // id로 단건 조회
memberRepository.findAll();           // 전체 조회
memberRepository.delete(member);      // 삭제
memberRepository.existsById(id);      // 존재 여부 확인
memberRepository.count();             // 개수 조회
```

예시:

```java
@Service
public class MemberService {

    private final MemberRepository memberRepository;

    public MemberService(MemberRepository memberRepository) {
        this.memberRepository = memberRepository;
    }

    public Member save(Member member) {
        return memberRepository.save(member);
    }
}
```

## 3. 쿼리 메서드

Spring Data JPA는 메서드 이름을 보고 쿼리를 자동으로 만들어준다.

예시:

```java
public interface MemberRepository extends JpaRepository<Member, Long> {

    Optional<Member> findByEmail(String email);

    List<Member> findByName(String name);

    boolean existsByEmail(String email);
}
```

의미:

- `findByEmail`: email 컬럼으로 회원 조회
- `findByName`: name 컬럼으로 회원 목록 조회
- `existsByEmail`: email 값이 존재하는지 확인

면접 답변:

> Spring Data JPA는 메서드 이름 규칙을 기반으로 쿼리를 자동 생성해줍니다. 예를 들어 `findByEmail`이라고 작성하면 email 값을 기준으로 조회하는 쿼리가 만들어집니다.

## 4. @Query

`@Query`는 직접 JPQL이나 SQL을 작성하고 싶을 때 사용한다.

쿼리 메서드 이름만으로 표현하기 어렵거나, 복잡한 조회 조건이 필요할 때 사용한다.

### JPQL 사용 예시

```java
public interface MemberRepository extends JpaRepository<Member, Long> {

    @Query("select m from Member m where m.email = :email")
    Optional<Member> findMemberByEmail(@Param("email") String email);
}
```

설명:

- `Member`는 DB 테이블명이 아니라 Entity 이름이다.
- `m.email`은 테이블 컬럼명이 아니라 Entity 필드명이다.
- `:email`은 파라미터 바인딩이다.
- `@Param("email")`로 메서드 파라미터와 쿼리 파라미터를 연결한다.

면접 답변:

> `@Query`는 Repository 메서드에 직접 JPQL이나 SQL을 작성할 때 사용하는 어노테이션입니다. 쿼리 메서드 이름으로 표현하기 어려운 복잡한 조회 조건이 있을 때 사용합니다.

### 여러 조건 조회 예시

```java
public interface MemberRepository extends JpaRepository<Member, Long> {

    @Query("select m from Member m where m.name = :name and m.email = :email")
    Optional<Member> findByNameAndEmail(
        @Param("name") String name,
        @Param("email") String email
    );
}
```

### 특정 컬럼만 조회하는 예시

```java
public interface MemberRepository extends JpaRepository<Member, Long> {

    @Query("select m.email from Member m where m.id = :id")
    Optional<String> findEmailById(@Param("id") Long id);
}
```

## 5. Native Query

`@Query`에서 실제 SQL을 작성하고 싶으면 `nativeQuery = true`를 사용한다.

```java
public interface MemberRepository extends JpaRepository<Member, Long> {

    @Query(value = "select * from members where email = :email", nativeQuery = true)
    Optional<Member> findByEmailNative(@Param("email") String email);
}
```

설명:

- JPQL은 Entity와 필드 기준으로 작성한다.
- Native Query는 실제 DB 테이블과 컬럼 기준으로 작성한다.

정리:

```text
JPQL: select m from Member m where m.email = :email
SQL:  select * from members where email = :email
```

면접 답변:

> `@Query`는 기본적으로 JPQL을 사용하지만, `nativeQuery = true`를 설정하면 실제 SQL을 작성할 수 있습니다. JPQL은 Entity와 필드 기준으로 작성하고, Native Query는 실제 DB 테이블과 컬럼 기준으로 작성합니다.

## 6. @Modifying

`@Query`로 수정이나 삭제 쿼리를 작성할 때는 `@Modifying`을 함께 사용한다.

```java
public interface MemberRepository extends JpaRepository<Member, Long> {

    @Modifying
    @Query("update Member m set m.name = :name where m.id = :id")
    int updateName(@Param("id") Long id, @Param("name") String name);
}
```

수정, 삭제 쿼리는 조회 쿼리가 아니기 때문에 `@Modifying`을 붙여야 한다.

또한 보통 서비스 계층에서 `@Transactional`과 함께 사용한다.

```java
@Transactional
public void changeName(Long id, String name) {
    memberRepository.updateName(id, name);
}
```

면접 답변:

> `@Modifying`은 `@Query`로 update나 delete 같은 변경 쿼리를 실행할 때 사용하는 어노테이션입니다. 변경 쿼리는 보통 트랜잭션 안에서 실행해야 하므로 서비스 계층에서 `@Transactional`과 함께 사용합니다.

## 7. Repository에서 @Repository는 필수인가?

Spring Data JPA에서 `JpaRepository`를 상속한 인터페이스는 Spring이 자동으로 Bean으로 등록해준다.

그래서 실무에서는 `@Repository`를 생략하는 경우도 많다.

```java
public interface MemberRepository extends JpaRepository<Member, Long> {
}
```

하지만 학습이나 명확한 계층 표현을 위해 붙여도 된다.

면접 답변:

> Spring Data JPA에서 `JpaRepository`를 상속하면 Spring이 Repository를 자동으로 Bean으로 등록하기 때문에 `@Repository`를 생략할 수 있습니다. 다만 데이터 접근 계층이라는 의미를 명확히 하기 위해 붙일 수도 있습니다.

## 8. 한 번에 외우는 면접 답변

> Repository는 데이터베이스에 접근하는 계층으로, 조회, 저장, 수정, 삭제 같은 DB 작업을 담당합니다. Spring Data JPA에서는 `JpaRepository<Entity, ID 타입>`을 상속하면 기본 CRUD 메서드를 사용할 수 있습니다. 또한 메서드 이름 규칙으로 쿼리를 자동 생성할 수 있고, 복잡한 쿼리가 필요할 때는 `@Query`를 사용해 JPQL이나 Native SQL을 직접 작성할 수 있습니다. 수정이나 삭제 쿼리를 작성할 때는 `@Modifying`을 함께 사용합니다.
