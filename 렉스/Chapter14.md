# @DateTimeFormat
Spring은 DTO의 기본 타입으로의 변환을 기본적으로 처리해주지만 LocalDateTime은 추가적인 설정이 필요하다.
추가 설정은 아래와 같이 `@DateTimeFormat`의 pattern을 설정해줘야 한다.
```java
public class ListCommand {

	@DateTimeFormat(pattern = "yyyyMMddHH")
	private LocalDateTime from;
	@DateTimeFormat(pattern = "yyyyMMddHH")
	private LocalDateTime to;

	public LocalDateTime getFrom() {
		return from;
	}

	public LocalDateTime getTo() {
		return to;
	}
}
```
> `@DateTimeFormat`은 `java.tim.LocalDateTime`, `java.time.LocalDate`, `java.util.Date`, `java.util.Calendar` 타입을 지원한다.

# 컨트롤러 Request DTO의 변환 처리
스프링 MVC는 RequestMappingHandlerAdvice 객체를 통해 요청 매핑 어노테이션 적용 메서드와 DispatcherServlet 사이를 연결한다.
`RequestMappingHandlerAdvice`는 요청 파라미터와 커멘드 객체 사이의 변환 처리를 위해 `WebDataBinder`를 이용하는데, `WebDataBinder`는 값을 직접 변환하기 않고 `ConversionService`에게 역할을 위임하여 실질적인 변환을 하게 한다.
(`@EnableWebMvc`를 사용하면 `DefaultFormattingConversionService`가 `ConversionService`로 사용된다.)

# @ExceptionHandler, @ControllerAdvice
- 예외가 발생하는 컨트로러에 `@ExceptionHandler`를 적용한 메서드가 존재하면 해당 메서드가 예외를 처리한다.
- 컨트롤러 클래스에 `@ExceptionHandler`를 통한 예외 처리를 하면 해당 클래스의 예외들만 처리를 하게 되는데 이를 모든 컨트롤러 클래스에 적용을 하려면 `@ControllerAdvice`이 붙은 클래스를 생성하여 해당 클래스 내에 `@ExceptionHandler`를 생성해주면 된다.
- `@ControllerAdvice`와 특정 클래스 내의 `@ExceptionHandler`가 같은 예외를 처리한다면 클래스 내의 `@ExceptionHandler`가 우선적으로 동작한다.
