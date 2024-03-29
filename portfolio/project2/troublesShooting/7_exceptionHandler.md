# :pushpin: 예외 처리에 대한 책임 할당의 고민
## 문제점
필요한 기능들을 구현하면서 발생할 수 있는 **예외 상황에 대한 처리 로직**을 추가하게 되는 경우가 생겨났습니다. 이때 예외 처리 로직을 어떤 레이어에 책임을 주어야 할지에 대한 고민을 하게 되었습니다.

예를 들면 회원가입 로직을 진행하기 위해 Request로 넘어온 email과 password가 양식에 맞게 입력이 되었는지 확인하거나 이미 가입된 email이 존재하는지 확인하는 과정 등이 필요한텐데, 정상적인 동작을 해선 안되는 경우에 예외를 던저주는 역할을 누가 할지에 대한 결정을 해 줄 필요가 있다고 판단했습니다.
## 해결방안
### Layered Architecture(레이어드 아키텍쳐)
레이어드 아키텍쳐는 코드들을 논리적으로 또는 역할에 따라 독립된 모듈로 구분하고 나눠서 설계해 코드들의 가독성, 재사용성, 확정성, 유지 보수 가능성을 높힐 수 있는 설계 방법을 의미합니다.

각 모듈의 형태를 그림으로 나타내면 아래와 같습니다.

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FbCxdbP%2Fbtrsnogd42h%2FOvPZq7EzMlOkCiKhRP22AK%2Fimg.png)

각 모듈에 대한 책임을 간단히 설명해보면

- Presentation Layer : 서버에 요청을 보낼 클라이언트와 직접적으로 연결되어 요청과 응답에 대한 처리를 담당
- Business Layer : 요청에 따른 데이터를 가공해 응답에 담아주기 위한 핵심 비즈니스 로직을 담당
- Persistence Layer : DB와 관련된 로직 구현을 담당

이라고 할 수 있습니다. 

결국 MVC 패턴에서 Controller - Service - Repository 가 각 모듈의 역할을 실제로 담당하는 녀석들이라 생각 할 수 있습니다.

![](https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FlCOWD%2FbtrslVyO4cD%2FfViidrJt2eCAId4ZEgQ9H1%2Fimg.png)

위 사진은 각 모듈이 담당하는 핵심 역할에 대한 내용입니다.

간단히 설명해본다면 인증과 Json 교환에 대한 책임을 Presentation Layer가 가지고 있고 비즈니스 로직, 유효성, 인가에 대한 책임은 Business Layer가 가지는 것이라 볼 수 있습니다.

그러므로 Request에 담아온 데이터에 대한 검사 및 처리는 Controller에서, 이후 비즈니스 로직을 수행하는 과정에서 생겨난 예외에 대한 책임은 Service가 가지는 것이 합리적이라는 결론을 내리게 되었습니다.

#### UserController 
```java
@RestController
@RequestMapping("/user")
@RequiredArgsConstructor
public class UserController {

  private final UserService userService;

  private final SecurityService securityService;

  /**
   *.
   */

  @PostMapping("/signup")   // request 객체에 대한 유효성 검사를 진행
  public ResponseDto signUp(@Valid @RequestBody SignUpRequest requestDto) {

    userService.signUp(SignUpRequest.toCommand(requestDto));

    return new ResponseDto(true, null, "회원가입 성공", null);

  }
}
```
위처럼 Request 객체에 대한 유효성 검사는 `@Valid`를 활용해 Controller에서 담당해줍니다.

#### UserServiceImpl
```java
@Service
@RequiredArgsConstructor
public class UserServiceImpl implements UserService {

  private final UserRepository userRepository;

  private final PasswordEncoder passwordEncoder;

  @Override
  @Transactional
  public void signUp(SignUpCommand commandDto) {

    // 비즈니스 로직에서 발생할 수 있는 예외 처리는 Service에서!
    if (isExistEmail(commandDto.getEmail())) {

      throw new AlreadyExistEmailException("이미 존재하는 이메일입니다");

    }

    String encodedPassword = passwordEncoder.encode(commandDto.getPassword());

    UserDto user = UserDto.builder()
        .email(commandDto.getEmail())
        .password(encodedPassword)
        .name(commandDto.getName())
        .phone(commandDto.getPhone())
        .address(commandDto.getAddress())
        .build();

    userRepository.save(user);

  }

  @Override
  @Transactional(readOnly = true)
  public UserDto checkEmailAndPw(String email, String password) {

    UserDto user = findByEmail(email);

    // 예외처리
    if (user == null) {

      throw new NotValidEmailException("이메일을 잘못 입력했습니다.");

    }
    
    // 예외처리
    if (!passwordEncoder.matches(password, user.getPassword())) {

      throw new NotValidPasswordException("비밀번호를 잘못 입력했습니다.");

    }

    return user;

  }
}
```
이후 비즈니스 로직 수행 과정에서 발생할 수 있는 예외는 Service에서 담당해줍니다.

## 마치며
이처럼 **레이어드 아키텍쳐의 각 모듈이 가져야 할 책임과 역할에 대한 구분을 명확히**하고 이에 따른 **예외처리 로직의 선언 위치도 분명한 목적을 가지고 선언**할 수 있었습니다.

## 참고자료
https://kimjingo.tistory.com/159