# Chapter16
## JSON (JavaScript Object Notation)
- 간단한 형식을 갖는 문자열로 데이터 교환에 주로 사용한다.
- 중괄호를 사용해 객체(이름:값)를 포함한다.
- 값에는 문자열, 숫자, Boolean, null, 배열, 객체 들이 올 수 있다.
  - 문자열은 큰따옴표나 작은따옴표로 묶어줘야한다.
  - `\"`(큰따옴표), `\n`, `\r` 등의 특수문자 표현도 가능하다.

## Jackson
- Jackson은 자바 객체와 JSON 형식 문자열 간 변환을 처리하는 라이브러리이다.
- `@RestController`를 붙이고 Jackon 의존성을 추가해줬다면 반환 객체를 JSON타입으로 변환해서 응답하게 된다.
  - `@RestController`는 Spring 4버전에 나왔다.
- Jackson의 `@JsonIgnore`를 변수 위에 사용하면 컨트롤러에서 해당 객체를 반환할 떄 Json타입으로 변환하는 과저에서 제외시킨다.
- `@JsonFormat`을 사용하면 날짜 값의 출력 형식을 바꿀 수 있다.
  - 책에서는 날짜를 예시로 들었다
  - `@JsonFormat(shape = Shape.STRING)`, `@JsonFormat(pattern = yyyyMMddHHmss)"`와 같이 출력 형식을 변경해 줄 수도 있다.

### `@JsonFormat`의 글로벌 설정
- `@JsonFormat`를 모든 대상에 직접 붙이지 않고 모든 대상에 적용을 하려면 Spring MVC 설정을 변경해야 한다.
- Spring MVC는 자바 객체를 HTTP Response로 반환할 때 HttpMessageConverter를 사용하여 Json으로 반환할 때 사용하는 MappingJackson2HttpMesssageConverter를 새롭게 등록하면, 원하는 형식으로 변경할 수 있다.
```java
@Configuration
@EnableWebMvc
public class MvcConfig implements WebMvcConfigurer {
    @Override
    public void extendMessageConverters(List<HttpMessageConverter<?>> converters) {
        //DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
        ObjectMapper objectMapper = Jackson2ObjectMapperBuilder
                .json()
                .featuresToDisable(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS)
                //.serializerByType(LocalDateTime.class, new LocalDateTimeSerializer(formatter))
                //.deserializerByType(LocalDateTime.class, new LocalDateTimeDeserializer(formatter))
                //.simpleDateFormat("yyyy-MM-dd HH:mm:ss")
                .build();
        converters.add(0, new MappingJackson2HttpMessageConverter(objectMapper));
    }
}
```
- `extendMessageConverters()`는 HttpMessageConverter를 추가로 설저할 때 사용한다.
- `@EnableWebMvc`이 있으면 HttpMessageConverter들을 미리 등록한다.
- 미리 등록된 HttpMessageConverter에는 Jackson을 이용하는 것도 있어서 위의 코드에서 새로운 설정을 추가할 때는 0번 인덱스에 추가했다.
- Json으로 값을 변환할 떄는 `ObjectMapper`를 이용한다.
  - `Jackson2ObjectMapperBuilder`는 스프링이 제공하는 클래스로 `ObjectMapper`를 보다 쉽게 생성한다.

> WebConfig에 해당 설정을 해주더라도 `@JsonFormat`을 변수 위에 사용하면 `@JsonFormat`이 우선시된다.

## @RequestBody
스프링 MVC가 JSON형식으로 전송된 데이터를 올바르게 처리하려면 요청데이터 타입이 `application/json`으로 설정되어 있어야 한다.

## ResponseEntity
요청에 대한 결과가 정상/비정상인 경우 모두 JSON 응답을 전송하는 방법은 ResponseEntity를 사용하면 된다.
```java
	@GetMapping("/api/members/{id}")
	public ResponseEntity<Object> member(@PathVariable Long id) {
		Member member = memberDao.selectById(id);
		if (member == null) {
			return ResponseEntity
					.status(HttpStatus.NOT_FOUND)
					.body(new ErrorResponse("no member"));
			// return ResponseEntity.notFound().build();
		}
		return ResponseEntity.ok(member);
	}
```

## @ExceptionHandler, @RestControllerAdvice
중복된 예외 처리의 경우 `@ExceptionHandler`, `@RestControllerAdvice`를 통해 해결할 수 있다.

## @Valid 의 에러 처리하기
`@RequestBody` 에서 `@Valid` 어노테이션을 통한 검증을 실패하였을 때, `Error` 파라미터가 존재하지 않으면 `MethodArgumentNotValidException`이 발생한다.
이를 ControllerAdvice에서 잡으려면 아래와 같이 해줘야한다.

```java
@RestControllerAdvice("controller")
public class ApiExceptionAdvice {

	@ExceptionHandler(MethodArgumentNotValidException.class)
	public ResponseEntity<ErrorResponse> handleBindException(MethodArgumentNotValidException ex) {
		String errorCodes = ex.getBindingResult().getAllErrors()
				.stream()
				.map(error -> error.getCodes()[0])
				.collect(Collectors.joining(","));
		return ResponseEntity
				.status(HttpStatus.BAD_REQUEST)
				.body(new ErrorResponse("errorCodes = " + errorCodes));
	}

}
```
