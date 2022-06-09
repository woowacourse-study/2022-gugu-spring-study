# Chapter17
## 프로필
- DB와 같이 개발용, 배포용을 다른 빈을 사용하는 경우 스프링의 **프로필** 기능을 사용하면 쉽게 전환할 수 있다.
- 프로필을 사용하면 프로젝트 실행 전 설정한 프로필에 따라 스프링 컨테이너를 초기화하여 쉽게 전환할 수 있다.
- `@Configuration`을 사용한 설정에서 프로필을 지정하려면 `@Profile`을 지정하면 된다.
```java
@Configuration
@Profile("dev")
public class DsDevConfig {

	@Bean(destroyMethod = "close")
	public DataSource dataSource() {
		DataSource ds = new DataSource();
		ds.setDriverClassName("com.mysql.jdbc.Driver");
		ds.setUrl("jdbc:mysql://localhost/spring5fs?characterEncoding=utf8");
		ds.setUsername("spring5");
		ds.setPassword("spring5");
		ds.setInitialSize(2);
		ds.setMaxActive(10);
		ds.setTestWhileIdle(true);
		ds.setMinEvictableIdleTimeMillis(60000 * 3);
		ds.setTimeBetweenEvictionRunsMillis(10 * 1000);
		return ds;
	}
}
```
- 특정 프로필을 설정하려면 아래와 같이 `setActiveProfiles()`를 통해 프로필을 선택해야 한다.
  - 두 개 이상의 프로필을 설정하고 싶은 경우 `setActiveProfiles("dev", "mysql")`과 같이 여러 프로필 이름을 전달해주면 된다.
> 🚨 **주의할 점**
> - 설정 정보를 전달하기 전에 `setActiveProfiles()`를 통해 사용할 프로필을 정해야한다. 그러지 않을 경우, 하위의 설정 정보가 프로필에 대한 설정으로 적용되지 않아 예외가 발생할 수 있다.

```java
public class MainProfile {

	public static void main(String[] args) {
		AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext();
		context.getEnvironment().setActiveProfiles("dev");
		context.register(MemberConfig.class, DsDevConfig.class, DsRealConfig.class);
		//context.register(MemberConfigWithProfile.class);
		context.refresh();
		
		MemberDao dao = context.getBean(MemberDao.class);
		List<Member> members = dao.selectAll();
		members.forEach(m -> System.out.println(m.getEmail()));
		
		context.close();
	}
}
```

- 프로필은 `@Configuration`이 붙은 클래스 내에 `@Configuration`이 붙은 중첩 클래스를 만들어 한 곳에 모아둘 수 있다. 하지만 해당 방법의 경우 중첩 클래스들은 static 클래스로 만들어줘야 한다.
- 프로필은 2개 이상의 이름을 가질 수 있다. 해당 설정은 `@Profile("real,test")`와 같이 설정하면 된다.
- 특정 프로필이 사용되지 않을 때, 기본으로 사용될 default 프로필을 설정하려면 `@Profile("!real")`와 같이 설정하면 된다.
  - 다음 의미는 real 프로필이 활성화되지 않았을 때 사용하겠다는 의미이다.

## 프로퍼티 파일을 이용한 프로퍼티 설정
프로퍼티 파일을 사용하려면 두 가지 설정이 필요하다
1. `PropertySourcesPlaceholderConfigurer` 빈을 `static`으로 등록
   - 해당 빈은 특수한 목적의 빈이기 때문에 정적 메서드로 지정하지 않으면 원하는 방식으로 동작하지 않는다.
2. `@Value` 어노테이션을 통해 프로퍼티 값 사용

```java
@Configuration
public class PropertyConfig {

	@Bean
	public static PropertySourcesPlaceholderConfigurer properties() {
		PropertySourcesPlaceholderConfigurer configurer = new PropertySourcesPlaceholderConfigurer();
		configurer.setLocations(
				new ClassPathResource("db.properties"),
				new ClassPathResource("info.properties"));
		return configurer;
	}
}
```

위와 같이 `PropertySourcesPlaceholderConfigurer`를 빈으로 등록한 경우 아래의 `@Value("${db.driver}")`와 같은 @Value어노테이션을 통해 정의되어있는 프로퍼티 값으로 치환하며 사용할 수 있다.
```java
@Configuration
public class DsConfigWithProp {
    @Value("${db.driver}")
    private String driver;
    @Value("${db.url}")
    private String jdbcUrl;
    @Value("${db.user}")
    private String user;
    @Value("${db.password}")
    private String password;

	@Bean(destroyMethod = "close")
	public DataSource dataSource() {
		DataSource ds = new DataSource();
		ds.setDriverClassName(driver);
		ds.setUrl(jdbcUrl);
		ds.setUsername(user);
		ds.setPassword(password);
		ds.setInitialSize(2);
		ds.setMaxActive(10);
		ds.setTestWhileIdle(true);
		ds.setMinEvictableIdleTimeMillis(60000 * 3);
		ds.setTimeBetweenEvictionRunsMillis(10 * 1000);
		return ds;
	}
}
```
