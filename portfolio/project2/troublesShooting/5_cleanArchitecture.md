# :pushpin: Layer 의존성 방향에 대한 고민
## 문제점
회원가입을 진행하기 위해선 꽤 많은 값을 Controller에서 request로 받아와 Service로 넘겨줘야 하는 소요가 발생했습니다. 여기서 문제는 많은 숫자의 변수를 각각의 매개변수로 넘겨주는 것이 합리적인 선택인지 의문이 생겼습니다.

#### SignUpRequest
```java
@Getter
@NoArgsConstructor
@AllArgsConstructor
public class SignUpRequest {

  @NotBlank
  private String email;
  @NotBlank
  @Length(max = 15)
  private String password;
  @NotBlank
  private String name;
  @NotBlank
  private String phone;
  @NotBlank
  private String address;
  
}
```
회원가입을 위한 데이터를 담아올 객체로써 생성한 SignUpRequest 클래스의 모습입니다.

#### UserController 
```java
@RestController
@RequestMapping("/user")
@RequiredArgsConstructor
public class UserController {

  private final UserService userService;

  private final SecurityService securityService;

  @PostMapping("/signup")
  public ResponseDto signUp(@Valid @RequestBody SignUpRequest requestDto) {

    // 1. 많은 수의 매개변수를 개별로 전달해주는 형태
    userService.signUp(requestDto.getEmail(), requestDto.getPassword(), requestDto.getName(), requestDto.getPhone(), requestDto.getAddress());
    
    // 2. RequestDto 자체를 Service 레이어에 넘겨주는 형태
    // userService.signUp(requestDto);
    
    return new ResponseDto(true, null, "회원가입 성공", null);

  }
  
  // 이하 코드 생략
}
```
1번 코드처럼 Controller에서 회원가입에 필요한 데이터를 5개의 매개변수를 각각 Service에 전달하게 된다면 코드의 길이가 길어질 뿐만 아니라 입력 정보에 대한 수정이 발생하면 Controller와 Service 레이어 모두 코드 수정이 필요한 상황이 생겨나게 됩니다.

그러면 2번 코드처럼 RequestDto 자체를 Service에 넘겨준 뒤, Service에서 필요한 데이터를 꺼내 쓰도록 설계한다면 1번 코드의 단점은 해결되는 것 처럼 보일 수 있습니다.

하지만 이는 **레이어 간의 의존성 방향**에 대해 고민해 봐야 하는 문제를 발생시켰습니다. 이를 해결하기 위해 필요한 개념 설명은 다음 단계에서 설명하겠습니다.

## 해결방안
### 클린 아키텍쳐
클린 아키텍쳐란 Robert C.Martin이 정립한 용어로 기존 계층형 아키텍쳐가 가지는 의존성에 대해 외부 인터페이스에 독립적으로 구현할 수 있도록 설계된 아키텍쳐라고 할 수 있습니다.

![](https://velog.velcdn.com/images%2F___pepper%2Fpost%2Fafcaa5d2-7653-4ccb-8d91-c8b8881142f6%2F%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202021-11-24%20%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE%206.41.12.png)

위 사진처럼 외부에서 내부 인터페이스로 데이터를 주고 받을 때 내부 인터페이스의 구현에 외부 인터페이스의 존재가 영향을 줘선 안되고 외부 인터페이스의 구현 형태에 상관없이 서비스를 구축해 나갈 수 있도록 하는 것이 클린 아키텍쳐의 목표라고 할 수 있습니다.

이를 프로젝트에 대입해본다면 2번 코드처럼 UserController에서 Request 객체를 그대로 Service에 전달하는 구조는 내부 인터페이스가 외부 인터페이스에 의존하게 되는 설계가 되며, 클린 아키텍쳐의 목적과 반대되는 형태가 됩니다.

결국 외부 변화에 따라 내부 구조가 변경될 가능성이 높아지는 설계가 되기 때문에 위험한 형태라고 볼 수 있습니다.

### Service 레어어를 위한 Dto 클래스 생성
위와 같은 문제를 해결하고 클린 아키텍쳐의 장점을 살릴 수 있는 설계를 위해선 Controller 레어어에서 Service 레이어로 데이터를 넘겨주기 전에 하나의 객체에 담아 보내주도록 Dto 객체를 생성하는 것을 생각했습니다. 즉, Service 레이어에서 사용하기 위한 Dto 클래스를 생성해보도록 했습니다.

#### SignUpCommand 클래스
```java
// 패키지 레이어도 Service 내에 포함되도록 설계
package api.soldout.io.soldout.service.user.command;

@Getter
@NoArgsConstructor
@AllArgsConstructor
public class SignUpCommand {

  private String email;
  private String password;
  private String name;
  private String phone;
  private String address;

}
```
`...Command` 라는 이름으로 Service 레이어에서 사용될 클래스를 선언했습니다. 이제 이 클래스 타입의 객체를 Controller에서 생성해 Request 객체에 담긴 데이터를 넘겨받고 Service 레이어에 넘겨주기만 하면 됩니다.

#### 수정된 SignUpRequest 클래스(팩토리 메소드 패턴 적용)
```java
@Getter
@NoArgsConstructor
@AllArgsConstructor
public class SignUpRequest {

  @NotBlank
  private String email;
  @NotBlank
  @Length(max = 15)
  private String password;
  @NotBlank
  private String name;
  @NotBlank
  private String phone;
  @NotBlank
  private String address;

  // 객체 생성의 책임을 Request 객체에 주기 위해 팩토리 메소드 패턴을 적용
  public static SignUpCommand toCommand(SignUpRequest requestDto) {
    return new SignUpCommand(
      requestDto.getEmail(),
      requestDto.getPassword(),
      requestDto.getName(),
      requestDto.getPhone(),
      requestDto.getAddress()
    );
  }
}
```
Command 객체를 생성할 때 new 예약어를 Controller에서 직접 선언해 생성하는 것은 OCP를 위반하는 설계가 될 가능성이 높습니다.(Command 객체만을 수정해도 Controller도 수정되어야 하기 때문입니다.)

그래서 객체 생성에 대한 책임을 Request에 위임하기 위해 Command 객체 생성을 위한 팩토리 메소드 패턴을 적용한 메소드를 선언했습니다.

#### 수정 후 UserController
```java
@RestController
@RequestMapping("/user")
@RequiredArgsConstructor
public class UserController {

  private final UserService userService;

  private final SecurityService securityService;

  @PostMapping("/signup")
  public ResponseDto signUp(@Valid @RequestBody SignUpRequest requestDto) {
    
    // 팩토리 메소드 패턴을 적용하지 않은 경우
    // userService.signUp(new signUpCommand(requestDto));
    
    // 팩토리 메소드 패턴 적용 코드
    userService.signUp(SignUpRequest.toCommand(requestDto));

    return new ResponseDto(true, null, "회원가입 성공", null);

  }
```

## 마치며
사실 레이어드 아키텍쳐에서 계층간 데이터를 주고받을 때 Dto 객체에 담아 소통하는 것이 가장 이상적인 구조이지만 오히려 중복된 값과 형태를 가지는 클래스를 과도하게 생성하게 되어 오히려 유지보수 측면에서 어려울 수 있다는 멘토님의 설명도 들었기에 Command 클래스 선언을 해야겠다는 결정을 하는데 조금은 돌아온 느낌도 있습니다. 

하지만 오히려 클린 아키텍쳐 개념에 대해서도 학습해볼 수 있었던 경험이라 생각했고 유연한 설계에 대한 고민을 해볼 수 있었다고 생각합니다.

## 참고자료
클린아키텍쳐에 대한 설명 글 : [링크](https://velog.io/@___pepper/%ED%81%B4%EB%A6%B0-%EC%95%84%ED%82%A4%ED%85%8D%EC%B2%98-%EA%B5%AC%EC%A1%B0-%EB%B0%8F-%EC%A3%BC%EC%9A%94-%EA%B0%9C%EB%85%90)