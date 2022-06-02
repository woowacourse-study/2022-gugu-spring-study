# MVC2 : 메시지, 커맨드 객체 검증

## MessageSource

> Strategy interface for resolving messages, with support for the parameterization and internationalization of such messages.

`MessageSource`는 메시지 국제화를 지원하는 인터페이스이다. 스프링은 Locale(지역)에 상관없이 일관되게 문자열을 관리할 수 있도록 지원하고 있다. MessageSource의 getMessage() 메서드를 이용해 특정 Locale에 대응하는 메시지를 가져올 수 있다.

```java
String getMessage(String code, @Nullable Object[] args, @Nullable String defaultMessage, Locale locale);
```
> code - 메시지 프로퍼티의 key값  
Object[] args - 메시지 내 인덱스 기반 변수 전달 ex. "{0}"  
defaultMessage - code에 대응하는 key값이 없을 때 출력하는 메시지  
Locale locale - 로케일
  

## 구현체

1. **ResourceBundleMessageSource**: built on top of the standard ResourceBundle, sharing its limitations.
2. **ReloadableResourceBundleMessageSource**: highly configurable, in particular with respect to reloading message definitions.

## 사용방법

1. 메시지 파일 작성
- src/main/resources 디렉토리에 [파일이름]_[언어코드]_[국가코드].properties 형식으로 메시지 파일 추가

> 예시  
message.properties
기본 메시지, 시스템의 언어 및 지역에 맞는 프로퍼티 파일 존재하지 않을 경우 사용  
message_en.properties : 영어 메시지  
message_ko.properties : 한글 메시지  
message_en_UK.properties : 영국을 위한 영어 메시지  
   
  - key-value 형식의 값 입력

> key=value, {index}  
✔ {index} : argument가 바인딩 되는 부분  
ex) greeting=Hello, so good {0}  
ex) greeting=안녕, 진짜 반가워 {0}  

2. **MessageSource** 빈 등록

Spring Boot를 사용할 경우 `ResourceBundleMessageSource`가 빈으로 자동등록되기에 별도로 등록할 필요가 없다.

빈을 등록할 경우 빈의 아이디를 `messageSource`로 지정해야만 한다. 다른 이름을 사용할 경우 인식하지 못한다.

```java
@Configuration
@EnableWebMvc
public class MvcConfig implements WebMvcConfigurer {
	@Bean
	public MessageSource messageSource() {

		var messageSource = new ResourceBundleMessageSource();
		messageSource.setBasename("messages");  //메시지 리소스 번들 지정
		messageSource.setBasename("messages.label");  //src/main/resources/messages 패키지 내 label.properties 파일 읽기
		
		messageSource.setDefaultEncoding("UTF-8");  //문자 집합 설정 - 한글 깨짐 방지
		
		return messageSource;
	}
}
```
`setBasename` : 사용할 메시지 프로퍼티 목록을 전달한다.
  
3. MessageSource를 통해 각 Locale에 맞는 메시지를 출력한다.

```java
@Component
public class AppRunner implements ApplicationRunner {
 
    @Autowired
    MessageSource messageSource;
 
    @Override
    public void run(ApplicationArguments args) throws Exception {

        System.out.println(messageSource.getMessage("greeting", new String[]{"java"}, Locale.US));
        System.out.println(messageSource.getMessage("greeting", new String[]{"java"}, Locale.KOREA));
    }
}
```

실행 결과
> 안녕, 진짜 반가워 java  
Hello, so good java