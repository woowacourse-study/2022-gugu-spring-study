# Chapter 14 - 날짜 값 변환, @PathVariable, 익셉션

### 날짜 값 변환

- `@DateTimeFormat` 활용
- ex: `@DateTimeFormat(pattern = "yyyyMMddHH")`
- `WebDataBinder`가 값 변환에 관여
  - `ConversionService`에게 변환 역할 위임
  - `@EnableWebMvc`를 적용했다면 `DefaultFormattingConversionService` 적용

### @PathVariable

- URL의 일부 데이터를 사용 가능

```java
@GetMapping("/{productId}")
public ResponseEntity<Product> product(@PathVariable final Long productId) {
    return ResponseEntity.ok(productService.findProductById(productId));
}
```

### 익셉션 처리

> [블로그 링크](https://yeonyeon.tistory.com/218) 로 대체