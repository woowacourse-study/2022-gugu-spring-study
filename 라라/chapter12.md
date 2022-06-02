# 12장. 메시지, 커맨드 객체 검증

## 글로벌 범위 Validator

- 글로벌 범위 Validator 는 모든 컨트롤러에 적용할 수 있는 Validator이다.
- 적용 방법
    - 설정 클래스에서 `WebMvcConfigure`의 `getValidator()` 메서드가 `Validator` 구현 객체를 리턴하도록 구현
    - 글로벌 범위 Validator 가 검증할 커맨드 객체에 `@Valid` 어노테이션 적용

```java
// 글로벌 범위 Validator 설정을 위한 WebMvcConfigurer 인터페이스의 getValidator() 메서드 구현
@Configuration
@EnableWebMvc
public class MvcConfig implements WebMvcConfigurer {

    @Override
    public Validator getValidator() {
        return new RegisterRequestValidator();
    }
}
```

- 스프링 MVC 는 WebMvcConfigurer 인터페이스의 getValidator() 메서드가 리턴한 객체를 글로벌 범위 Validator 로 사용한다.
- RegisterRequestValidator는 RegisterRequest 타입을 지원한다.

```java
@GetMapping
public String handleStep3(@Valid RegisterRequest request, Errors errors) {
    if (errors.hasErrors()) {
        return "register/step2"
    }
    // ...
}
```

- 검증 수행 결과는 `Errors` 타입 파라미터로받는다.
- 이 과정은 컨트롤러(요청 처리) 메소드가 실행되기 전에 이루어지므로 컨트롤러 메소드에서는 RegisterRequest 객체를 검증할 필요가 없다.
- 하지만 파라미터에 Errors 타입 파라미터가 없으면 검증 실패시 400 에러를 응답한다.
- RegisterRequestValidator 클래스는 RegisterRequest 타입의 객체만 검증할 수 있으므로 모든 컨트롤러에 적용할 수 있는 글로벌 범위 Validator 로 적합하지 않다. 스프링 MVC 는 자체적으로 제공하는 글로벌 Validator 가 존재하는데 이 Validator를 사용하면 Bean Validation 이 제공하는 어노테이션을 이용해서 값을 검증할 수 있다.

## @InitBinder 어노테이션을 이용한 컨트롤러 범위 Validator

`@InitBinder` 어노테이션을 이용하면 컨트롤러 범위 Validator를 설정할 수 있다.

`WebDataBinder.setValidator()` 메서드를 이용해서 컨트롤러 범위에 적용할 Validator 를 적용할 수 있다.

```java
@Controller
public class RegisterController {
		// ...

    @InitBinder
    protected void initBinder(WebDataBinder binder) {
        binder.setValidator(new RegisterRequestValidator());
    }
}
```

- `@Valid` 애노테이션이 붙은 `RegisterRequest` 객체를 검증할 때 `RegisterRequestValidator` 를 사용한다.
- `@InitBinder` 이 붙은 메서드는 컨트롤러의 요청 처리 메서드를 실행하기 전에 매번 실행된다. a(), b() c() 메서드를 실행하기 전에 `initBinder()` 메서드를 매번 호출해서 `WebDataBinder` 를 초기화한다.

## Bean Validation을 이용한 값 검증 처리

- `@Valid` 어노테이션은 Bean Validation 스펙에 정의되어 있다.
- `@NotNull` , `@Digits` , `@Size` 등의 어노테이션을 정의하고 있다.
- 해당 어노테이션으로 Validator 작성 없이 어노테이션만으로 커맨드 객체의 값 검증을 처리할 수 있다.
    - 즉, Validator 등의 클래스를 만드는 번거로운 과정 없이 핵심 로직에만 집중할 수 있다.

**사용 방법**

Spring boot 에서 의존성 추가

```java
implementation 'org.springframework.boot:spring-boot-starter-validation'
```

주요 어노테이션

- `@AssertFalse`
- `@AssertTrue`
- `@DecimalMax`
- `@DecimalMin`
- `@Digits`
- `@Email`
- `@Future`
- `@FutureOrPresent`
- `@Max`
- `@Min`
- `@Negative`
- `@NegativeOrZero`
- `@NotBlank`
- `@NotEmpty`
- `@NotNull`
- `@Null`
- `@Past`
- `@PastOrPresent`
- `@Pattern`
- `@Positive`
- `@PositiveOrZero`
- `@Size`