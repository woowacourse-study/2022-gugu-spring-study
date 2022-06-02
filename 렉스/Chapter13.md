# Chapter 13
## 컨트로러의 HttpSession
1. 요청 매핑 어노테이션 적용 메서드에 HttpSession 파라미터를 추가한다.
2. 요청 매핑 어노테이션 적용 메서드에 HttpServletRequest 파라미터를 추가하고 HttpServletRequest를 이용하여 HttpSession을 구한다.

### 세션 사용법
방법 1. Controller 메서드에 HttpSession을 파라미터로 설정하는 방법
   - HttpSession을 생성하기 전이면 새로운 HttpSession을 생성하고 그렇지 않으면 기존 HttpSession을 전달한다.
   - 항상 HttpSession을 생성한다.
```java
@PostMapping
public String form(LonginRequest request, HttpSession session) {
        ...    
}
```
방법 2. HttpServletRequest.getSession()을 통해 가져오은 방법
   - 필요한 시점에만 HttpSession을 생성할 수 있다.
```java
@PostMapping
public String form(LonginRequest request, HttpServletRequest servletRequest) {
        HttpSession session = servletRequest.getSession();
}
```

## 인터셉터 (Interceptor)
웹 어플리케이션에서 인증이 필요한 모든 컨트롤러 코드마다 세션 코드를 삽입하면 많은 중복을 일으킨다.

다수의 컨트롤러에 대해 동일해야 할 때 사용할 수 있는 것이 HandlerInterceptor이다.

인터셉터에는 3가지 메서드가 있다.
- `preHandle()`
  - 컨트로러 실행 전에 실행된다.
  - return 타입은 boolean이다. (false를 반환하면 다음 Interceptor가 실행되지 않는다.)
- `postHandle()`
  - 컨트롤러 실행 후, 아직 뷰를 실행하기 전에 실행된다.
  - 컨트롤러(핸들러)가 정상적으로 실행된 이후에 추가 기능을 구현할 때 사용한다.
  - 컨트롤러가 예외를 발생시키면 해당 메서드는 실행되지 않는다.
- `afterCompletion()`
  - 뷰를 실행한 이후 실행된다.
  - 컨트롤러 실행 이후에 예상하지 못한 예외를 로그로 남기거나 실행 시간을 기록하는 등의 후처리를 하기 위해 사용된다.

인터페이스인 HandlerInterceptor의 위 세가지 메서드는 자바의 디폴트 메서드로 꼭 모든 것을 재정의할 필요가 없다.
필요한 것들만 재정의하여 사용하면 된다.

### HandlerInterceptor 설정
`HandlerInterceptor`를 구현하여 만든 인터셉터는 아래와 같이 `WebMvcConfigurer`의 `addInterceptors()`메서드를 재정의하며 추가할 수 있다.
해당 메서드에서는 `addPathPatterns()`를 통해 해당 인터셉터를 적용시킬 주소를 정할 수 있으며, `excludePathPatterns()`를 통해 경로 중 제외하고 싶은 경로를 지정할 수 있다.
```java
@Configuration
@EnableWebMvc
public class MvcConfig implements WebMvcConfigurer {
    ...
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(authCheckInterceptor())
                .addPathPatterns("/edit/**")
                .excludePathPatterns("/edit/help/**");
    }
    ...
}
```

## 컨트롤러의 쿠키
### 구현 방법
- 로그인 폼에 "이메일 기억하기" 옵션을 추가한다.
- 로그인 시에 "이메일 기억하기" 옵션을 선택했으면 로그인 성공 후 쿠키에 이메일을 저장한다. 이 때 쿠키는 웹 브라우저를 닫더라도 삭제되지 않도록 유효시간을 길게 설정한다.
- 이후 로그인 폼을 보여줄 때 이메일을 저장한 쿠키가 존재하면 입력 폼에 이메일을 보여준다.

### 쿠키 사용하기
스프링 MVC에서 쿠키를 사용하는 방법 중 하나는 `@CookieValue` 어노테이션을 사용하는 것이다.
해당 어노테이션은 컨트롤러에 Cookie타입의 파라미터를 적용하면 쉽게 쿠키를 파라미터로 전달받게 할 수 있다.
```java
@RequestMapping("/login")
public class LoginController {
    private AuthService authService;

    public void setAuthService(AuthService authService) {
        this.authService = authService;
    }

    @GetMapping
    public String form(LoginCommand loginCommand,
                       @CookieValue(value = "REMEMBER", required = false) Cookie rCookie) {
        if (rCookie != null) {
            loginCommand.setEmail(rCookie.getValue());
            loginCommand.setRememberEmail(true);
        }
        return "login/loginForm";
    }

    @PostMapping
    public String submit(
            LoginCommand loginCommand, Errors errors, HttpSession session,
            HttpServletResponse response) {
        new LoginCommandValidator().validate(loginCommand, errors);
        if (errors.hasErrors()) {
            return "login/loginForm";
        }
        try {
            AuthInfo authInfo = authService.authenticate(
                    loginCommand.getEmail(),
                    loginCommand.getPassword());

            session.setAttribute("authInfo", authInfo);

            Cookie rememberCookie =
                    new Cookie("REMEMBER", loginCommand.getEmail());
            rememberCookie.setPath("/");
            if (loginCommand.isRememberEmail()) {
                rememberCookie.setMaxAge(60 * 60 * 24 * 30);
            } else {
                rememberCookie.setMaxAge(0);
            }
            response.addCookie(rememberCookie);

            return "login/loginSuccess";
        } catch (WrongIdPasswordException e) {
            errors.reject("idPasswordNotMatching");
            return "login/loginForm";
        }
    }
}
```

쿠키의 생성은 로그인시에 만들어지며 위의 submit 메서드와 같이 생성하면 된다.
