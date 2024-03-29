# :pushpin: 로그인 회원 정보 조회를 위한 Resolver 구현
## 문제점
로그인한 회원이 구매나 판매 서비스에 대한 요청을 보낼 경우, 특정 메소드들은 UserId 정보를 필요로 하는 경우가 존재합니다.

예를 들면 즉시 구매 요청을 보낸 경우, 판매 입찰가와 거래 체결을 진행하기 위해선 구매자의 정보도 필요하기 때문에 userId를 필요로 하게 됩니다. 

하지만 이를 위해 모든 Service 객체에 세션에 대한 기능을 수행하는 SecuriryService 객체를 주입하는 것은 복잡한 연결 구조를 가지게 됩니다.

이러한 문제점을 간단히 해결할 수 있는 방법은 **HadlerMethodArgumentResolver** 객체를 생성하는 것입니다.

## 해결방안
### HadlerMethodArgumentResolver
HandlerMethodArgumentResolver 인터페이스를 구현한 구현 클래스들은 controller의 매개변수로 지정된 변수들을 어노테이션이나 타입에 따라 알맞는 형태로 가공해 Controller에 넘겨주는 역할을 합니다.

![](https://platanus.me/wp-content/uploads/2021/09/04857c3909c14b24bef63927501b9afd.png)

위 그림에서도 볼 수 있듯이 DispatcherServlet에서 받아온 request에 대한 처리를 해줄 핸들러로 넘어가기 전에 ArgumentResolver에서 요청 데이터를 생성해 넘겨주도록 설계되어 있습니다.

### 적용 코드
다음은 실제 HandlerMethodArgumentResolver의 구현 클래스와 기능 구현을 위해 추가된 코드들입니다. 

#### SignInUserId 어노테이션
```java
@Target(ElementType.PARAMETER)
@Retention(RetentionPolicy.RUNTIME)
public @interface SignInUserId {
}
```
위 어노테이션이 선언된 매개변수일 경우, userId를 탐색하는 resolver가 실행되도록 실행 시점을 지정합니다.


#### UserContoller
```java
@RequestMapping("/user")
@RequiredArgsConstructor
public class UserController {

  @GetMapping("/signin/check")
  @CheckSignIn                   // resolver가 실행
  public ResponseDto signInCheck(@SignInUserId int userId) {
    
    return new ResponseDto(true, null, "로그인 체크", null);

  }
}
```
@SignInUserId가 선언된 매개변수가 제대로 동작하는지 확인하기 위한 API를 구현했습니다.

리졸버가 제대로 동작한다면 userId 라는 매개변수에 현재 로그인된 회원의 userId를 가져오게 됩니다.

#### SignInUserArgumentResolver
```java
@Component
@RequiredArgsConstructor
public class SignInUserArgumentResolver implements HandlerMethodArgumentResolver {

  private final SecurityService securityService;

  // 해당 리졸버를 실행시키기 위한 조건이 될 어노테이션을 지정
  @Override
  public boolean supportsParameter(MethodParameter parameter) {

    return parameter.hasParameterAnnotation(SignInUserId.class);

  }
  
  // 리졸버가 생성할 매개변수에 담을 값을 생성
  @Override
  public Object resolveArgument(MethodParameter parameter,
      ModelAndViewContainer mavContainer,
      NativeWebRequest webRequest,
      WebDataBinderFactory binderFactory) {
      
    // UserId를 가져오는 메소드를 호출
    return securityService.getCurrentUserId();

  }

}

```
로그인한 회원의 userId를 Contoller의 매개변수로 가져와줄 리졸버입니다. 

해당 리졸버에서만 SecurityService 객체를 주입받고 관련 메소드를 호출하도록 구성한다면 다른 Service 클래스에선 SecurityService를 주입받을 소요를 제거할 수 있습니다.

#### SessionSecurityService(SecurityService 구현 클래스)
```java
@Slf4j
@RequiredArgsConstructor
public class SessionSecurityService implements SecurityService {
  
    private final HttpSession session;

  // Redis 적용 전에 로그인 회원 정보를 담아둘 DB를 대체할 자료구조
  private final ExpiringMap<String, Integer> sessionDataBase = ExpiringMap.builder()
                                                              .variableExpiration()
                                                              .build();
  // 일부 코드 생략
  
  // SessionId를 가지고 UserId를 조회
  @Override
  public int getCurrentUserId() {

    String sessionId = getCurrentSessionId();

    return sessionDataBase.get(sessionId);

  }
  // Session에 저장된 SessionId를 조회
  private String getCurrentSessionId() {

    return (String) session.getAttribute(SESSION_ID);

  }
```
리졸버에서 호출하는 메소드의 구현 코드입니다.

세션에서 SessionId를 조회하고, SessionDatabase에 저장된 userId를 조회하고 리턴하는 역할을 합니다.

> 해당 작업 이후 Redis를 연동한 새로운 클래스를 구현했습니다.

## 마치며
이처럼 Resolver에 대한 동작 원리를 이해하고 로그인 회원 조회라는 부가 기능에 대한 책임을 구분함으로써 핵심 기능을 수행하는 다른 클래스들에게 코드추가를 하지 않고 깔끔한 코드 구성을 할 수 있었습니다.

## 참고자료
- https://velog.io/@gillog/Spring-HandlerMethodArgumentResolver-PathVariable-RequestHeader-RequestParam
- https://platanus.me/post/1678
