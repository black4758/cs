# Auth 정리

## 1. 전체 흐름

회원가입과 로그인, JWT 인증 흐름은 다음 순서로 이해하면 좋다.

```text
1. 회원가입
   → 비밀번호 암호화
   → Member 저장

2. 로그인
   → 이메일로 회원 조회
   → 비밀번호 검증
   → Access Token, Refresh Token 발급

3. 사용자가 API 요청
   → Authorization 헤더에 Access Token 포함
   → JWT 필터가 토큰 검증
   → SecurityContext에 인증 정보 저장

4. 로그인 여부 판단
   → SecurityConfig의 인가 규칙 확인
   → permitAll이면 통과
   → authenticated면 인증 정보가 있어야 통과

5. API에서 사용자 정보 사용
   → @AuthenticationPrincipal로 로그인 사용자 정보 꺼내기
```

면접 답변:

> Spring Security와 JWT를 사용하면, 회원가입 시 비밀번호를 암호화해서 저장하고 로그인 시 비밀번호를 검증한 뒤 Access Token과 Refresh Token을 발급합니다. 이후 클라이언트는 Access Token을 Authorization 헤더에 담아 요청하고, 서버의 JWT 필터는 토큰을 검증해 인증 정보를 SecurityContext에 저장합니다. SecurityConfig는 URL별 접근 권한을 판단하고, Controller에서는 `@AuthenticationPrincipal`로 로그인 사용자 정보를 사용할 수 있습니다.

## 2. 패키지 구조

```text
src/main/java/com/example/demo
├── member
│   ├── controller
│   │   └── MemberController.java
│   ├── service
│   │   └── MemberService.java
│   ├── repository
│   │   └── MemberRepository.java
│   ├── entity
│   │   └── Member.java
│   └── dto
│       └── MemberCreateRequest.java
├── auth
│   ├── controller
│   │   └── AuthController.java
│   ├── service
│   │   └── AuthService.java
│   ├── dto
│   │   ├── LoginRequest.java
│   │   └── TokenResponse.java
│   ├── jwt
│   │   ├── JwtProvider.java
│   │   └── JwtAuthenticationFilter.java
│   └── security
│       ├── CustomUserDetails.java
│       └── CustomUserDetailsService.java
└── config
    └── SecurityConfig.java
```

## 3. 회원가입

회원가입은 사용자의 정보를 받아 DB에 회원을 저장하는 기능이다.

중요한 점은 비밀번호를 원문 그대로 저장하지 않고, `PasswordEncoder`로 암호화해서 저장하는 것이다.

흐름:

```text
POST /members
→ MemberController
→ MemberService
→ 이메일 중복 확인
→ 비밀번호 암호화
→ Member 엔티티 생성
→ MemberRepository.save()
→ DB 저장
```

### MemberCreateRequest.java

```java
@Getter
@NoArgsConstructor
public class MemberCreateRequest {

    private String email;
    private String password;
    private String name;
}
```

어노테이션 설명:

- `@Getter`: 요청 DTO의 값을 꺼내기 위한 getter를 만든다.
- `@NoArgsConstructor`: Jackson이 JSON 요청을 DTO 객체로 변환할 때 사용할 기본 생성자를 만든다.

### Member.java

```java
@Entity
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class Member {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String email;

    // 원문 비밀번호가 아니라 암호화된 비밀번호를 저장한다.
    private String password;

    private String name;

    private String role;

    private Member(String email, String password, String name, String role) {
        this.email = email;
        this.password = password;
        this.name = name;
        this.role = role;
    }

    public static Member create(String email, String encodedPassword, String name) {
        return new Member(email, encodedPassword, name, "ROLE_USER");
    }
}
```

어노테이션 설명:

- `@Entity`: JPA가 관리하는 엔티티로 등록한다.
- `@Getter`: 필드 값을 조회할 getter를 만든다.
- `@NoArgsConstructor(access = AccessLevel.PROTECTED)`: JPA 기본 생성자를 만들고, 외부에서 빈 객체를 무분별하게 생성하지 못하게 한다.
- `@Id`: 기본키를 의미한다.
- `@GeneratedValue`: 기본키 자동 생성 전략을 지정한다.

`create()`를 쓰는 이유:

- 생성 의미를 명확하게 표현할 수 있다.
- 기본 권한인 `ROLE_USER` 같은 생성 규칙을 한 곳에서 관리할 수 있다.
- 생성자를 `private`로 막아 정해진 방식으로만 객체를 만들 수 있다.

### MemberRepository.java

```java
public interface MemberRepository extends JpaRepository<Member, Long> {

    Optional<Member> findByEmail(String email);

    boolean existsByEmail(String email);
}
```

설명:

- `JpaRepository<Member, Long>`: `Member` 엔티티를 다루고, 기본키 타입은 `Long`이다.
- `findByEmail`: 로그인할 때 이메일로 회원을 조회한다.
- `existsByEmail`: 회원가입할 때 이메일 중복 여부를 확인한다.
- `Optional`: 조회 결과가 없을 수 있음을 표현한다.

### MemberService.java

```java
@Service
@RequiredArgsConstructor
@Transactional(readOnly = true)
public class MemberService {

    private final MemberRepository memberRepository;
    private final PasswordEncoder passwordEncoder;

    @Transactional
    public void join(MemberCreateRequest request) {
        // 이미 가입된 이메일인지 확인한다.
        if (memberRepository.existsByEmail(request.getEmail())) {
            throw new IllegalArgumentException("이미 존재하는 이메일입니다.");
        }

        // 원문 비밀번호를 BCrypt 해시로 암호화한다.
        String encodedPassword = passwordEncoder.encode(request.getPassword());

        // 암호화된 비밀번호로 Member 엔티티를 생성한다.
        Member member = Member.create(
            request.getEmail(),
            encodedPassword,
            request.getName()
        );

        // 새 엔티티를 DB에 저장한다.
        memberRepository.save(member);
    }
}
```

어노테이션 설명:

- `@Service`: 비즈니스 로직을 처리하는 Service 계층으로 등록한다.
- `@RequiredArgsConstructor`: `final` 필드 생성자를 자동 생성한다. 실제 주입은 Spring 컨테이너가 생성자를 통해 해준다.
- `@Transactional(readOnly = true)`: 기본적으로 조회 전용 트랜잭션을 적용한다.
- `@Transactional`: 저장, 수정, 삭제처럼 DB 변경이 필요한 메서드에 사용한다.

비밀번호 처리:

```java
passwordEncoder.encode(rawPassword);
```

회원가입 시 원문 비밀번호를 암호화한다.

### MemberController.java

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
}
```

어노테이션 설명:

- `@RestController`: REST API 컨트롤러로 등록하고, 반환값을 HTTP Body에 담아 응답한다.
- `@RequiredArgsConstructor`: `MemberService` 생성자를 자동 생성한다.
- `@RequestMapping("/members")`: 공통 URL을 `/members`로 지정한다.
- `@PostMapping`: HTTP POST 요청을 처리한다.
- `@RequestBody`: 요청 body의 JSON을 DTO로 변환해 받는다.

## 4. 로그인

로그인은 사용자가 입력한 이메일과 비밀번호를 검증하고, 성공하면 토큰을 발급하는 기능이다.

흐름:

```text
POST /auth/login
→ AuthController
→ AuthService
→ 이메일로 회원 조회
→ PasswordEncoder.matches()로 비밀번호 검증
→ Access Token 생성
→ Refresh Token 생성
→ TokenResponse 반환
```

### LoginRequest.java

```java
@Getter
@NoArgsConstructor
public class LoginRequest {

    private String email;
    private String password;
}
```

### TokenResponse.java

```java
@Getter
@AllArgsConstructor
public class TokenResponse {

    private String accessToken;
    private String refreshToken;
}
```

설명:

- `accessToken`: API 요청 인증에 사용한다.
- `refreshToken`: Access Token 재발급에 사용한다.

### AuthController.java

```java
@RestController
@RequiredArgsConstructor
@RequestMapping("/auth")
public class AuthController {

    private final AuthService authService;

    @PostMapping("/login")
    public TokenResponse login(@RequestBody LoginRequest request) {
        return authService.login(request);
    }
}
```

설명:

- `/auth/login` 요청을 받는다.
- 로그인 검증은 Controller가 아니라 Service에서 처리한다.
- Controller는 요청 DTO를 받고 Service를 호출한 뒤 응답 DTO를 반환한다.

### AuthService.java

```java
@Service
@RequiredArgsConstructor
public class AuthService {

    private final MemberRepository memberRepository;
    private final PasswordEncoder passwordEncoder;
    private final JwtProvider jwtProvider;

    public TokenResponse login(LoginRequest request) {
        // 이메일로 회원을 조회한다.
        Member member = memberRepository.findByEmail(request.getEmail())
            .orElseThrow(() -> new IllegalArgumentException("존재하지 않는 회원입니다."));

        // 사용자가 입력한 비밀번호와 DB에 저장된 암호화 비밀번호를 비교한다.
        if (!passwordEncoder.matches(request.getPassword(), member.getPassword())) {
            throw new IllegalArgumentException("비밀번호가 일치하지 않습니다.");
        }

        // 인증 성공 시 Access Token과 Refresh Token을 발급한다.
        String accessToken = jwtProvider.createAccessToken(member.getId(), member.getRole());
        String refreshToken = jwtProvider.createRefreshToken(member.getId());

        return new TokenResponse(accessToken, refreshToken);
    }
}
```

비밀번호 검증:

```java
passwordEncoder.matches(rawPassword, encodedPassword);
```

로그인할 때는 입력 비밀번호와 DB 저장 비밀번호를 직접 문자열 비교하지 않고 `matches()`를 사용한다.

## 5. 토큰

토큰은 로그인한 사용자를 식별하기 위해 클라이언트와 서버가 주고받는 문자열이다.

로그인 성공 시 보통 두 개의 토큰을 발급한다.

```text
Access Token
Refresh Token
```

### Access Token

Access Token은 API 요청 인증에 사용하는 토큰이다.

특징:

- 수명이 짧다.
- API 요청마다 Authorization 헤더에 담아 보낸다.
- 서버는 Access Token을 검증해 사용자를 인증한다.

요청 예시:

```http
GET /members/me
Authorization: Bearer eyJhbGciOi...
```

### Refresh Token

Refresh Token은 Access Token이 만료되었을 때 새 Access Token을 발급받기 위한 토큰이다.

특징:

- 수명이 길다.
- Access Token 재발급에 사용한다.
- DB나 Redis에 저장해서 관리하는 경우가 많다.

재발급 흐름:

```text
Access Token 만료
→ 서버가 401 Unauthorized 응답
→ 프론트가 Refresh Token으로 재발급 요청
→ 서버가 Refresh Token 검증
→ 새 Access Token 발급
```

### JWT에 담는 정보

JWT에는 보통 인증에 필요한 최소 정보만 담는다.

예시 payload:

```json
{
  "sub": "1",
  "role": "ROLE_USER",
  "iat": 1710000000,
  "exp": 1710003600
}
```

의미:

- `sub`: 토큰의 주인, 보통 memberId를 저장한다.
- `role`: 사용자 권한이다.
- `iat`: 발급 시간이다.
- `exp`: 만료 시간이다.

주의:

JWT는 쉽게 디코딩할 수 있으므로 비밀번호, 주민번호, 카드 번호 같은 민감한 정보는 넣으면 안 된다.

## 6. JwtProvider.java

`JwtProvider`는 JWT를 생성하고 검증하고, 토큰 안의 정보를 꺼내는 클래스이다.

```java
@Component
public class JwtProvider {

    private final String secretKey = "secret-key-example";
    private final long accessTokenExpiration = 1000 * 60 * 30; // 30분
    private final long refreshTokenExpiration = 1000 * 60 * 60 * 24 * 7; // 7일

    public String createAccessToken(Long memberId, String role) {
        Date now = new Date();
        Date expiry = new Date(now.getTime() + accessTokenExpiration);

        return Jwts.builder()
            .setSubject(String.valueOf(memberId)) // JWT의 sub 값에 memberId 저장
            .claim("role", role)                  // 권한 정보 저장
            .setIssuedAt(now)                     // 발급 시간
            .setExpiration(expiry)                // 만료 시간
            .signWith(SignatureAlgorithm.HS256, secretKey) // 서명
            .compact();
    }

    public String createRefreshToken(Long memberId) {
        Date now = new Date();
        Date expiry = new Date(now.getTime() + refreshTokenExpiration);

        return Jwts.builder()
            .setSubject(String.valueOf(memberId))
            .setIssuedAt(now)
            .setExpiration(expiry)
            .signWith(SignatureAlgorithm.HS256, secretKey)
            .compact();
    }

    public Long getMemberId(String token) {
        Claims claims = Jwts.parser()
            .setSigningKey(secretKey)
            .parseClaimsJws(token)
            .getBody();

        return Long.valueOf(claims.getSubject());
    }

    public String getRole(String token) {
        Claims claims = Jwts.parser()
            .setSigningKey(secretKey)
            .parseClaimsJws(token)
            .getBody();

        return claims.get("role", String.class);
    }

    public boolean validateToken(String token) {
        try {
            Jwts.parser()
                .setSigningKey(secretKey)
                .parseClaimsJws(token);
            return true;
        } catch (JwtException | IllegalArgumentException e) {
            return false;
        }
    }
}
```

필요 import:

```java
import io.jsonwebtoken.Claims;
import io.jsonwebtoken.JwtException;
import io.jsonwebtoken.Jwts;
import io.jsonwebtoken.SignatureAlgorithm;
import org.springframework.stereotype.Component;

import java.util.Date;
```

Gradle 의존성 예시:

```gradle
implementation 'io.jsonwebtoken:jjwt-api:0.11.5'
runtimeOnly 'io.jsonwebtoken:jjwt-impl:0.11.5'
runtimeOnly 'io.jsonwebtoken:jjwt-jackson:0.11.5'
```

설명:

- `createAccessToken`: Access Token을 만든다.
- `createRefreshToken`: Refresh Token을 만든다.
- `setSubject`: JWT의 `sub` 값에 memberId를 저장한다.
- `getMemberId`: JWT의 `sub` 값에서 memberId를 꺼낸다.
- `validateToken`: 토큰의 서명과 만료 여부를 검증한다.

주의:

실무에서는 `secretKey`를 코드에 직접 쓰지 않고 `application.yml` 같은 설정 파일에서 주입받는다.

## 7. 사용자가 API 요청을 보냈을 때

로그인 후 클라이언트는 API 요청마다 Access Token을 Authorization 헤더에 담아 보낸다.

요청 예시:

```http
GET /members/me
Authorization: Bearer eyJhbGciOi...
```

전체 흐름:

```text
사용자가 API 요청
→ Spring Security Filter Chain 진입
→ JwtAuthenticationFilter 실행
→ Authorization 헤더에서 Access Token 추출
→ JwtProvider로 토큰 검증
→ 토큰의 sub 값에서 memberId 추출
→ memberId로 사용자 정보 조회
→ Authentication 객체 생성
→ SecurityContext에 저장
→ SecurityConfig의 인가 규칙 확인
→ 인증된 요청이면 Controller로 이동
```

핵심:

- JWT 필터는 토큰을 검증하고 인증 정보를 만든다.
- SecurityConfig는 해당 API가 인증이 필요한지 판단한다.
- SecurityContext에 인증 정보가 있으면 로그인한 사용자로 처리된다.

## 8. JwtAuthenticationFilter.java

`JwtAuthenticationFilter`는 요청이 Controller에 도착하기 전에 JWT를 검사하는 필터이다.

역할:

```text
Authorization 헤더 확인
→ Bearer 토큰 추출
→ 토큰 검증
→ memberId 추출
→ 사용자 정보 조회
→ Authentication 객체 생성
→ SecurityContext에 저장
```

코드:

```java
@RequiredArgsConstructor
public class JwtAuthenticationFilter extends OncePerRequestFilter {

    private final JwtProvider jwtProvider;
    private final CustomUserDetailsService customUserDetailsService;

    @Override
    protected void doFilterInternal(
        HttpServletRequest request,
        HttpServletResponse response,
        FilterChain filterChain
    ) throws ServletException, IOException {

        // Authorization 헤더에서 토큰을 꺼낸다.
        String authorization = request.getHeader("Authorization");

        // Bearer 토큰 형식인지 확인한다.
        if (authorization != null && authorization.startsWith("Bearer ")) {

            // "Bearer "를 제거하고 순수 JWT만 꺼낸다.
            String token = authorization.substring(7);

            // 토큰의 서명과 만료 시간을 검증한다.
            if (jwtProvider.validateToken(token)) {

                // 토큰의 subject에서 memberId를 꺼낸다.
                Long memberId = jwtProvider.getMemberId(token);

                // memberId로 사용자 정보를 조회하고 UserDetails 객체로 만든다.
                CustomUserDetails userDetails =
                    customUserDetailsService.loadUserByMemberId(memberId);

                // Spring Security가 이해할 수 있는 인증 객체를 만든다.
                Authentication authentication =
                    new UsernamePasswordAuthenticationToken(
                        userDetails,                  // principal, 현재 로그인 사용자 정보
                        null,                         // credentials, JWT 인증에서는 비밀번호 불필요
                        userDetails.getAuthorities()  // 권한 정보
                    );

                // SecurityContext에 인증 정보를 저장한다.
                SecurityContextHolder.getContext().setAuthentication(authentication);
            }
        }

        // 다음 필터로 요청을 넘긴다.
        filterChain.doFilter(request, response);
    }
}
```

설명:

- `OncePerRequestFilter`: 요청당 한 번만 실행되는 필터이다.
- `Authorization`: Access Token이 들어오는 HTTP 헤더이다.
- `Bearer`: 토큰 인증 방식에서 사용하는 접두어이다.
- `Authentication`: 현재 사용자의 인증 정보를 나타내는 객체이다.
- `SecurityContext`: 인증된 사용자 정보를 저장하는 공간이다.

중요한 코드:

```java
SecurityContextHolder.getContext().setAuthentication(authentication);
```

이 코드가 실행되면 Spring Security는 해당 요청을 인증된 사용자 요청으로 판단할 수 있다.

## 9. 로그인 여부를 어디서 판단하는가?

로그인 여부와 URL별 접근 권한은 `SecurityConfig`에서 설정한다.

정확히는 `SecurityConfig`가 규칙을 설정하고, 실제 처리는 Spring Security Filter Chain에서 이루어진다.

예시:

```java
@Configuration
@RequiredArgsConstructor
@EnableWebSecurity
public class SecurityConfig {

    private final JwtProvider jwtProvider;
    private final CustomUserDetailsService customUserDetailsService;

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        // JWT를 검사할 커스텀 필터를 만든다.
        JwtAuthenticationFilter jwtAuthenticationFilter =
            new JwtAuthenticationFilter(jwtProvider, customUserDetailsService);

        http
            // JWT 기반 REST API에서는 서버 세션을 사용하지 않기 때문에 CSRF를 보통 비활성화한다.
            .csrf(csrf -> csrf.disable())

            // 세션을 만들지 않고, 매 요청마다 토큰으로 인증한다.
            .sessionManagement(session ->
                session.sessionCreationPolicy(SessionCreationPolicy.STATELESS)
            )

            // URL별 접근 권한을 설정한다.
            .authorizeHttpRequests(auth -> auth
                // permitAll(): 인증 없이 접근 허용
                .requestMatchers("/auth/login", "/members").permitAll()

                // 게시글 조회는 인증 없이 허용
                .requestMatchers(HttpMethod.GET, "/posts/**").permitAll()

                // authenticated(): 인증된 사용자만 접근 허용
                .anyRequest().authenticated()
            )

            // JWT 필터를 UsernamePasswordAuthenticationFilter보다 먼저 실행한다.
            .addFilterBefore(jwtAuthenticationFilter, UsernamePasswordAuthenticationFilter.class);

        return http.build();
    }
}
```

설정 의미:

```java
.requestMatchers("/auth/login", "/members").permitAll()
```

로그인과 회원가입은 인증 없이 접근할 수 있다.

```java
.requestMatchers(HttpMethod.GET, "/posts/**").permitAll()
```

게시글 조회는 비회원도 접근할 수 있다.

```java
.anyRequest().authenticated()
```

나머지 요청은 인증된 사용자만 접근할 수 있다.

예시:

```text
POST /auth/login → 허용
POST /members → 허용
GET /posts → 허용
GET /posts/1 → 허용
POST /posts → 인증 필요
PATCH /posts/1 → 인증 필요
DELETE /posts/1 → 인증 필요
GET /members/me → 인증 필요
```

비회원 요청 흐름:

```text
Authorization 헤더 없음
→ JwtAuthenticationFilter가 인증 정보를 만들지 못함
→ SecurityContext 비어 있음
→ permitAll API면 통과
→ authenticated API면 401 Unauthorized
```

로그인한 사용자 요청 흐름:

```text
Authorization 헤더에 Access Token 있음
→ JwtAuthenticationFilter가 토큰 검증
→ SecurityContext에 Authentication 저장
→ authenticated API 통과
```

정리:

- `permitAll()`: 로그인 없이 접근 가능하다.
- `authenticated()`: 로그인한 사용자만 접근 가능하다.
- JWT 필터: 토큰을 검증하고 인증 정보를 만든다.
- SecurityConfig: 어떤 API를 허용하고 막을지 정한다.

## 10. API에서 헤더의 사용자 정보를 꺼내 쓰는 방법

실무에서는 Controller가 직접 헤더에서 토큰을 파싱하지 않는 경우가 많다.

대신 JWT 필터가 토큰을 검증하고, `SecurityContext`에 인증 정보를 저장한 뒤 Controller에서는 `@AuthenticationPrincipal`로 사용자 정보를 받는다.

### CustomUserDetails.java

```java
@Getter
@AllArgsConstructor
public class CustomUserDetails implements UserDetails {

    private Long id;
    private String email;
    private String password;
    private Collection<? extends GrantedAuthority> authorities;

    @Override
    public String getUsername() {
        return email;
    }

    @Override
    public boolean isAccountNonExpired() {
        return true;
    }

    @Override
    public boolean isAccountNonLocked() {
        return true;
    }

    @Override
    public boolean isCredentialsNonExpired() {
        return true;
    }

    @Override
    public boolean isEnabled() {
        return true;
    }
}
```

### CustomUserDetailsService.java

```java
@Service
@RequiredArgsConstructor
public class CustomUserDetailsService {

    private final MemberRepository memberRepository;

    public CustomUserDetails loadUserByMemberId(Long memberId) {
        Member member = memberRepository.findById(memberId)
            .orElseThrow(() -> new IllegalArgumentException("존재하지 않는 회원입니다."));

        return new CustomUserDetails(
            member.getId(),
            member.getEmail(),
            member.getPassword(),
            List.of(new SimpleGrantedAuthority(member.getRole()))
        );
    }
}
```

### Controller에서 사용

```java
@GetMapping("/me")
public MemberResponse myInfo(
    @AuthenticationPrincipal CustomUserDetails userDetails
) {
    Long memberId = userDetails.getId();

    return memberService.findMember(memberId);
}
```

흐름:

```text
Authorization 헤더
→ JWT 필터가 토큰 검증
→ 토큰에서 memberId 추출
→ CustomUserDetails 생성
→ Authentication의 principal로 저장
→ SecurityContext 저장
→ @AuthenticationPrincipal로 Controller에 주입
```

직접 헤더에서 꺼내는 방식도 가능하다.

```java
@GetMapping("/me")
public MemberResponse myInfo(
    @RequestHeader("Authorization") String authorization
) {
    String token = authorization.substring(7);
    Long memberId = jwtProvider.getMemberId(token);

    return memberService.findMember(memberId);
}
```

하지만 이 방식은 Controller가 토큰 파싱까지 알아야 하므로, 실무에서는 보통 필터와 `@AuthenticationPrincipal`을 사용한다.

## 11. 401과 403

### 401 Unauthorized

인증되지 않은 요청이다.

발생 예시:

- Access Token이 없음
- Access Token이 만료됨
- Access Token이 잘못됨
- 로그인하지 않은 사용자가 보호된 API에 접근함

### 403 Forbidden

인증은 되었지만 권한이 없는 요청이다.

예시:

```text
USER 권한 사용자가 ADMIN API에 접근
```

정리:

```text
401 Unauthorized → 인증 안 됨
403 Forbidden → 인증은 됐지만 권한 없음
```

## 12. 한 번에 외우는 면접 답변

> Spring Security와 JWT 기반 인증에서는 회원가입 시 비밀번호를 `PasswordEncoder`로 암호화해서 저장하고, 로그인 시 `matches()`로 입력 비밀번호와 저장된 해시값을 비교합니다. 인증에 성공하면 Access Token과 Refresh Token을 발급합니다. 이후 클라이언트는 Access Token을 Authorization 헤더에 Bearer 형식으로 담아 요청합니다. JWT 필터는 헤더에서 토큰을 꺼내 검증하고, 토큰의 subject에서 memberId를 추출해 사용자 정보를 조회한 뒤 Authentication 객체를 SecurityContext에 저장합니다. URL별 접근 권한은 SecurityConfig에서 `permitAll()`과 `authenticated()`로 설정하며, Controller에서는 `@AuthenticationPrincipal`로 인증된 사용자 정보를 사용할 수 있습니다.
