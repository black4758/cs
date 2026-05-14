# Auth 정리

## 1. 회원가입과 로그인을 같이 정리하는 이유

회원가입과 로그인은 따로 떨어진 기능처럼 보이지만 실제 흐름은 연결되어 있다.

```text
회원가입
→ 비밀번호 암호화 저장
→ 로그인
→ 비밀번호 검증
→ Access Token, Refresh Token 발급
→ 이후 요청에서 Access Token 사용
```

그래서 한 파일에 정리하되, 내부 섹션을 회원가입, 로그인, JWT, 필터, 토큰으로 나누는 것이 좋다.

## 2. 전체 패키지 구조

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

## 3. 회원가입 흐름

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

회원가입에서 중요한 점은 비밀번호를 원문 그대로 저장하지 않는 것이다.

```text
raw password
→ PasswordEncoder.encode()
→ encoded password DB 저장
```

## 4. MemberCreateRequest.java

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

- `@Getter`: 요청 DTO의 값을 꺼내기 위한 getter를 생성한다.
- `@NoArgsConstructor`: Jackson이 JSON 요청을 DTO 객체로 변환할 때 사용할 기본 생성자를 만든다.

## 5. Member.java

```java
@Entity
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class Member {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String email;

    // 원문 비밀번호가 아니라 암호화된 비밀번호가 저장된다.
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
- `@Getter`: 필드 값을 조회할 getter를 생성한다.
- `@NoArgsConstructor(access = AccessLevel.PROTECTED)`: JPA가 사용할 기본 생성자를 만들고, 외부에서 무분별하게 빈 객체를 생성하지 못하게 한다.
- `@Id`: 기본키를 의미한다.
- `@GeneratedValue`: 기본키 자동 생성 전략을 지정한다.

`create()`를 쓰는 이유:

- 생성 의미를 명확하게 표현할 수 있다.
- 기본 권한인 `ROLE_USER` 같은 생성 규칙을 한 곳에서 관리할 수 있다.
- 생성자를 `private`로 막아 정해진 방식으로만 객체를 만들 수 있다.

## 6. MemberRepository.java

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

## 7. MemberService.java

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

        // 새 엔티티 저장은 save()를 사용한다.
        memberRepository.save(member);
    }
}
```

어노테이션 설명:

- `@Service`: 비즈니스 로직을 처리하는 Service 계층으로 등록한다.
- `@RequiredArgsConstructor`: `final` 필드에 대한 생성자를 자동 생성해서 생성자 주입을 가능하게 한다.
- `@Transactional(readOnly = true)`: 기본적으로 조회 전용 트랜잭션을 적용한다.
- `@Transactional`: 저장, 수정, 삭제처럼 DB 변경이 필요한 메서드에 사용한다.

비밀번호 처리:

```java
passwordEncoder.encode(rawPassword);
```

회원가입 시 원문 비밀번호를 암호화한다.

```java
passwordEncoder.matches(rawPassword, encodedPassword);
```

로그인 시 입력 비밀번호와 DB에 저장된 암호화 비밀번호를 비교한다.

## 8. MemberController.java

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

- `@RestController`: REST API 컨트롤러로 등록하고, 반환값을 JSON 응답으로 처리한다.
- `@RequiredArgsConstructor`: `MemberService`를 생성자 주입한다.
- `@RequestMapping("/members")`: 이 컨트롤러의 공통 URL을 `/members`로 지정한다.
- `@PostMapping`: HTTP POST 요청을 처리한다.
- `@RequestBody`: 요청 body의 JSON을 DTO로 변환해 받는다.

## 9. 로그인 흐름

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

로그인 성공 시 보통 두 개의 토큰을 발급한다.

- Access Token
- Refresh Token

## 10. LoginRequest.java

```java
@Getter
@NoArgsConstructor
public class LoginRequest {

    private String email;
    private String password;
}
```

설명:

- 로그인 요청으로 email과 password를 받는다.
- Request DTO이므로 `@Getter`, `@NoArgsConstructor`를 자주 사용한다.

## 11. TokenResponse.java

```java
@Getter
@AllArgsConstructor
public class TokenResponse {

    private String accessToken;
    private String refreshToken;
}
```

설명:

- 로그인 성공 후 클라이언트에게 토큰을 응답하는 DTO이다.
- 서버가 직접 생성해서 반환하므로 `@AllArgsConstructor`를 사용할 수 있다.

## 12. AuthController.java

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
- 로그인 검증 자체는 Controller가 아니라 Service에서 처리한다.
- Controller는 요청 DTO를 받고 Service를 호출한 뒤 응답 DTO를 반환한다.

## 13. AuthService.java

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

설명:

- `MemberRepository`: 이메일로 회원을 조회한다.
- `PasswordEncoder`: 비밀번호를 검증한다.
- `JwtProvider`: 토큰을 생성한다.

로그인할 때는 DB에 저장된 비밀번호와 입력 비밀번호를 직접 문자열 비교하지 않는다.

```java
passwordEncoder.matches(rawPassword, encodedPassword)
```

를 사용한다.

## 14. 토큰이란?

토큰은 로그인한 사용자를 식별하기 위해 클라이언트와 서버가 주고받는 문자열이다.

JWT 방식에서는 로그인 성공 시 서버가 토큰을 발급하고, 클라이언트는 이후 요청마다 Access Token을 헤더에 담아 보낸다.

```http
Authorization: Bearer access-token
```

토큰에는 보통 다음 정보가 들어간다.

- 사용자 식별자: memberId
- 권한: role
- 발급 시간: iat
- 만료 시간: exp

예시 payload:

```json
{
  "sub": "1",
  "role": "ROLE_USER",
  "iat": 1710000000,
  "exp": 1710003600
}
```

주의할 점:

JWT는 쉽게 디코딩할 수 있으므로 민감한 정보를 넣으면 안 된다.

넣으면 안 되는 정보:

- 비밀번호
- 주민번호
- 카드 번호
- 주소
- 민감한 개인정보

## 15. Access Token과 Refresh Token

### Access Token

Access Token은 API 요청 인증에 사용하는 토큰이다.

특징:

- 수명이 짧다.
- API 요청마다 Authorization 헤더에 담아 보낸다.
- 서버는 Access Token을 검증해서 사용자를 인증한다.

예시:

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

흐름:

```text
Access Token 만료
→ 서버가 401 Unauthorized 응답
→ 프론트가 Refresh Token으로 재발급 요청
→ 서버가 Refresh Token 검증
→ 새 Access Token 발급
```

## 16. JwtProvider.java

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
            .setSubject(String.valueOf(memberId)) // 사용자 식별자
            .claim("role", role)                  // 사용자 권한
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
- `getMemberId`: 토큰의 subject에서 memberId를 꺼낸다.
- `getRole`: 토큰의 claim에서 role을 꺼낸다.
- `validateToken`: 토큰이 유효한지 검증한다.

주의:

실무에서는 `secretKey`를 코드에 직접 쓰지 않고 `application.yml` 같은 설정 파일에서 주입받는다.

## 17. JwtAuthenticationFilter.java

JWT 필터는 요청이 Controller에 도착하기 전에 실행된다.

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
            String token = authorization.substring(7);

            // 토큰이 유효하면 사용자 정보를 SecurityContext에 저장한다.
            if (jwtProvider.validateToken(token)) {
                Long memberId = jwtProvider.getMemberId(token);

                CustomUserDetails userDetails =
                    customUserDetailsService.loadUserByMemberId(memberId);

                Authentication authentication =
                    new UsernamePasswordAuthenticationToken(
                        userDetails,
                        null,
                        userDetails.getAuthorities()
                    );

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
- `SecurityContext`: 인증된 사용자 정보를 저장하는 공간이다.
- `Authentication`: 현재 사용자의 인증 정보를 나타내는 객체이다.

Access Token이 만료되거나 유효하지 않으면 인증 정보가 저장되지 않는다. 보호된 API라면 보통 `401 Unauthorized` 응답이 발생한다.

## 18. Access Token 요청 처리 흐름

로그인 후 클라이언트는 API를 요청할 때 Access Token을 Authorization 헤더에 담아 보낸다.

요청 예시:

```http
GET /members/me
Authorization: Bearer eyJhbGciOi...
```

처리 흐름:

```text
1. 클라이언트가 Authorization 헤더에 Access Token을 담아 요청한다.

2. JwtAuthenticationFilter가 요청을 가로챈다.

3. request.getHeader("Authorization")로 헤더 값을 꺼낸다.

4. 헤더 값이 "Bearer "로 시작하는지 확인한다.

5. "Bearer " 부분을 제거하고 순수 JWT 토큰만 꺼낸다.

6. JwtProvider가 토큰의 서명과 만료 시간을 검증한다.

7. 토큰이 유효하면 토큰 안의 subject에서 memberId를 꺼낸다.

8. memberId로 사용자 정보를 조회한다.

9. CustomUserDetails 객체를 만든다.

10. Authentication 객체를 만든다.

11. SecurityContext에 Authentication을 저장한다.

12. 이후 Controller에서는 @AuthenticationPrincipal로 사용자 정보를 사용할 수 있다.
```

코드 흐름:

```java
// 1. Authorization 헤더를 꺼낸다.
String authorization = request.getHeader("Authorization");

// 2. Bearer 토큰 형식인지 확인한다.
if (authorization != null && authorization.startsWith("Bearer ")) {

    // 3. "Bearer "를 제거하고 순수 토큰만 꺼낸다.
    String token = authorization.substring(7);

    // 4. 토큰이 유효한지 검증한다.
    if (jwtProvider.validateToken(token)) {

        // 5. 토큰에서 사용자 id를 꺼낸다.
        Long memberId = jwtProvider.getMemberId(token);

        // 6. 사용자 정보를 조회해서 Spring Security용 객체로 만든다.
        CustomUserDetails userDetails =
            customUserDetailsService.loadUserByMemberId(memberId);

        // 7. 인증 객체를 만든다.
        Authentication authentication =
            new UsernamePasswordAuthenticationToken(
                userDetails,
                null,
                userDetails.getAuthorities()
            );

        // 8. SecurityContext에 인증 정보를 저장한다.
        SecurityContextHolder.getContext().setAuthentication(authentication);
    }
}

// 9. 다음 필터로 요청을 넘긴다.
filterChain.doFilter(request, response);
```

SecurityContext에 인증 정보가 저장되면 Spring Security는 해당 요청을 인증된 요청으로 판단한다.

Controller 예시:

```java
@GetMapping("/me")
public MemberResponse myInfo(
    @AuthenticationPrincipal CustomUserDetails userDetails
) {
    Long memberId = userDetails.getId();

    return memberService.findMember(memberId);
}
```

정리:

```text
Authorization 헤더
→ JWT 추출
→ JWT 검증
→ memberId 추출
→ 사용자 정보 조회
→ Authentication 생성
→ SecurityContext 저장
→ Controller에서 사용자 정보 사용
```

면접 답변:

> 로그인 후 클라이언트는 Access Token을 Authorization 헤더에 Bearer 형식으로 담아 요청합니다. 서버의 JWT 필터는 이 헤더에서 토큰을 추출하고, 토큰의 서명과 만료 시간을 검증합니다. 토큰이 유효하면 토큰 안의 사용자 id를 꺼내 사용자 정보를 조회하고, Authentication 객체를 만들어 SecurityContext에 저장합니다. 이후 Controller에서는 `@AuthenticationPrincipal`을 통해 인증된 사용자 정보를 사용할 수 있습니다.

## 19. CustomUserDetails.java

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

설명:

- Spring Security에서 사용하는 사용자 정보 객체이다.
- 컨트롤러에서 `@AuthenticationPrincipal`로 받을 수 있다.
- `getUsername()`은 보통 email이나 로그인 id를 반환한다.

## 20. CustomUserDetailsService.java

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

설명:

- 토큰에서 꺼낸 memberId로 회원을 조회한다.
- 조회한 회원 정보를 Spring Security가 사용할 수 있는 `CustomUserDetails`로 변환한다.

## 21. SecurityConfig.java

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

                // authenticated(): 인증된 사용자만 접근 허용
                .anyRequest().authenticated()
            )

            // JWT 필터를 UsernamePasswordAuthenticationFilter보다 먼저 실행한다.
            .addFilterBefore(jwtAuthenticationFilter, UsernamePasswordAuthenticationFilter.class);

        return http.build();
    }
}
```

어노테이션 설명:

- `@Configuration`: 설정 클래스임을 나타낸다.
- `@EnableWebSecurity`: Spring Security 설정을 활성화한다.
- `@Bean`: 메서드가 반환하는 객체를 Spring Bean으로 등록한다.

설정 설명:

- `PasswordEncoder`: 비밀번호 암호화와 검증에 사용한다.
- `csrf.disable()`: JWT 기반 REST API에서는 보통 CSRF를 비활성화한다.
- `SessionCreationPolicy.STATELESS`: 세션을 사용하지 않고 토큰 기반으로 인증한다.
- `requestMatchers(...).permitAll()`: 로그인과 회원가입은 인증 없이 접근을 허용한다.
- `anyRequest().authenticated()`: 나머지 요청은 인증이 필요하다.
- `addFilterBefore(...)`: JWT 필터를 Spring Security 필터 체인에 등록한다.

## 22. @AuthenticationPrincipal

인증이 끝난 사용자 정보를 Controller에서 받을 때 사용한다.

```java
@GetMapping("/me")
public MemberResponse myInfo(
    @AuthenticationPrincipal CustomUserDetails userDetails
) {
    Long memberId = userDetails.getId();

    return memberService.findMember(memberId);
}
```

여기서 들어오는 값은 토큰 자체가 아니다.

```text
토큰
→ JWT 필터 검증
→ CustomUserDetails 생성
→ SecurityContext 저장
→ @AuthenticationPrincipal로 주입
```

## 23. 401 Unauthorized

`401 Unauthorized`는 인증되지 않은 요청이라는 뜻이다.

발생 예시:

- Access Token이 없음
- Access Token이 만료됨
- Access Token이 잘못됨
- 로그인하지 않은 사용자가 보호된 API에 접근함

정리:

```text
401 Unauthorized → 인증 안 됨
403 Forbidden → 인증은 됐지만 권한 없음
```

## 24. 로그인하지 않은 요청은 어디서 막히는가?

로그인하지 않은 사용자가 보호된 API에 접근하면 Spring Security 필터 체인에서 막힌다.

예를 들어 다음 요청은 Access Token 없이 보호된 API에 접근하는 경우이다.

```http
GET /members/me
```

이 요청에는 Authorization 헤더가 없다.

```text
Authorization: 없음
```

처리 흐름:

```text
요청 들어옴
→ JwtAuthenticationFilter 실행
→ Authorization 헤더 없음
→ SecurityContext에 Authentication 저장 안 됨
→ 다음 필터로 이동
→ SecurityConfig의 anyRequest().authenticated() 검사
→ 인증 정보가 없으므로 401 Unauthorized 응답
```

즉, `JwtAuthenticationFilter`가 직접 요청을 막는 것이 아니라, 토큰이 없거나 유효하지 않으면 인증 정보를 만들지 않는다.

그 다음 Spring Security가 `SecurityContext`에 인증 정보가 있는지 확인하고, 인증 정보가 없으면 요청을 거부한다.

관련 설정:

```java
@Bean
public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
    // JWT를 검사할 커스텀 필터를 만든다.
    JwtAuthenticationFilter jwtAuthenticationFilter =
        new JwtAuthenticationFilter(jwtProvider, customUserDetailsService);

    http
        // JWT 기반 REST API에서는 CSRF를 보통 비활성화한다.
        .csrf(csrf -> csrf.disable())

        // 세션을 사용하지 않는 Stateless 인증 방식으로 설정한다.
        .sessionManagement(session ->
            session.sessionCreationPolicy(SessionCreationPolicy.STATELESS)
        )

        // URL별 접근 권한을 설정한다.
        .authorizeHttpRequests(auth -> auth
            // 로그인, 회원가입은 인증 없이 허용한다.
            .requestMatchers("/auth/login", "/members").permitAll()

            // 그 외 요청은 인증된 사용자만 접근할 수 있다.
            .anyRequest().authenticated()
        )

        // JWT 필터를 Spring Security 필터 체인에 등록한다.
        .addFilterBefore(jwtAuthenticationFilter, UsernamePasswordAuthenticationFilter.class);

    return http.build();
}
```

핵심 설정:

```java
.requestMatchers("/auth/login", "/members").permitAll()
```

로그인과 회원가입 요청은 인증 없이 허용한다.

```java
.anyRequest().authenticated()
```

그 외 나머지 요청은 인증된 사용자만 접근할 수 있다.

예시:

```text
POST /auth/login → 허용
POST /members → 허용
GET /members/me → 인증 필요
PATCH /members/1/name → 인증 필요
DELETE /members/1 → 인증 필요
```

면접 답변:

> 로그인하지 않은 요청은 Spring Security 필터 체인에서 막힙니다. JWT 필터는 Authorization 헤더에서 토큰을 꺼내 검증하고, 유효하면 Authentication 객체를 SecurityContext에 저장합니다. 토큰이 없거나 유효하지 않으면 인증 정보가 저장되지 않고, 이후 `SecurityConfig`의 `anyRequest().authenticated()` 설정에 의해 보호된 API 접근이 거부되어 401 응답이 발생합니다.

## 25. 기능별 접근 권한 설정

실무에서는 기능마다 로그인 필요 여부가 다를 수 있다.

예를 들어 게시글 기능은 다음처럼 나눌 수 있다.

```text
GET /posts → 글 목록 조회, 로그인 없이 가능
GET /posts/{id} → 글 상세 조회, 로그인 없이 가능
POST /posts → 글 작성, 로그인 필요
PATCH /posts/{id} → 글 수정, 로그인 필요
DELETE /posts/{id} → 글 삭제, 로그인 필요
```

이런 접근 권한은 JWT 필터에서 정하는 것이 아니라 `SecurityConfig`에서 설정한다.

예시:

```java
@Bean
public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
    // JWT를 검사할 커스텀 필터를 만든다.
    JwtAuthenticationFilter jwtAuthenticationFilter =
        new JwtAuthenticationFilter(jwtProvider, customUserDetailsService);

    http
        // JWT 기반 REST API에서는 CSRF를 보통 비활성화한다.
        .csrf(csrf -> csrf.disable())

        // 세션을 만들지 않고, 매 요청마다 토큰으로 인증한다.
        .sessionManagement(session ->
            session.sessionCreationPolicy(SessionCreationPolicy.STATELESS)
        )

        // URL과 HTTP Method 기준으로 접근 권한을 설정한다.
        .authorizeHttpRequests(auth -> auth
            // 로그인, 회원가입은 인증 없이 허용
            .requestMatchers("/auth/login", "/members").permitAll()

            // 게시글 조회는 인증 없이 허용
            .requestMatchers(HttpMethod.GET, "/posts/**").permitAll()

            // 나머지 요청은 인증 필요
            .anyRequest().authenticated()
        )

        // JWT 필터를 UsernamePasswordAuthenticationFilter보다 먼저 실행한다.
        .addFilterBefore(jwtAuthenticationFilter, UsernamePasswordAuthenticationFilter.class);

    return http.build();
}
```

설명:

```java
.requestMatchers(HttpMethod.GET, "/posts/**").permitAll()
```

게시글 조회 요청은 로그인하지 않아도 접근할 수 있다.

```java
.anyRequest().authenticated()
```

그 외 요청은 인증된 사용자만 접근할 수 있다.

예시 결과:

```text
GET /posts → 허용
GET /posts/1 → 허용
POST /posts → 인증 필요
PATCH /posts/1 → 인증 필요
DELETE /posts/1 → 인증 필요
```

흐름:

```text
요청 들어옴
→ JWT 필터에서 토큰이 있으면 검증
→ SecurityConfig의 인가 규칙 확인
→ permitAll이면 인증 없어도 통과
→ authenticated면 SecurityContext에 인증 정보가 있어야 통과
```

정리:

- JWT 필터: 토큰을 검증하고 인증 정보를 만든다.
- SecurityConfig: 어떤 API를 허용하고 막을지 정한다.
- `permitAll()`: 로그인 없이 접근 가능하다.
- `authenticated()`: 로그인한 사용자만 접근 가능하다.

면접 답변:

> 기능마다 접근 권한은 `SecurityConfig`에서 URL과 HTTP Method 기준으로 설정합니다. 예를 들어 게시글 조회는 `GET /posts/**`에 대해 `permitAll()`로 열어두고, 작성, 수정, 삭제는 `authenticated()`가 필요하도록 설정할 수 있습니다. JWT 필터는 토큰을 검증해 인증 정보를 만드는 역할이고, 실제로 어떤 API를 허용하거나 막을지는 `SecurityConfig`의 인가 규칙이 담당합니다.

## 26. 한 번에 외우는 면접 답변

> Spring에서 회원가입 시 비밀번호는 원문으로 저장하지 않고 `PasswordEncoder`를 사용해 BCrypt 해시로 암호화해서 저장합니다. 로그인 시에는 이메일로 회원을 조회하고, 사용자가 입력한 비밀번호와 DB에 저장된 해시값을 `matches()`로 비교합니다. 인증에 성공하면 Access Token과 Refresh Token을 발급합니다. Access Token은 API 요청 인증에 사용하고, Refresh Token은 Access Token 재발급에 사용합니다. 이후 클라이언트는 Access Token을 Authorization 헤더에 담아 요청하고, 서버의 JWT 필터는 토큰을 검증한 뒤 사용자 정보를 SecurityContext에 저장합니다. URL별 접근 권한은 `SecurityConfig`에서 설정하며, Controller에서는 `@AuthenticationPrincipal`을 통해 인증된 사용자 정보를 사용할 수 있습니다.
