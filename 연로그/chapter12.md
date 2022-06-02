# Chapter 12 - 메시지, 커맨드 객체 검증

### Validator

- supports(): Validator가 검증할 수 있는 타입인지 검사
- validate(): target을 검증하고 오류 결과를 errors에 담기

```java
public interface Validator {
    boolean supports(Class<?> clazz);
    void validate(Object target, Errors errors);
}
```

#### 적용하기

```java
public class TestValidator implements Validator {
    
    @Override
    public boolean supports(Class<?> clazz) {
        return TestRequest.class.isAssignableFrom(clazz);
    }
    
    @Override
    public void validate(Object target, Errors errors) {
        TestRequest testRequest = (TestRequest) target;
        ValidationUtils.rejectIfEmptyOrWhitespace(errors, "value", "required");
        if (testRequest.getValue().length() < 10) {
            errors.rejectValue("value", "bad");
        }
    }
}
```

```java
// Controller
@PostMapping("/test")
public void test(TestRequest testRequest, Errors errors) {
    new TestValidator().validate(testRequest, errors);
    if(errors.hasErrors()) {
        throw new IllegalArgumentException();
    }
}
```

#### 궁금한 점
- 나는 실행 안되던데 실습 성공한 사람이 있을까?
- Validator를 직접 구현해서 쓸 일이 있을까?

### Bean Validation

- Bean Validation 관련 의존 추가
- `@EnableWebMvc` 어노테이션 추가 시 `OptionalValidatorFactoryBean`이 글로벌 범위 Validator로 등록
- 스프링은 별도 설정이 없을 경우 `OptionalValidatorFactoryBean`을 글로벌 범위 Validator로 사용

<br>

- `@AssertTrue`: true인지 (null은 유효)
- `@AssertFalse`: false인지
- `@DecimalMax`: 지정 값보다 작은지
- `@DecimalMin`: 지정 값보다 큰지
- `@Max`: 지정 값보다 작거나 같은지
- `@Min`: 지정 값보다 크거나 같은지
- `@Digits`: 자릿수
- `@Size`: 길이, 크기
- `@Null`
- `@Notnull`
- `@Pattern`
- `@NotEmpty`: null이 아니고 길이가 0이 아님
- `@NotBlank`: null이 아니고 공백이 아닌 문자 1개 이상
- `@Positove`: 양수인지
- `@Negative`: 음수인지
- `@Email`
- `@Future` / `@FutureOrPresent`: 해당 시간이 미래 시간인지
- `@Past` / `@PastOrPresent`: 해당 시간이 과거 시간인지