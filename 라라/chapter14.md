# 14.날짜 값 변환, @PathVariable, 익셉션 처리

## 날짜 값 변환하기

```java
@DataTimeFormat(pattern = "yyyyMMddHH")
private LocalDateTime from;
```

- `@DataTimeFormat` 은 Request DTO 에서 문자열로 들어온 값을 `LocalDateTime` 타입으로 변환한다.
- “2022060215” → 2022년 6월 2일 15시 값을 갖느느 객체로 변환

## @PathVariable 을 이용한 경로 변수  처리

```java
@GetMapping("members/{id}")
public String detail(@PathVariable("id") Long memberId) {
    // ...
}
```

## 컨트롤러 익셉션 처리하기

```java
@ExceptionHandler(EmptyResultDataAccessException.class)
public ResponseEntity<ErrorResponse> handle() {
    return ResponseEntity.badRequest().body(new ErrorResponse("존재하지 않는 데이터 요청입니다."));
}
```

- 컨트롤러에서 발생한 익셉션을 직접 처리하고 싶다면 컨트롤러 내에 `@ExceptionHandler` 어노테이션을 적용한 메서드를 구현하면 된다.
- 해당 컨트롤러에서 발생한 익셉션만 처리한다.

## 공통 익셉션 처리

```java
@RestControllerAdvice
public class ControllerAdvice {

    @ExceptionHandler(RuntimeException.class)
    public ResponseEntity<ErrorResponse> handleUnhandledException() {
        return ResponseEntity.badRequest().body(new ErrorResponse("Unhandled Exception"));
    }
		
		// ...
}
```

- `@RestControllerAdvice` 를 사용하면 다수의 컨트롤러에서 동일한 익셉션 처리가 가능하다.