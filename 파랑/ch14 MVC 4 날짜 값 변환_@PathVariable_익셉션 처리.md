# Chapter 14 MVC 4: 날짜 값 변환, @PathVariable, 익셉션 처리

## @DateTimeFormat

```java
public class ListCommand { 
    @DateTimeFormat(pattern = "yyyyMMddHH") 
    private LocalDateTime time;
    
    ...
}
```

- `@DateTimeFormat` 이 적용되어 있으면 지정한 형식을 이용해서 문자열을 LocalDateTime 형식으로 반환한다.
- 누가 이것을 변환하는가?

  `WebDataBinder`가 값 변환에도 관여한다.

  스프링 MVC는 RequestMapping 적용 메서드와 DispatcherServlet 사이를 연결하기 위해 `RequestMappingHandlerAdapter` 객체를 사용한다. `RequestMappingHandlerAdapter` 객체는 RequestMapping 파라미터와 Command 객체 사이의 변환 처리를 위해 `WebDataBinder`를 사용한다.


## @PathVariable

경로의 일부가 고정되어 있지 않고 달라질 때 사용한다.

```java
// 컨트롤러
@GetMapping("/members/{id}")
public String detail(@PathVariable("id") Long memId, Model model) {
    ...
}
```

- `{id}` : 경로 변수, 경로 변수 이름과 파라미터 이름이 같으면 어노테이션의 괄호를 생략할 수 있다. → `@PathVariable Long id`

## 컨트롤러 익셉션 처리하기

### @ExceptionHandler

같은 컨트롤러에 `@ExceptionHandler` 를 적용한 메서드가 존재하면 그 메서드가 예외를 처리한다. 예외 객체에 대한 정보를 메서드의 파라미터로 전달받아 사용할 수 있다.

- 받을 수 있는 파라미터
    - HttpServletRequest, HttpServletResponse, HttpSession
    - Model
    - Exception

### @ControllerAdvice

다수의 컨트롤러에서 동일 타입의 예외를 동일하게 처리할 때 사용한다. `@ControllerAdvice` 적용 클래스가 동작하려면 해당 클래스를 스프링에 빈으로 등록해야 한다.

### 적용 순서

1. 같은 컨트롤러에 위치한 `@ExceptionHandler` 를 적용한 메서드
2. `@ControllerAdvice` 적용 클래스에 위치한 `@ExceptionHandler` 를 적용한 메서드
