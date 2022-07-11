# Chapter 16 JSON 응답과 요청 처리

### Jackson

자바 객체와 JSON 형식 문자열 간 변환을 처리하는 라이브러리

### @RestController

- JSON 형식으로 데이터를 응답할 수 있다.
- @ResponseBody + @Controller

### @JsonIgnore

응답에서 제외할 대상에 붙인다.

## 날짜 형식 변환 방법

### 1. @JsonFormat

- Jackson에서 날짜나 시간 값을 특정한 형식으로 표현한다.
- 원하는 형식으로 변환하고 싶다면 pattern 속성을 사용한다.

```java
public class Member {
    @JsonFormat(shape = Shape.STRING)
    private LocalDateTime registerDateTime;
    ...
}
```

### 2. 기본 적용 설정 (응답으로 반환시)

- 날짜 타입에 해당하는 모든 대상에 동일한 변환 규칙을 적용한다.
- 스프링 MVC는 자바 객체를 HTTP 응답으로 반환할 때 `HttpMessageConverter` 를 사용한다. JSON으로 변환할 때 사용하는 `MappingJackson2HttpMessageConverter`를 새롭게 등록하여 날짜 형식을 설정한다.

```java
@Configuration
@EnableWebMvc
public class MvcConfig implements WebMvcConfigurer {
    ... 생략
    @Override
    public void extendMessageConverters(List<HttpMessageConverter<?>> converters) {
        ObjectMapper objectMapper = Jackson2ObjectMapperBuilder
                .json()
                .featuresToDisable(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS)
                .build();
        converters.add(0, new MappingJackson2HttpMessageConverter(objectMapper));
    }
}
```

- extendMessageConverters()
    - WebMvcConfigurer 인터페이스에 정의된 메서드로 HttpMessageConverter를 추가로 설정할 때 사용
    - 등록된 HttpMessageConverter 목록을 파라미터로 받는다.
- ObjectMapper
    - Jackson이 날짜 형식 출력할 때 유닉스 타임 스탬프로 출력하는 기능 비활성화.
- HttpMessageConverter
    - 새로 생성한 HttpMessageConverter는 목록의 제일 앞에 위치시켜야 가장 먼저 적용된다.
    - 이를 위해 새로운 HttpMessageConverter를 0번 인덱스에 추가했다.

---

- simpleDateFormat()
    - 모든 `java.util.Date` 타입의 값을 원하는 형식으로 출력하도록 설정하고 싶을 때
    - 위 예시의 featuresToDisable 대신 사용

    ```java
    .simpleDateFormat("yyyyMMddHHmmss")
    ```

- serializerByType()
    - 원하는 패턴을 설정하고 싶을 때
    - 위 예시의 featuresToDisable 대신 사용

    ```java
    .serializerByType(LocalDateTime.class, new LocalDateTimeSerializer(formatter))
    ```

- deserializerByType() (요청으로 받을 때)
    - JSON 형식의 데이터를 `LocalDateTime` 형식으로 변환

    ```java
    .deserializerByType(LocalDateTime.class, new LocalDateTimeDeserializer(formatter))
    ```
