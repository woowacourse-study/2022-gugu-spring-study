# MVC2 : 메시지, 커맨드 객체 검증

# MessageSource

> Strategy interface for resolving messages, with support for the parameterization and internationalization of such messages.

`MessageSource`는 메시지 국제화를 지원하는 인터페이스이다. 스프링은 Locale(지역)에 상관없이 일관되게 문자열을 관리할 수 있도록 지원하고 있다. MessageSource의 getMessage() 메서드를 이용해 특정 Locale에 대응하는 메시지를 가져올 수 있다.

```java
String getMessage(String code, @Nullable Object[] args, @Nullable String defaultMessage, Locale locale);
```
> **code** - 메시지 프로퍼티의 key값  
**Object[] args** - 메시지 내 인덱스 기반 변수 전달 ex. "{0}"  
**defaultMessage** - code에 대응하는 key값이 없을 때 출력하는 메시지  
**locale** - 로케일
  

## 구현체

1. **ResourceBundleMessageSource**: built on top of the standard ResourceBundle, sharing its limitations.
2. **ReloadableResourceBundleMessageSource**: highly configurable, in particular with respect to reloading message definitions.

## 사용방법

1. 메시지 파일 작성
- src/main/resources 디렉토리에 [파일이름]_[언어코드]_[국가코드].properties 형식으로 메시지 파일 추가

> 예시  
message.properties
기본 메시지, 시스템의 언어 및 지역에 맞는 프로퍼티 파일 존재하지 않을 경우 사용  
message_en.properties : 영어 메시지  
message_ko.properties : 한글 메시지  
message_en_UK.properties : 영국을 위한 영어 메시지  
   
  - key-value 형식의 값 입력

> key=value, {index}  
✔ {index} : argument가 바인딩 되는 부분  
ex) greeting=Hello, so good {0}  
ex) greeting=안녕, 진짜 반가워 {0}  

2. **MessageSource** 빈 등록

Spring Boot를 사용할 경우 `ResourceBundleMessageSource`가 빈으로 자동등록되기에 별도로 등록할 필요가 없다.

빈을 등록할 경우 빈의 아이디를 `messageSource`로 지정해야만 한다. 다른 이름을 사용할 경우 인식하지 못한다.

```java
@Configuration
@EnableWebMvc
public class MvcConfig implements WebMvcConfigurer {
	@Bean
	public MessageSource messageSource() {

		var messageSource = new ResourceBundleMessageSource();
		messageSource.setBasename("messages");  //메시지 리소스 번들 지정
		messageSource.setBasename("messages.label");  //src/main/resources/messages 패키지 내 label.properties 파일 읽기
		
		messageSource.setDefaultEncoding("UTF-8");  //문자 집합 설정 - 한글 깨짐 방지
		
		return messageSource;
	}
}
```
`setBasename` : 사용할 메시지 프로퍼티 목록을 전달한다.
  
3. MessageSource를 통해 각 Locale에 맞는 메시지를 출력한다.

```java
@Component
public class AppRunner implements ApplicationRunner {
 
    @Autowired
    MessageSource messageSource;
 
    @Override
    public void run(ApplicationArguments args) throws Exception {

        System.out.println(messageSource.getMessage("greeting", new String[]{"java"}, Locale.US));
        System.out.println(messageSource.getMessage("greeting", new String[]{"java"}, Locale.KOREA));
    }
}
```

실행 결과
> 안녕, 진짜 반가워 java  
Hello, so good java

# 커맨드 객체의 값 검증과 에러 메시지 처리

입력한 값에 대한 검증, 에러 메시지 처리  
스프링은 이 문제를 해결하기 위해 `커맨드 객체를 검증하고 결과를 에러 코드로 저장`하는 방법을 제공한다.

## Validator

객체를 검증할 때 사용하는 `Validator` 인터페이스는 다음과 같다.

```java
public interface Validator {
    boolean supports(Class<?> clazz);
    void validate(Object target, Errors errors);
}
```

> **supports** - Validator가 검증할 수 있는 타입인지 검사  
**validate** - 첫 번째 파라미터로 전달받은 객체를 검증하고 오류 결과를 Errors에 저장   

## 사용방법

입력받은 이메일이 정규표현식과 맞는지 일치하는 예시이다.  
먼저 생성자에서 패턴을 생성한다.


```java
public class RegValidator implements Validator{

    private static final String emailRegex = 
		    "^[_A-Za-z0-9-\\+]+(\\.[_A-Za-z0-9-]+)*@" +
		    "[A-Za-z0-9-]+(\\.[A-Za-z0-9]+)*(\\.[A-Za-z]{2,})$";
    private Pattern pattern;

    public RegValidator() {
	    pattern = Pattern.compile(emailRegex);
    }
```

```java
@Override
public boolean supports(Class<?> clazz) {
	return RegisterRequest.class.isAssignableFrom(clazz);
}
```

supports()는 파라미터로 전달받은 clazz 객체가 RegisterRequest클래스로 타입이 변환 될 수 있는지를 검사한다. 스프링 MVC의 자동 검증을 수행하도록 설정하기 위해서는 올바르게 구현해야한다.

```java
@Override
public void validate(Object target, Errors errors) {
	System.out.println("RegValidator#validate(): " + this);
	RegisterRequest regReq = (RegisterRequest) target; // 검증하려는 객체를 커맨드 객체 타입으로 변환
	if (regReq.getEmail() == null || regReq.getEmail().trim().isEmpty()) {
		errors.rejectValue("email", "required"); //값이 없으므로 email 프로퍼티에 required라는 에러코드 추가
	} else {
		Matcher matcher = pattern.matcher(regReq.getEmail());
		if (!matcher.matches()) {
			errors.rejectValue("email", "bad"); //값이 정규표현식과 맞지 않으므로 email 프로퍼티에 bad라는 에러코드 추가
		}
	}
	ValidationUtils.rejectIfEmptyOrWhitespace(errors, "name", "required"); // error.getFieldValue("name")을 통해 커맨드 객체의 name 프로퍼티 값을 구함
	ValidationUtils.rejectIfEmpty(errors, "password", "required");
	ValidationUtils.rejectIfEmpty(errors, "confirmPassword", "required");
	if (!regReq.getPassword().isEmpty()) {
		if (!regReq.checkConfirmPassword()) {
			errors.rejectValue("confirmPassword", "notmatch");
		}
	}
}
```

`Validate()` 메서드는 먼저 검사 대상을 구하기 위해 전달받은 target을 커맨드 객체의 타입으로 변환한 뒤 값을 검사한다. 위에서는 'email' 프로퍼티 값이 null이나 빈 문자열이나 공백인지 검사해서 걸리면 에러 코드로 'require'를 추가하고, 아니라면 패턴을 이용해서 정규표현식으로 이메일 값이 올바른지 검사하고 올바르지 않으면 에러 코드로 'bad'를 추가한다.

`ValidationUtils` 클래스를 이용해서 객체의 값 검증 코드를 간결하게 작성할 수도 있다.

`rejectIfEmptyOrWhitespace`는 지정한 프로퍼티의 값이 널이나 빈 문자열이나 공백일 경우 지정한 에러 코드를 추가한다.

`rejectIfEmpty`는 지정한 프로퍼티의 값이 널이나 빈 문자열의 경우 지정한 에러 코드를 추가한다.

> **ValidationUtils가 검증하려는 객체인 target을 인자로 전달하지 않았는데에도 불구하고 검증할 수 있는 이유** :  `ValidationUtils`에서 커맨드 객체의 프로퍼티 값을 검사할 수 있는 이유는 파라미터인 Error 타입 객체의 getFieldValue() 메서드로 커맨드 객체의 값을 참조하기 때문이다.

### 컨트롤러에서 커맨드 객체 검증

```java
@PostMapping("/register/step2")
public String handleStep2(RegisterRequest regReq, Errors errors) {
  new RegValidator().validate(regReq, errors);
  if (errors.hasErrors())
    return "register/step1";

  try {
    memberRegisterService.regist(regReq);
    return "register/step2";
  } catch (DuplicateMemberException ex) {
    errors.rejectValue("email", "duplicate");
    return "register/step1";
  }
}
```

> 주의할점 : 컨트롤러에서 Error 타입 파라미터는 커맨드 객체 뒤에 위치해야 한다.


### 커맨드 객체 자체에 에러코드 추가

 만약 로그인이 실패한다면 ID가 잘못되었거나 PW가 잘못된 것 두 가지이므로, 한 가지 프로퍼티를 대상으로하는 에러는 적절하지 않다. 이럴 땐 `rejectValue()`가 아닌 `reject()`를 사용해서 커맨드 객체의 프로퍼티가 아닌 커맨드 객체 자체에 에러를 추가해야 한다.

> errors.reject("notMatchIdPassword");

### BindingResult

Errors 대신 BindingResult 인터페이스를 파라미터 타입으로 사용해도 된다.

```java
@PostMapping("/register/step2")
public String handleStep2(RegisterRequest regReq, BindingResult errors) {
  //생략
}
```

## Errors와 ValidationUtils 주요 메서드

### Errors 주요 메서드

- reject(String errorCode)  
- reject(String errorCode, String defaultMessage) 
- reject(String errorCode, Object[] errorArgs, String defaultMessage)  
- rejectValue(String field, String errorCode)  
- rejectValue(String field, String errorCode, String defaultMessage)  
- rejectValue(String field, String errorCode, Object[] errorArgs, String defaultMessage)  

### ValidationUtils 주요 메서드

-	rejectIfEmpty(Errors errors, String field, String errorCode)
- rejectIfEmpty(Errors errors, String field, String errorCode, Object[] errorArgs)
-	rejectIfEmpty(Errors errors, String field, String errorCode, Object[] errorArgs, String defaultMessage)
-	rejectIfEmpty(Errors errors, String field, String errorCode, String defaultMessage)
-	rejectIfEmptyOrWhitespace(Errors errors, String field, String errorCode)
-	rejectIfEmptyOrWhitespace(Errors errors, String field, String errorCode, Object[] errorArgs)
-	rejectIfEmptyOrWhitespace(Errors errors, String field, String errorCode, Object[] errorArgs, String defaultMessage)
-	rejectIfEmptyOrWhitespace(Errors errors, String field, String errorCode, String defaultMessage)

# 글로벌 범위 Validator와 컨트롤러 범위 Validator

스프링 MVC는 모든 컨트롤러에서 적용할 수 있는 글로벌 Validator와 단일 컨트롤러에 적용할 수 있는 Validator를 제공한다.

## 글로벌 범위 Validator 설정과 @Valid 애노테이션
글로벌 범위 Validator를 적용하려면 다음 두가지를 설정해야 한다..

1. 설정클래스에서 WebMvcConfigurer의 getValidator() 메서드가 Validator 구현 객체를 리턴하도록 구현
2. 검증할 커맨드 객체에 @Valid 애노테이션 적용

```java
@Configuration
@EnableWebMvc
public class MvcConfig implements WebMvcConfigurer {
  @Override
  public Validator getValidator() {
      return new RegisterRequestValidator();
  }
}
```

```java
@Controller
public class RegisterController {
  @PostMapping("/register/step3")
  public String handleStep3(@Valid RegisterRequest registerRequest, Errors errors) {
      if (errors.hasErrors()) {
          return "register/step2";
      }
      try {
          registerService.register(registerRequest);
          return "register/step3";
      } catch (Exception e) {
          errors.rejectValue("email", "myduplicate");
          return "register/step2";
      }
  }
}
```

## @InitBinder를 통한 컨트롤러 범위 Validator 설정

```java
@Controller
public class RegisterController {
    @PostMapping("/register/step3")
    public String handleStep3(@Valid RegisterRequest registerRequest, Errors errors) {
        ...
    }

    @InitBinder
    public void initBinder(WebDataBinder binder) {
        StringTrimmerEditor stringTrimmerEditor = new StringTrimmerEditor(true);
        binder.registerCustomEditor(String.class, stringTrimmerEditor);
    }
}
```

# Bean Validation을 이용한 값 검증 처리

## 사용법

```java
@RestController
public class TestController {

    @PostMapping("/user")
    public ResponseEntity<String> savePost(@Valid @RequestBody UserDto userDto) {
        log.info(userDto.toString());
        return ResponseEntity.ok().body("postDto 객체 검증 성공");
    }
}
```

```java
public class UserDto {

    @NotNull
    private String name;
    
    @Email
    private String email;
}
```

> 주의사항 : @Size, @Email 사용시에도 @NotNull을 붙여줘야 한다.

## @Valid시 발생하는 Exception Handling 

MethodArgumentNotValidException 에러를 @ExceptionHandler로 받아 처리하면 된다.

```java
@ControllerAdvice
public class ApiControllerAdvice {

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<Map<String, String>> handleValidationExceptions(MethodArgumentNotValidException ex){
        Map<String, String> errors = new HashMap<>();
        ex.getBindingResult().getAllErrors()
                .forEach(c -> errors.put(((FieldError) c).getField(), c.getDefaultMessage()));
        return ResponseEntity.badRequest().body(errors);
    }
}
```