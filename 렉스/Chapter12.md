뷰에서 여러 페이지에 중복된 내용들이 존재하는 경우 해당 내용이 변경되었을 때, 여러 내용들을 변경해줘야한다는 유지보수 측면의 문제가 있다.

Spring에서는 `MessageSource`빈과 `<spring:message>` 태그를 통해 해당 문제를 쉽게 해결할 수 있다.
- 문자열을 담은 메시지 파일을 작성한다.
- 메시지 파일에서 값을 읽어오는 `MessageSource` 빈을 설정한다.
  - 주의할 점: 빈의 아이디를 무조건 messageSource로 설정하여야한다. 그러지 않으면 정상적으로 동작하지 않는다.
- JSP 코드에서 `<spring:message = ~~>` 태그를 사용하여 메시지를 출력한다.

# MessageSource
스프링에서는 로케일(지역)에 상관없이 일관된 방법으로 문자열을 관리할 수 있는 MessageSource인터페이스를 정의하고 있다.
덕분에 같은 코드라도 지역에 따라 다른 메시지를 제공할 수 있다. 특정 로케일에 해당하는 메시지가 필요한 코드는 `getMessage()`메서드를 이용해 필요한 메시지를 가녀올 수 있다.

`<spring:message = ~~>` 태그는를 실행하면 스프링 설정에 등록된 `MessageSource` 빈의 `getMessage()`메서드를 이용해서 메시지를 구하게 된다.

# Validation
스프링 MVC는 모든 컨트롤러에 적용할 수 있는 글로벌 Validator와 단일 컨트롤러에 적용할 수 있는 Validator를 설정하는 방법을 제공하고 있다.

## 글로벌 범위의 Validator
**적용 순서**
1. 설정 크래스에서 `WebMvcConfigurer`의 `getValidator()`메서드가 Validator 구현 객체를 반환하도록 구현한다.
  - 스프링 MVC는 `getValidator()`가 반환하는 객체를 글로벌 범위의 Validator로 사용한다. 이를 지정하면 `@Valid`를 통해 검증을 적용할 수 있다.
```java
@Configuration
@EnableWebMvc
public class MvcConfig implements WebMvcConfigurer {

    @Override
    public Validator getValidator() {
        return new RegisterRequestValidator();
    }
    ...
}
```  

2. 글로벌 범위 Validator가 검증할 커맨드 객체에 `@Valid` 어노테이션을 적용한다.
   - 해당 어노테이션이 붙으면 앞에서 지정한 글로벌 범위의 Validator가 해당 타입을 요청을 처리하는 메서드(컨트롤러 메서드) 실행 전에 검증을 하게 된다. 
   - @Valid를 사용할 때 주의점: Error 타임 파라미터가 없으면 검증 실패시 400 에러를 응답한다.
```java
@Controller
@RequestMapping("/local")
public class RegisterControllerWithLocalValidator {
    
    @PostMapping("/register/step3")
    public String handleStep3(@Valid RegisterRequest regReq, Errors errors) {
        if (errors.hasErrors())
            return "register/step2";

        try {
            memberRegisterService.regist(regReq);
            return "register/step3";
        } catch (DuplicateMemberException ex) {
            errors.rejectValue("email", "duplicate");
            return "register/step2";
        }
    }
}
```


> @Valid 어노테이션은 Bean Validation API에 포함되어 있다.
> - 의존성 추가 방법:
>    - Javax 의존성: `implementation 'javax.validation:validation-api:2.0.1.Final'`
>    - 스프링 부트 Validation 의존성: `implementation 'org.springframework.boot:spring-boot-starter-validation'`

## 컨트롤러 범위의 Validator
`@InitBinder`를 사용하면 컨트롤러 범위의 Validator를 설정할 수 있다. (p.344)

```java
@Controller
@RequestMapping("/local")
public class RegisterControllerWithLocalValidator {
    ...
    @InitBinder
    protected void initBinder(WebDataBinder binder) {
        binder.setValidators(new RegisterRequestValidator());
    }
}
```
`@InitBinder` 어노테이션을 적용한 메서드는 WebDataBinder를 매개변수로 받는데 `setValidators()`를 이용해 컨트롤러 범위에 적용할 Validator를 설정할 수 있다.

위의 코드로 설명을 하면 `@InitBinder`이 붙은 메서드는 요청에 맞는 컨트롤러 메서드가 실행되기 전에 호출되어 매번 `setValidators()`을 통해 설정한 Validator로 `WebDataBinder`를 초기화한다.

> `WebDataBinder`는 내부적으로 Validator의 목록을 갖는다. 
> - `setValidators()`를 실행하면 해당 컨트롤러에 사용될 Validator를 지정한다.
> - `addValidators()`는 글로벌 범위의 Validator 뒤에 새로운 컨트롤러 범위의 Validator를 추가한다.

## Bean Validation 어노테이션
- `@AssertTrue`, `@AssertFalse`: 값이 true인지 false인지 검사한다. (null은 유효하다고 판단)
- `@DecimalMax`, `@DecimalMin`: 지정한 값보다 크거나 같은지, 작거나 같은지 확인한다. (null은 유효하다고 판단)
- `@Max`, `@Min`: 지정한 값보다 작은지, 큰지 검사한다. (null은 유효하다고 판단)
- `@Digit`: 자릿수가 지정한 크기를 넘지 않는지 검사한다. (null은 유효하다고 판단)
- `@Size`: 길이나 크기가 지정한 값 범위에 있는지 검사한다. (null은 유효하다고 판단)
- `@Null`, `@NotNull`: 값이 null인지, null이 아닌지 검사한다.
- `@Pattern`: 값이 정규 표현식에 일치하는지 검사한다. (null은 유효하다고 판단)
- `@NotEmpty`: 문자열이나 배열의 경우 null이 아니고 길이가 0이 아닌지 검사한다. 컬렉션의 경우 null이 아니고 크기가 0인지 검사한다.
- `@NotBlank`: null이 아니고 최소한 한 개 이상의 공백이 아닌 문자를 포함하는지 검사한다.
- `@Email`: 이메일 주소가 유효하지 검사한다.
