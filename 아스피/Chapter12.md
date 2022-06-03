# Chp 12. MVC 2: 메세지, 커맨드 객체 검증

---

1. 메세지
    1. resources에 메세지 파일을 작성한다.
    2. MessageSource 차입의 빈을 추가한다.
    3. jsp파일에서 <spring: message> 태그를 이용하여 메시지를 출력한다.

<br>

2. 커맨드 객체의 값 검증
    1. Validator 인터페이스를 이용한다.<br>
    ```java
        public interface Validator{
            boolean supports(Class<?> clazz);
            void validate(Object target, Errors errors);
        }
    ```
   supports()는 Validator가 검증할 수 있는 타입인지 검증한다. 대상인 객체를 validate()를 이용해 검증한다.
   <br><br>

    2. validate()는 두개의 파라미터를 받는다. target은 검증 대상 객체이고, errors는 검사 결과 에러코드를 설정하기 위한 객체이다.<br>
   ```java
        errors.rejectValue("email","required");
   
        ValidationUtils.rejectIfEmpty(errors,"email","required");
   ```
   target의 email 프로퍼티를 검증하기위해 위의 두 가지 방법이 있다.<br>
   <br>

    3. 글로벌 범위 Validator와 컨트롤러 범위 Validator<br>
       글로벌 범위 Validator를 이용하면 @Valid 애노테이션을 이용해서 검증이 가능하다.<br>
       글로벌 범위 Validator를 설정하려면 WebMvcConfigurer에 getValidator()를 구현한다. 그 후, 커맨드 객체에 해당하는 파라미터에 @Valid 애노테이션을 붙이면 해당 타입을
       검증할 수 있는 경우, Error에 결과를 저장한다.<br>
       이 때, 유의할 점은 Errors 타입 파라미터를 주지 않으면 400에러가 발생한다.
       <br><br>

    4. Bean Validation을 이용한 값 검증처리도 다양하게 있다(ex. @NotNull)
    
