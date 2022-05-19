# Chapter 4 - 의존 자동 주입

### @Autowired 어노테이션

- `@Autowired`를 이용한 다양한 주입 방식은 [블로그](https://yeonyeon.tistory.com/220) 에 기술
- @Autowired
    - 일치하는 빈이 없는 경우: UnsatisfiedDependencyException
    - 타입이 같은 빈이 여러개인 경우: UnsatisfiedDependencyException

### @Qualifier 어노테이션

- 자동 주입 가능한 빈이 여러개인 경우 자동 주입할 빈을 지정할 수 있음
- 어노테이션 생략 시 빈의 이름을 한정자로 지정

#### example

```java
public class UserService {

    private UserDao userDao;

    @Autowired
    @Qualifier("dao1")
    public void setUserDao(UserDao userDao) {
        this.userDao = userDao;
    }
}
```

```java

@Configuration
public class ApplicationContext {

    @Bean
    @Qualifier("dao1") // <- 해당 bean이 사용된다!
    public UserDao userDao1() {
        return new UserDaoImpl1();
    }

    @Bean
//    @Qualifier("dao2")
    public UserDao dao2() { // @Qualifier 어노테이션 생략 시 빈의 이름을 한정자로 지정
        return new UserDaoImpl2();
    }
}
```

### @Autowired 어노테이션 필수 여부 정하기

- 매칭되지 않는 bean 이 존재하면 Exception 발생
- 자동 주입할 대상이 필수가 아닌 경우는 어떻게 할까?

#### @Autowired(required = false)

- Spring 5 버전부터 지원
- 빈이 존재하지 않으면 자동 주입 대상에 할당 안함

```java
public class UserService {

    @Autowired(required = false)
    private UserDao userDao;
}
```

#### Optional

- Java 8부터 지원
- 빈이 존재하지 않으면 자동 주입 대상에 빈 Optional 할당

```java
public class UserService {

    @Autowired
    private Optional<UserDao> userDao;
}
```

#### @Nullable

- 빈이 존재하지 않으면 자동 주입 대상에 null 할당

```java
public class UserService {

    @Autowired
    @Nullable
    private UserDao userDao;
}
```

### 자동 주입 vs 수동 주입

 자동 주입과 수동 주입 코드가 섞이면 주입이 되지 않아 NullPointerException 발생하는 경우 원인 찾는게 오래 걸린다. 
일관되게 사용하는 것이 좋다. 
관리할 빈이 너무 많으면 설정 정보가 커지고 이를 관리하는 것이 매우 부담스러워진다. 
가능하면 스프링이 잘 지원해주는 자동 주입을 사용하자.
다만 수동 주입이 더 좋은 경우도 분명 존재한다. 
어플리케이션 기능은 크게 핵심 기능과 부가 기능으로 나뉜다.

- 핵심 기능: 해당 객체가 제공하는 고유한 기능 (ex: CustomerService - 구매 로직)
- 부가 기능: 핵심 기능을 보조하는 기능 (ex: 로그 추적, 트랜잭션 처리 등)

부가 기능은 핵심과 비교해 수가 적고 어플리케이션에 광범위하게 영향을 미친다. 
이런 부분에 대해서는 수동 주입을 통해 명확하게 들어내는 것이 좋을 때도 있다.
