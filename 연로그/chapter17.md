## Chapter 17 - 프로필과 프로퍼티 파일

### 프로필

- 개발 목적 설정과 서비스 목적 설정을 구분하기 위해 스프링에서 지원하는 기능
- 논리적인 이름으로서 설정 집합에 프로필 지정 가능

```java

@Configuration
@Profile("dev")
public class DevConfig {

    @Bean
    public DataSource dataSource() {
        DataSource dataSource = new DataSource();
        dataSource.setDrivrClassName("com.mysql.jdbc.Driver");
        dataSource.setUrl("jdbc:mysql://localhost/spring5");
        dataSource.setUsername("yeonlog");
        dataSource.setPassword("12343");
        dataSource.setInitialSize(2);
        return dataSource;
    }
}
```

아래 명령어를 통해 실행하면 'dev' 프로필이 활성화

```
java -Dspring.profiles.active=dev main.Main
```

#### 프로퍼티 파일 이용

아래와 같이 프로퍼티 파일을 등록해두면 `@Value`를 통해 값을 불러올 수 있다.

```java
@Configuration
public class PropertyConfig {
    @Bean
    public static PropertySourcesPlaceholderConfigurer properties() {
        PropertySourcesPlaceholderConfigurer configurer = new PropertySourcesPlaceholderConfigurer();
        configurer.setLocations(new ClassPathResource("db.properties"));
        return configurer;
    }
}
```

### 궁금한 점
- application.properties는 어디서 기본적으로 등록되는걸까