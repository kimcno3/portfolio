# :pushpin: Jwt Token에 담길 사용자 정보에 대한 결정과 표준에 대한 이해

## 문제점
쿠키/세션 인증 방식을 사용할 경우, API 서버로 요청을 보낸 사용자에 대한 정보를 조회하기 위해 헤더에 담겨온 세션 ID로 세션을 조회를 해야만 합니다. 이는 결국 session에 대한 접근을 해야 하는 소요가 발생해 트래픽이 집중되는 경우 성능에 영향을 미칠 수 있는 가능성이 높다고 생각했습니다. 

그러나 이러한 문제는 Jwt 토큰 인증 방식을 사용하면서 크게 해결되었는데 Jwt 토큰은 토큰 안에 요청을 보낸 클라이언트에 대한 정보가 담을 수 있기 때문입니다. 

토큰에 클라이언트가 누군지에 대해 식별할 수 있으며 외부에 노출될 경우 큰 문제가 되지 않을 정보들을 토큰에 넣어두고 이를 헤더에 담아 요청과 함께 받는다면, 서버 입장에서도 DB에 접근없이 복호화 과정을 통해서만 클라이언트 정보를 조회할 수 있다는 점이 Jwt 인증 방식의 장점입니다. 그래서 클라이언트에 대한 id, email, name 등과 같은 기본적인 인적 정보를 토큰 PayLoad에 넣어두도록 하는 것이 일반적입니다.

하지만 토큰에 저장될 정보를 개발자 입맛에 따라 저장해 둘 수 있지만 많은 데이터와 경험을 토대로 효율적인 방식으로 토큰에 담을 정보의 정해보고 싶었고 많은 자료룰 찾아보던 중 등록된 클레임이라는 것을 알게 되었습니다.

## 해결방안
### 등록된 클레임(Registered Claims)
등록된 클레임들은 서비스에서 필요한 정보들이 아닌, 토큰에 대한 정보들을 담기위하여 이름이 이미 정해진 클레임들입니다. 등록된 클레임의 사용은 모두 선택적 (optional)이며, 이에 포함된 클레임 이름들은 다음과 같습니다:

- `iss` : 토큰 발급자 (issuer)
- `sub` : 토큰 제목 (subject)
- `aud` : 토큰 대상자 (audience)
- `exp` : 토큰의 만료시간 (expiraton), 시간은 NumericDate 형식으로 되어있어야 하며 (예: 1480849147370) 언제나 현재 시간보다 이후로 설정되어있어야합니다.
- `nbf` : Not Before 를 의미하며, 토큰의 활성 날짜와 비슷한 개념입니다. 여기에도 NumericDate 형식으로 날짜를 지정하며, 이 날짜가 지나기 전까지는 토큰이 처리되지 않습니다.
- `iat` : 토큰이 발급된 시간 (issued at), 이 값을 사용하여 토큰의 age 가 얼마나 되었는지 판단 할 수 있습니다.
- `jti` : JWT의 고유 식별자로서, 주로 중복적인 처리를 방지하기 위하여 사용됩니다. 일회용 토큰에 사용하면 유용합니다.

### 적용 코드
#### JwtSecurityService 
```java
@Slf4j
@Service
@RequiredArgsConstructor
public class JwtSecurityService implements SecurityService {

  private final TokenProvider tokenProvider;
  private final AuthenticationManagerBuilder authenticationManagerBuilder;

  @Override
  public void logIn(LogInCommand command) {

    // 인증용 객체 생성 -> username으로 userId를 지정해준다.
    UsernamePasswordAuthenticationToken authenticationToken =
        new UsernamePasswordAuthenticationToken(command.getUserId(), command.getPassword());

    Authentication authentication = authenticationManagerBuilder.getObject()
        .authenticate(authenticationToken);

    // 토큰 생성 로직
    String jwt = tokenProvider.createToken(authentication, command.getName());

    ServletRequestAttributes attributes = (ServletRequestAttributes) RequestContextHolder
        .currentRequestAttributes();

    HttpServletResponse response = attributes.getResponse();

    response.setHeader(AUTHORIZATION_HEADER, "Bearer " + jwt);

  }
```

실제 코드로 적용해본다면 Login 메소드가 동작할 때 검증을 위해 사용된 UsernamePasswordAuthenticationToken 객체에 username을 userId로 지정합니다.

#### TokenProvider
```java
@Slf4j
@Component
public class TokenProvider implements InitializingBean {
  
  // (생략)

  public String createToken(Authentication authentication, String userName) {

    String authorities = authentication.getAuthorities()
        .stream()
        .map(GrantedAuthority::getAuthority)
        .collect(Collectors.joining(","));

    long now = (new Date()).getTime();
    Date validity = new Date(now + this.tokenValidityInMilliseconds);

	// authentication의 name(userId 값)을 subject로 지정
    Claims claims = Jwts.claims()
        .setSubject(authentication.getName())
        .setExpiration(validity);

	// 접근권한 정보와 사용자 이름은 추가로 저장
    claims.put(AUTHORITIES_KEY, authorities);
    claims.put(NAME_KEY, userName);

    return Jwts.builder()
        .setClaims(claims)
        .signWith(key, SignatureAlgorithm.HS512)
        .compact();

  }
```

이후 Jwt 토큰을 생성하는 과정에서 Claims 객체의 subject 필드에 이전에 담아둔 username(userId)가 저장되고 접근권한 정보나 사용자 이름과 같은 부수적인 사용자 정보는 추가로 저장해주도록 했습니다.

#### 생성된 토큰의 Payload 예시
```
{
  "sub": "12",
  "exp": 1660932040,
  "auth": "ROLE_MANAGER",
  "name": "manager_name"
}
```

위 예시 Payload처럼 sub에 userId를 저장하고 email,name과 같은 부수적인 정보는 지정한 키에 값으로 담겨져서 토큰이 생성되는 것을 볼 수 있습니다.

## 마치며
이러한 구성을 통해 단순히 복호화를 통해 토큰에서 직접 사용자 정보를 조회하면서 저장소 접근을 위한 과정을 생략함으로써 어플리케이션의 성능을 최적화해볼 수 있었고 표준 클레임 양식을 지키면서 정보를 넣어 Spring Security에서 제공해주는 Claims 객체의 필드들을 효과적으로 활용해 볼 수 있었습니다.

## 참고자료
https://velopert.com/2389