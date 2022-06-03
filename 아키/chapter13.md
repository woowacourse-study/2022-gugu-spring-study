# MVC3 : 세션, 인터셉터, 쿠키

# 컨트롤러에서 HttpSession 사용하기

로그인 상태를 유지하는 방법은 두 가지 방법이 존재한다.

1. HttpSession를 이용하는 방법
2. 쿠키를 이용하는 방법

컨트롤러에서 HttpSession을 사용하려면 HttpSession, HttpServletRequest 중 하나를 사용하면 된다.

1. 요청 매핑 어노테이션 적용 매서드에 HttpSession 파라미터를 사용
```java
@PostMapping
public String form(LoginCommand loginCommand, Errors errors, HttpSession session){
    ... // session을 사용하는 코드
}
```

2. 요청 매핑 어노테이션 적용 매서드에 HttpServletRequest 파라미터를 추가하고 HttpServletRequest.getSession()을 이용해 HttpSession을 구한다.

```java
  @PostMapping
  public String submit(
      LoginCommand loginCommand, Errors errors, HttpServletRequest req){
    HttpSession session = req.getSession();
      ... // session을 사용하는 코드
  }
```
첫 번째 방법은 항상 HttpSession을 생성하지만, 두 번째 방법은 필요한 시점에만 HttpSession을 생성한다.

> 두 방법 모두 기존에 존재하는 세션이 있을시, 존재하는 세션을 전달한다.

세션 정보는 HttpSession의 setAttribute() 메서드를 통해 저장한다.

```java
// LoginController.java
@Controller
@RequestMapping("/login")
public class LoginController {
    ...
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
            
            // 로그인에 성공 시 HttpSession의 authInfo 속성에 인증 정보 객체(authInfo)를 저장
            session.setAttribute("authInfo", authInfo);

        ...
        } catch (WrongIdPasswordException e) {
            errors.reject("idPasswordNotMatching");
            return "login/loginForm";
        }
    }
}
```

# 인터셉터 사용하기

내정보 조회, 비밀번호 변경 등과 같은 페이지는 로그인한 회원만 접속이 가능하다. 그래서 매번 로그인 여부를 확인해야 한다.  
이렇게 다수의 컨트롤러에 대해 동일한 기능을 적용해야 할 때 사용할 수 있는 것이 `HandlerInterceptor`이다.

> 궁금한점 : AOP와 인터셉터의 차이가 뭘까.

## HandlerInterceptor 구현
org.springframework.web.HandlerInterceptor 인터페이스를 이용해 구현하며 다음과 같은 시점에 공통 기능 삽입 가능하다.  
- 컨트롤러 실행 전
- 컨트롤러 실행 후, 아직 뷰를 실행 전
- 뷰를 실행한 이후

이러한 시점을 처리하기 위해 HandlerInterceptor 인터페이스는 다음 매서드를 정의한다.
- boolean prehandle()
- void postHandle())
- void afterCompletion()

1. preHandle(): 리턴 타입은 boolean으로써, 만약 false를 리턴하게 되면 컨트롤러 또는 다음 핸들러인터셉터를 실행하지 않음  
2. postHandle():컨트롤러가 정상적으로 실행된 이후에 추가 기능을 구현할 때 사용하며, 컨트롤러가 익셉션을 발생하면 postHandle() 매서드는 실행하지 않음  
3. afterCompletion(): 뷰가 클라이언트에 응답을 전송한 뒤에 실행하며,
컨트롤러 실행 이후에 예기치 않게 발생한 익셉션 로그나 실행 시간을 기록하기에 적합  

```java
 // AuthCheckInterceptor.java
 public class AuthCheckInterceptor implements HandlerInterceptor {
     @Override
     public boolean preHandle(
             HttpServletRequest request,
             HttpServletResponse response,
             Object handler) throws Exception {
         HttpSession session = request.getSession(false);
         if (session != null) {
             Object authInfo = session.getAttribute("authInfo");
             if (authInfo != null) {
                 return true;
             }
         }
                 // 인증정보가 없어 실패 시, 다음과 같은 경로로 리다이렉트 시킴
         response.sendRedirect(request.getContextPath() + "/login");
         return false;
     }
 }
```

## HandlerInterceptor 설정
구현한 HandlerInterceptor를 WebMvcConfigurer를 상속받은 MvcConfig 파일에서 어디에 적용할지 설정할 수 있다.

```java
// MvcConfig.java
@Configuration
@EnableWebMvc
public class MvcConfig implements WebMvcConfigurer {
    ...
        // 인터셉트를 정의하는 매서드
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(authCheckInterceptor())
            .addPathPatterns("/edit/**")
            .excludePathPatterns("/edit/help/**");
    }
    ...
}
```

addInterceptor 매서드는 인터셉터를 적용할 경로 패턴을 Ant 경로 패턴을 이용하여 지정한다.

> Ant 경로 패턴?  
Ant 패턴은 *, **, ?의 세 가지 특수 문자를 사용해 경로를 다음과 같이 표현
*: 0개 또는 그 이상의 글자  
?: 1개 글자  
**: 0개 또는 그 이상의 폴더 경로  
따라서 앞선 코드의 경우, http://localhost:8080/sp5-chap13/edit/changePassword 에 접근하면 로그인 폼으로 리다이렉트 된다.

# 컨트롤러에서 쿠키 사용

로그인할 때 이메일을 기억하는 기능을 구현할 경우 쿠키에 저장하는 방식을 이용해 구현한다.  

스프링 MVC에서 쿠키를 사용하는 방법 중 하나는 `@CookieValue` 어노테이션을 사용하는 것이다.  

@CookieValue 어노테이션은 Cookie 타입의 파라미터에 적용한다.  

```java
// LoginController.java
@Controller
@RequestMapping("/login")
public class LoginController {
    @GetMapping
    public String form(LoginCommand loginCommand,
            /*
                * 어노테이션을 통해 쿠키의 이름을 REMEMBER로 지정  
                * 지정한 이름의 쿠키가 없다면, required 속성 값을 false로 지정
                * 만약 지정한 이름의 쿠키가 없는데, required가 ture면 익셉션 발생
                */
            @CookieValue(value = "REMEMBER", required = false) Cookie rCookie) {
        if (rCookie != null) {
            loginCommand.setEmail(rCookie.getValue());
            loginCommand.setRememberEmail(true);
        }
        return "login/loginForm";
    }

    /*
    * 실제 쿠키를 생성하는 부분은 로그인을 처리하는 다음 매서드
    * 쿠키를 사용하기 위해 HttpServletResponse 객체가 필요하므로 파라미터로 전달
    */
    @PostMapping
    public String submit(
            LoginCommand loginCommand, Errors errors, HttpSession session,
            HttpServletResponse response) {
        ...
                // 쿠키를 추가하는 코드
        Cookie rememberCookie = 
                new Cookie("REMEMBER", loginCommand.getEmail());
        rememberCookie.setPath("/");
            
            /*
                * 로그인에 성공했을 때, 이메일 기억하기 체크박스 선택 여부에 따라
                * 30일동안 유지되는 쿠키를 생성하거나
                * 바로 삭제되는 쿠키를 생성
                */
        if (loginCommand.isRememberEmail()) {
            rememberCookie.setMaxAge(60 * 60 * 24 * 30);
        } else {
            rememberCookie.setMaxAge(0);
        }
        response.addCookie(rememberCookie);
        ...
    }
}
```
