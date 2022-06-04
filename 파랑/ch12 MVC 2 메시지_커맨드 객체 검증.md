# Chapter 12 MVC 2: 메시지, 커맨드 객체 검증

## MessageSource

### 사용하는 이유

- 문자열이 view 코드에 하드코딩 되어있으면 동일 문자열을 변경할 때 번거롭다.
- 다국어 지원이 불가능하다.

### 해결 방법

- 문자열을 담은 메시지 파일을 작성한다 → UTF-8 인코딩을 사용하여 `label.properties` 작성
- 메시지 파일에서 값을 읽어오는 MessageSource 빈을 설정한다.
    - basenames 프로퍼티 값으로 메시지 파일의 경로를 설정
    - 빈의 아이디는 반드시 `messageSource`
- JSP 코드에서 `<spring:message>` 태크를 사용하여 메시지를 출력한다.
    - 두 개 이상의 값을 전달해야 할 경우다음 방법 중 하나를 사용한다.
        - 콤마로 구분한 문자열
        - 객체 배열
        - `<spring:argument>` 태그 사용

## Validator

커맨드 객체의 값이 올바른지 검사하기 위해서 Validator 인터페이스를 사용한다.

```java
public class RegisterRequestValidator implements Validator {
    
    // clazz 객체가 RegisterRequest 클래스로 타입 변환이 가능한지 스프링이 자동 검증
    @Override
    public boolean supports(Class<?> clazz) {
        return RegisterRequestValidator.class.isAssignableFrom(clazz);
    }
    
    // target을 검증하고 오류 결과를 errors에 담음
    @Override
    public void validate(Object target, Errors errors) {
        RegisterRequest request = (RegisterRequest) target;
        if (/* 이메일 형식이 맞지 않으면 */) {
            errors.rejectValue("email", "bad");
        }
        ValidationUtils.rejectIfEmptyOrWhitespace(errors, "name", "required");
    }
}
```

```java
// RegisterController
@PostMapping("/register/step3")
public void test(RegisterRequest request, Errors errors) {
    new RegisterRequestValidator().validate(request, errors);
    if(errors.hasErrors()) {
        return "register/step2";
    }
    try {
        ...
        return "register/step3";
    } catch (Exception e) {
        errors.rejectValue("email", "duplicate");
        return "register/step2";   
    }
}
```

### @Valid 어노테이션 적용

1. 글로벌 범위 설정

   WebMvcConfigurer 인터페이스에 정의된 getValidator() 메서드 구현

   getValidator() 메서드가 반환한 객체를 글로벌 범위 validator로 사용한다.

2. 컨트롤러 범위 설정

   `@InitBinder` 를 이용한다.


## Bean Validation

- Bean Validation 의존 추가

    ```java
    implementation 'org.springframework.boot:spring-boot-starter-validation'
    ```

- `@EnableWebMvc`

  `OptionalValidatorFactoryBean` 을 글로벌 범위 Validator로 등록한다.

- 어노테이션 종류

  [@Valid 정리해보기](https://velog.io/@gudnr1451/Valid-%EC%A0%95%EB%A6%AC%ED%95%B4%EB%B3%B4%EA%B8%B0)

---

> DTO 검증과 도메인 검증의 중복
도메인의 검증 로직은 필수적이다. DTO 검증은 선택이다. 가장 기초적인(null, blank) 검증만 넣거나 or 도메인과 중복으로 검증하거나.
>
