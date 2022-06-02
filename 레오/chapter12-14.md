# 스프링 프로그래밍 입문

# 12장 메세지, 커맨드 객체 검증

## 글로벌 범위 Validator 설정과 @Valid

### 글로벌 범위 Validator 설정

- 설정 클래스에서 WebMvcConfigurer의 getValidator() 메서드가 Validator 구현 객체를 리턴하도록 구현
- 글로벌 범위 Validator가 검증할 커맨드 객체에 @Valid 어노테이션 적용

```java
@Configuragtion
@EnableWebMvc
public class MvConfig implements WebMvcConfigurer {
	@Override
	public Validator getValidator() {
		return new RegisterRequestValidator();
	}
}
```

### @InitBinder

컨트롤러에서 @InitBinder가 붙은 메서드는 컨트롤러 범위에 적용할 Validator를 설정할 수 있다.

```java
@Controller
public class RegisterController {
	@InitBinder
	protected void initBinder(WebDataBinder binder) {
		binder.setValidator(new RegisterRequestValidator());
	}
}
```

## Bean Validation을 이용한 값 검증 처리

### 주요 @어노테이션

- AssertTrue, AssertFalse: 값이 true/false 인가? null은 통과
- DecimalMax, DecimalMin: 지정한 값 보다 큰/작은가? inclusive(지정값 포함 여부) : boolean 기본값 true
- Max, Min: 지정한 값보다 큰/작은가? null은 통과 value : long
- Digits: 자릿수가 지정한 크기를 넘지 않는지 검사한다. null은 통과 integer(정수 자릿수) : int, fraction(소수 자릿수) : int
- Size: 길이 또는 크기가 지정한 값 범위에 있는지 검사한다. null은 통과 min : int, max : int
- Null, NotNull: 값이 null 인지/아닌지 검사한다.
- Pattern: 값이 정규표현식에 일치하는지 검사한다. null은 통과 regexp : String
- NotEmpty: null과 길이가 또는 크기가 0이 아닌지 검사한다.
- NotBlank: null과 공백 문자만 있지 않은지 검사한다.
- Positive, PositiveOrZero: 0초과/이상인지 확인한다. null은 유효
- Negative, NegativeOrZero: 0미만/이하인지 확인한다. null은 유효
- Future, FutureOrPresent: 미래/현재 또는 미래시간인지 검사한다. null은 유효
- Past, PastOrPresent: 현재/현재 또는 과거인지 검사한다. null은 유효

# 13장 세션, 인터셉터, 쿠키

## 컨트롤러에서 HttpSession 사용하기

로그인 상태를 유지하는 방법은 HttpSession과 쿠키 (또는 토큰) 방식이 있다.

컨트롤러에서 HttpSession을 사용하려면 다음의 두 가지 방법 중 한 가지를 사용하면 된다.

- 요청 매핑 메서드에 HttpSession 파라미터를 추가한다. 항상 세션을 생성한다.

```java
@PostMapping
public String form(HttpSession session) {
}
```

- 요청 매핑 메서드에 HttpServletRequest 파라미터를 추가하고 HttpServletRequest를 이용해서 HttpSession을 구한다. 필요한 시점에만 세션을 생성할 수 있다.

```java
@PostMapping
public String submit(HttpServletRequest request) {
	HttpSession session = request.getSession();
}
```

로그아웃을 하려면 `session.invalidate();` 를 실행하면 된다. 서버를 재시작하면 세션 정보가 유지되지 않는다.

## 인터셉터 사용하기

다수의 컨트롤러에 대해 동일한 기능을 적용해야 할 때 HandlerInterceptor를 사용한다.

### HandlerInterceptor 인터페이스 구현하기

- 컨트롤러 실행 전

`preHandle()` 메서드는 컨트롤러 객체를 실행하기 전에 필요한 기능을 구현할 때 사용한다. false를 리턴하면 컨트롤러를 실행하지 않는다.

```java
default boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler)
      throws Exception {

   return true;
}
```

- 컨트롤러 실행 후, 아직 뷰를 실행하기 전

`postHandle()` 메서드는 컨트롤러가 정상적으로 실행된 이후에 추가 기능을 구현할 때 사용한다. 컨트롤러에서 예외가 발생하면 실행되지 않는다.

```java
default void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler,
			@Nullable ModelAndView modelAndView) throws Exception {
	}
```

- 뷰를 실행한 이후

afterCompletion() 메서드는 뷰가 클라이언트에 응답을 전송한 뒤에 실행된다. 컨트롤러 실행 과정에서 예외가 있을 경우에만 메서드의 네번째 파라미터에 예외가 전달된다.

```java
default void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler,
      @Nullable Exception ex) throws Exception {
}
```

HandlerInterceptor를 구현하면 HandlerInterceptor를 어디에 적용해야 할지 설정해야 한다. `AuthCheckIntercceptor()` 가 Bean 객체이면 다음과 같이 설정한다.

```java
// MvcConfig.java

@Override
public void addInterceptors(InterceptorRegistry registry)) {
	registry.addInterceptor(return new AuthCheckIntercceptor())
		.addPathPatterns("/edit/**")
		.excludePathPatterns("/edit/help/**");
}
```

### Ant 경로 패턴

- *: 0개 이상의 글자
- ?: 1개의 글자
- ** : 0개 이상의 폴더 경로

## 컨트롤러에서 쿠키 사용하기

사용자 편의를 위해 아이디를 기억해 두었다가 다음에 로그인할 대 아이디를 자동으로 넣어주는 기능을 구현하는 등 미리 인증정보 등을 저장해 놓을 필요가 있을 때 쿠키를 사용한다. @CookieValue를 파라미터에 붙여 사용할 수 있다. 이메일 기억하기 기능을 구현하는 방식이다.

- 로그인 폼에 ‘이메일 기억하기' 옵션을 추가한다.
- 로그인 시에 ‘이메일 기억하기’ 옵션을 선택했으면 로그인 성공 후 쿠키에 이메일을 저장한다.
- 이후 로그인 폼에서 쿠키가 존재하면 입력 폼에 이메일을 보여준다.

# 14장 @PathVariable, 예외 처리

## @PathVariable을 이용한 경로 변수 처리

uri의 {경로변수} 중괄호로 둘러 쌓인 부분을 경로변수라고 한다. 타입에 맞게 알아서 변환된다. 변수명이 경로변수와 일치하면 이름설정을 생략할 수 있다.

```java
@GetMapping("members/{id}")
public String detail(@PathVariable("id") Long id) {
}
```

## 컨트롤러 예외 처리하기

같은 컨트롤러 클래스에 `@ExceptionHandler`를 적용한 메서드가 존재하면 그 메서드가 예외를 처리한다.

```java
@ExceptionHandler(Exception.class)
public ResponseEntity<Void> handle(Exception e) {
	return ResponseEntity().badRequest().body(new ErrorResponse(e.getmessage));
}
```

`@ControllerAdvice(”패키지 이름")` 은 지정한 범위의 컨트롤러에 handle을 적용한다. 이때 `@ControllerAdvice`이 있는 해당 클래스는 Bean으로 등록해야 한다. `@ExceptionHandler` 의 우선 순위는 같은 클래스 → `@ControllerAdvice` 순 이다.

### `@ExceptionHandler` 파라미터와 리턴 타입

**Parameter types**

- HttpServletRequest, HttpServletResponse, HttpSession
- Model
- Exception

**Return types**

- MoelAndView
- “view name” : String
- @ResponseBody 임의 객체
- ResponseEntity
