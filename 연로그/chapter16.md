## Chapter 16 - JSON 응답과 요청 처리

### JSON

> [블로그 링크](https://yeonyeon.tistory.com/48) 로 대체

### Jackson vs Gson

#### Jackson

- 스프링 프레임워크에 내장
- 다양한 어노테이션 지원

#### Gson

- JSON 변환 간단
- deserialization을 위해 Java 엔티티에 접근하지 않아도 됨

### JSON으로 요청 / 응답하기

- @RequestBody
- @RestController
    - = @Controller + @Repository
    - return 한 객체를 알맞은 형식으로 변환해서 응답 데이터로 전송
    - 클래스 패스에 Jackson이 존재하면 JSON 형식의 문자열로 변환해서 응답
- @ResponseEntity

### @JsonIgnore

- 비밀번호 같은 민감한 데이터 숨길 때 이용

```java
public class User {
    private Long id;
    private String account;
    @JsonIgnore
    private String pasword;
}
```

### @JsonProperty

> [블로그 링크](https://yeonyeon.tistory.com/65) 로 대체

### @JsonFormat

```java
public class User {
    private Long id;
    private String account;
    @JsonFormat(shape = Shape.STRING)
//    @JsonFormat(pattern = "yyyyMMdd HHmmss) 식으로도 사용 가능
    private LocalDateTime registerDate;
}
```

### 기본 적용 설정

```java

@Configuration
@EnableWebMvc
public class MvcConfig implements WebMvcConfigurer {
    // ...

    @Override
    public void extendMessageConverters(List<HttpMessageConverter<?>> converters) {
        ObjectMapper objectMapper = new Jackson2ObjectMapperBuilder.json()
                .featuresToDisblae(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS)
                .build();

        converters.add(0, new MappingJackson2HttpMessageConverter(objectMapper));
    }
}
```

***

### 참고

- https://www.baeldung.com/jackson-vs-gson