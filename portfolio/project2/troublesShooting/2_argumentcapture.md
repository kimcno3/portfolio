# :pushpin: 내부 생성 객체에 대한 테스트 방법 구성

## 문제점
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

    if (isExistEmail(commandDto.getEmail())) {

      throw new AlreadyExistEmailException("이미 존재하는 이메일입니다");

    }

    String encodedPassword = passwordEncoder.encode(commandDto.getPassword());
    
    // 클래스 내부적으로만 동작하는 builder 메소드
    UserDto user = UserDto.builder()
        .email(commandDto.getEmail())
        .password(encodedPassword)
        .name(commandDto.getName())
        .phone(commandDto.getPhone())
        .address(commandDto.getAddress())
        .build();

    userRepository.save(user);

  }
}
```
UserServiceImpl 클래스의 signUp()의 내부 구현 코드를 보면 Entity 객체인 UserDto 객체를 생성하는 builder 메소드는 signUp() 메소드 내에서만 동작하고 외부로 생성된 UserDto를 리턴하지 않도록 구현되어 있습니다.


#### UserServiceImplTest
```java
@ExtendWith(MockitoExtension.class)
class UserServiceImplTest {

  UserService userService;

  UserRepository userRepository;

  PasswordEncoder passwordEncoder;

  @BeforeEach
  void setUp() {

    userRepository = mock(MybatisUserRepository.class);

    passwordEncoder = mock(BCryptPasswordEncoder.class);

    userService = new UserServiceImpl(userRepository, passwordEncoder);

  }

  @Test
  @DisplayName("회원가입 로직 테스트")
  void signUpTest() {
    // given
    SignUpCommand command = new SignUpCommand(
        "email", "password", "name", "phone", "address"
    );

    // when
    when(userService.isExistEmail(command.getEmail())).thenReturn(false);

    when(passwordEncoder.encode(command.getPassword())).thenReturn(command.getPassword());

    userService.signUp(command);

    // then
    // build된 UserDto에 대한 검증이 불가능하다.
    verify(userRepository, times(1)).save(any());

  }
```
그러다보니 UserServiceImpl에 대한 Unit Test를 진행하는 경우에도 생성된 UserDto 객체를 가져올 수 있는 방법이 없어 단순하게 UserRepository 객체의 메소드가 호출되었는지에 대한 검증만 진행할 수 있었습니다.

저는 테스트 코드에서 UserDto 객체또한 제대로 생성되고 Repository의 매개변수로 넘겨지는지에 대한 확인도 하고 싶었기 때문에 이를 해결하기 위한 방법을 찾기로 했습니다.

## 해결방안
### ArgumentCapture 클래스
ArgumentCapture 클래스는 이름 그대로 매개변수의 값을 캡쳐해 꺼내볼 수 있도록 기능을 지원해주는 클래스입니다.

즉, 위 테스트 코드에서 userRepository에 매개변수로 넘겨질 UserDto 객체에 대한 정보를 가져와 검증해볼 수 있다는 의미입니다.

### 구현코드
#### 수정된 UserServiceImplTest
```java
// 코드 생략

  @Test
  @DisplayName("회원가입 로직 테스트")
  void signUpTest() {
    // given
    SignUpCommand command = new SignUpCommand(
        "email", "password", "name", "phone", "address"
    );

    // 1. UserDto에 대한 ArgumentCaptor 객체 생성
    ArgumentCaptor<UserDto> userCap = ArgumentCaptor.forClass(UserDto.class);

    // when
    when(userService.isExistEmail(command.getEmail())).thenReturn(false);

    when(passwordEncoder.encode(command.getPassword())).thenReturn(command.getPassword());

    userService.signUp(command);

    // then
    // 2. any() 대신 userCap.capture()를 호출해 매개변수로 사용될 객체에 대한 정보를 저장
    verify(userRepository).save(userCap.capture());

    // 3. 저장된 객체를 호출
    UserDto user = userCap.getValue();
    
    // 4. 호출된 객체에 대한 검증 진행
    assertThat(user.getEmail()).isEqualTo(command.getEmail());
    assertThat(user.getPassword()).isEqualTo(command.getPassword());
    assertThat(user.getName()).isEqualTo(command.getName());
    assertThat(user.getPhone()).isEqualTo(command.getPhone());
    assertThat(user.getAddress()).isEqualTo(command.getAddress());

  }
```
## 마치며
ArgumentCaptor 클래스를 활용해 void 메소드에 대한 테스트에서도 내부적으로 특정 개체를 리턴하는 메소드에 대해 데이터를 꺼내와 검증해 볼 수 있습니다.

이를 통해 조금 더 확실한 검증 과정을 가진 테스트 코드를 구성할 수 있었습니다.

## 참고 자료
- https://kogle.tistory.com/255
- https://sun-22.tistory.com/94