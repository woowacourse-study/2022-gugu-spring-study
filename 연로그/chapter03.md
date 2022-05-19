# Chapter 3 - 스프링 DI

### 의존 주입; DI; Dependency Injection

- 여기서 의미하는 의존은 **객체 간의 의존**을 의미
- `의존`: 변경에 의해 영향받는 관계
- 한 클래스가 다른 클래스의 메서드를 실행할 때 이를 **의존**한다고 표현

#### example1 - 의존 객체 직접 생성하기

- 의존 객체 직접 생성
- 의존 객체인 `UserDao`를 변경하고 싶은 경우 `UserService`에서 직접 수정

```java
public class UserService {
    private UserDao userDao = new UserDao();
}
```

#### example2 - 의존 객체 주입받기

- 의존 객체 변경을 유연하게 하기 위해서는 생성자를 이용하는 것이 좋음

```java
public class UserService {
    private UserDao userDao;

    public UserService(UserDao userDao) {
        this.userDao = userDao;
    }
}
```

### 객체 조립기

- 객체를 생성하고 의존 대상이 되는 객체를 주입하는 역할
- Service 또는 Service 를 호출하는 곳에서 객체를 주입시키지 않아도 됨

#### example - 객체 조립기를 이용한 의존 주입

```java
public class Assembler {

    private UserDao userDao;
    private UserService userService;

    public Assembler() {
        userDao = new UserDao(); // userService의 의존 객체를 수정하고 싶다면 요 부분만 수정하면 된다.
        userService = new UserService(userDao);
    }

    public UserDao getUserDao() {
        return userDao;
    }

    public UserService getUserService() {
        return userService;
    }
}
```

### Spring & DI

- 스프링은 위의 조립기와 유사한 기능을 이미 제공함
- 객체 생성 및 생성 객체에 의존 주입

#### example - @Configuration과 @Bean 활용

```java
// 설정 클래스
@Configuration
public class ApplicationContext {

    @Bean
    public UserDao userDao() {
        return new UserDao();
    }

    @Bean
    public UserService userService() {
        return new UserService(userDao());
    }
}
```

#### 컨테이너 생성

- `ApplicationContext context = new AnnotationConfigApplicaionContext(ApplicaionContext.class)`
- 설정 파일 여러개도 가능 (ex: `new AnnotationConfigApplicaionContext(ApplicaionContext1.class, ApplicationContext2.class)`)

> 컨테이너를 직접 선언하지 않아도 자동으로 컨테이너가 생성되었던 것은 SpringApplication.run()에서 생성했기 때문이다.  
> 참고 자료: [Spring 공식 문서](https://docs.spring.io/spring-framework/docs/5.3.x/reference/html/core.html#beans-basics)

#### getBean()

- 컨테이너를 생성하면 getBean()을 통해 빈 객체를 가져올 수 있음
- BeanFactory 인터페이스에서 정의
- AbstractApplicationContext 클래스에서 구현
- `context.getBean("bean 이름", bean타입.class)`
    - 이름 불일치하는 경우: NoSuchBeanDefinitionException
    - 타입 불일치하는 경우: BeanNotOfRequiredTypeException
- `context.getBean(bean타입.class)`
    - 존재하지 않는 빈인 경우: NoSuchBeanDefinitionException
    - 타입 같은 빈이 여러개인 경우: NoUniqueBeanDefinitionException

### @Import 어노테이션

- 함께 사용할 설정 클래스 지정

#### example

```java

@Configuration
@Import(ApplicationContext2.class)
// @Import({ApplicationContext2.class, ApplicationContext2.class})
public class ApplicationContext {

    @Bean
    public UserDao userDao() {
        return new UserDao();
    }

    @Bean
    public UserService userService() {
        return new UserService(userDao());
    }
}
```
