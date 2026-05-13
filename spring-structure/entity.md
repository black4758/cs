# Entity 정리

## 1. Entity란?

Entity는 DB 테이블과 매핑되는 Java 객체이다.

JPA에서 `@Entity`가 붙은 클래스는 데이터베이스 테이블과 연결된다. 클래스는 테이블과 매핑되고, 클래스의 필드는 테이블의 컬럼과 매핑된다.

예시:

```text
Member 클래스 → members 테이블
id 필드 → id 컬럼
name 필드 → name 컬럼
email 필드 → email 컬럼
```

면접 답변:

> Entity는 JPA에서 데이터베이스 테이블과 매핑되는 Java 객체입니다. `@Entity`를 붙여 엔티티로 등록하고, `@Table`로 매핑할 테이블을 지정할 수 있습니다. 엔티티의 필드는 테이블의 컬럼과 매핑됩니다.

## 2. Entity 파일 위치

Entity는 보통 `domain` 또는 `entity` 패키지에 만든다.

예시:

```text
src/main/java/com/example/demo/domain/Member.java
```

또는

```text
src/main/java/com/example/demo/entity/Member.java
```

## 3. 기본 Entity 예시

```java
package com.example.demo.domain;

import jakarta.persistence.Entity;
import jakarta.persistence.GeneratedValue;
import jakarta.persistence.GenerationType;
import jakarta.persistence.Id;
import jakarta.persistence.Table;
import lombok.AccessLevel;
import lombok.Getter;
import lombok.NoArgsConstructor;

@Entity
@Table(name = "members")
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

## 4. Entity 설정 어노테이션

### @Entity

해당 클래스가 JPA Entity임을 나타낸다.

`@Entity`가 붙은 클래스는 JPA가 관리하며, 데이터베이스 테이블과 매핑된다.

```java
@Entity
public class Member {
}
```

면접 답변:

> `@Entity`는 해당 클래스가 JPA가 관리하는 엔티티임을 나타내는 어노테이션입니다. 이 클래스는 데이터베이스 테이블과 매핑됩니다.

### @Table

Entity와 매핑할 실제 DB 테이블 이름을 지정할 때 사용한다.

```java
@Entity
@Table(name = "members")
public class Member {
}
```

`@Table`은 필수는 아니지만, 테이블명을 명확하게 지정하기 위해 사용하는 경우가 많다.

면접 답변:

> `@Table`은 엔티티와 매핑할 실제 DB 테이블 이름을 지정할 때 사용합니다. 생략할 수도 있지만, 테이블명을 명확히 하기 위해 사용하는 경우가 많습니다.

### @Id

기본키를 의미한다.

Entity에는 반드시 식별자 역할을 하는 필드가 필요하고, 그 필드에 `@Id`를 붙인다.

```java
@Id
private Long id;
```

면접 답변:

> `@Id`는 엔티티의 기본키를 나타내는 어노테이션입니다.

### @GeneratedValue

기본키 값을 자동으로 생성할 때 사용한다.

```java
@Id
@GeneratedValue(strategy = GenerationType.IDENTITY)
private Long id;
```

`GenerationType.IDENTITY`는 DB의 auto increment 기능을 사용하는 방식이다.

면접 답변:

> `@GeneratedValue`는 기본키 값을 자동 생성할 때 사용합니다. `GenerationType.IDENTITY`는 데이터베이스의 auto increment를 사용하는 전략입니다.

### @Column

필드를 테이블 컬럼과 매핑할 때 사용한다.

```java
@Column(name = "user_name", nullable = false, length = 50)
private String name;
```

`@Column`도 필수는 아니다. 생략하면 필드 이름을 기준으로 컬럼과 매핑된다.

자주 쓰는 속성:

- `name`: 컬럼명 지정
- `nullable`: null 허용 여부
- `length`: 문자열 길이
- `unique`: 유니크 제약 조건

면접 답변:

> `@Column`은 엔티티 필드와 테이블 컬럼을 매핑할 때 사용합니다. 생략할 수 있지만, 컬럼명이나 제약 조건을 명확히 지정할 때 사용합니다.

## 5. 기본 생성자가 필요한 이유

JPA는 엔티티 객체를 만들 때 리플렉션을 사용한다.

그래서 Entity에는 기본 생성자가 필요하다.

```java
protected Member() {
}
```

기본 생성자는 `public` 또는 `protected`로 만들 수 있다. 보통 외부에서 직접 생성하는 것을 막기 위해 `protected`로 둔다.

면접 답변:

> JPA는 리플렉션을 통해 엔티티 객체를 생성하기 때문에 기본 생성자가 필요합니다. 보통 외부에서 직접 호출하지 못하도록 `protected` 기본 생성자를 사용합니다.

`@NoArgsConstructor`를 쓰지 않고 직접 기본 생성자를 작성할 수도 있다.

```java
@Entity
@Table(name = "members")
public class Member {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;

    private String email;

    protected Member() {
    }

    public Member(String name, String email) {
        this.name = name;
        this.email = email;
    }
}
```

정리:

- Lombok 사용: `@NoArgsConstructor(access = AccessLevel.PROTECTED)`
- Lombok 미사용: `protected Member() { }` 직접 작성

## 6. Entity에서 자주 쓰는 Lombok

Entity에서는 Lombok을 사용할 때 보통 `@Getter`와 `@NoArgsConstructor(access = AccessLevel.PROTECTED)`를 자주 사용한다.

```java
@Entity
@Table(name = "members")
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

### @Getter

`@Getter`는 필드의 getter 메서드를 자동으로 만들어준다.

Entity에서는 값을 조회할 일이 많기 때문에 `@Getter`를 자주 사용한다.

### @NoArgsConstructor(access = AccessLevel.PROTECTED)

`@NoArgsConstructor`는 기본 생성자를 자동으로 만들어준다.

JPA는 엔티티 객체를 생성할 때 기본 생성자가 필요하다. 하지만 외부에서 아무렇게나 기본 생성자를 호출하는 것은 막는 것이 좋기 때문에 접근 제어자를 `PROTECTED`로 둔다.

면접 답변:

> 엔티티에서는 Lombok의 `@Getter`와 `@NoArgsConstructor(access = AccessLevel.PROTECTED)`를 자주 사용합니다. JPA는 기본 생성자가 필요하기 때문에 `@NoArgsConstructor`를 사용하고, 외부에서 무분별하게 생성하지 못하도록 접근 수준을 `PROTECTED`로 둡니다.

## 7. Setter를 무조건 만들지 않는 이유

Entity에 setter를 무조건 열어두면 객체의 상태가 아무 곳에서나 변경될 수 있다.

그래서 필요한 경우에만 의미 있는 메서드를 만들어 상태를 변경하는 것이 좋다.

예시:

```java
public void changeName(String name) {
    this.name = name;
}
```

면접 답변:

> Entity에 setter를 무분별하게 만들면 객체의 상태가 어디서든 변경될 수 있어서 유지보수가 어려워질 수 있습니다. 그래서 필요한 경우 의미 있는 메서드를 통해 상태를 변경하는 방식이 좋습니다.

## 8. Entity에서 Builder를 쓰는 경우

Entity에서도 Builder를 사용할 수 있다. 하지만 클래스 위에 `@Builder`를 붙이는 방식은 조심해야 한다.

안 좋은 예시:

```java
@Entity
@Table(name = "members")
@Getter
@Builder
@AllArgsConstructor
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class Member {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;

    private String email;
}
```

위 방식은 `id`까지 Builder에 포함될 수 있다.

```java
Member member = Member.builder()
    .id(1L)
    .name("kim")
    .email("kim@test.com")
    .build();
```

하지만 `id`는 보통 DB가 자동 생성하고 관리해야 하는 값이다. 그래서 외부에서 `id`를 넣을 수 있게 만드는 것은 좋지 않다.

좋은 예시:

```java
@Entity
@Table(name = "members")
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class Member {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;

    private String email;

    @Builder
    public Member(String name, String email) {
        this.name = name;
        this.email = email;
    }
}
```

이렇게 필요한 필드만 받는 생성자 위에 `@Builder`를 붙이면 `id`는 Builder에 포함되지 않는다.

사용 예시:

```java
Member member = Member.builder()
    .name(request.getName())
    .email(request.getEmail())
    .build();

memberRepository.save(member);
```

정리:

- `@Builder`를 쓴다고 `@AllArgsConstructor`가 필수는 아니다.
- Entity에서는 `@AllArgsConstructor`를 조심해서 사용해야 한다.
- Entity에서 Builder를 쓴다면 클래스 위가 아니라 생성자 위에 붙이는 것이 안전하다.
- 생성자에는 `id`를 제외하고 필요한 비즈니스 필드만 넣는 것이 좋다.

면접 답변:

> Entity에서 Builder를 사용할 수는 있지만, 클래스 레벨에 `@Builder`를 붙이면 id처럼 DB가 관리해야 하는 값까지 외부에서 설정할 수 있어 조심해야 합니다. Builder를 사용한다면 `@AllArgsConstructor`를 함께 쓰기보다는 필요한 필드만 받는 생성자에 `@Builder`를 붙이는 방식이 더 안전합니다.

## 9. 한 번에 외우는 면접 답변

> Entity는 DB 테이블과 매핑되는 Java 객체입니다. 클래스에는 `@Entity`를 붙이고, 실제 테이블명을 지정할 때는 `@Table`을 사용합니다. 기본키 필드에는 `@Id`를 붙이고, 자동 생성을 사용할 경우 `@GeneratedValue`를 사용합니다. JPA는 리플렉션으로 객체를 생성하기 때문에 기본 생성자가 필요하며, Lombok을 사용하면 `@NoArgsConstructor(access = AccessLevel.PROTECTED)`를 자주 사용합니다. 또한 값을 조회하기 위해 `@Getter`를 사용하고, setter를 무분별하게 열기보다는 의미 있는 메서드로 상태를 변경하는 것이 좋습니다.
