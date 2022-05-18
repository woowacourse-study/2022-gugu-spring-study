# 의존 자동 주입
```java
@Configuration
@Import(AppCtx2.class)
public class AppCtx1 {

    @Bean
    public MemberDao memberDao() {
        return new MemberDao();
    }

    @Bean
    public MemberPrinter memberPrinter() {
        return new MemberPrinter();
    }
}
```
앞의 챕터에서 배운바와 같이 이전까지는 DI를 `@Configuration`이 붙은 설정 클래스에서 직접 생성자나 setter를 통해 주입을 해주었다.
하지만 위와 같이 직접 의존 빈을 주입하지 않고 스프링이 자동으로 주입해주는 **자동 주입 기능** 도 존재한다. 

- 자동 주입 기능은 스프링 3,4 버전에서는 호불호가 갈렸으나 스프링 부트가 나오면서 의존 자동 주입을 사용하는 추세로 바뀌었다. (거의 기본으로 사용한다.)
- 자동 주입은 `@Autowired`를 사용하여 한다.

## Autowired
- `@Autowired`은 스프링 빈에 의존하는 다른 빈을 자동으로 주입하고 싶을 때 사용한다.
- 해당 어노테이션을 붙여주면 해당 **타입** 의 빈을 찾아서 해당 필드에 할당해준다.
  - 의존하는 빈들을 클래스 코드에서 `@Autowired`를 사용하여 필드에 추가를 하면, `@Bean`메서드를 통해 빈을 생성할 때, 된 빈의 의존 주입을 위한 코드를 작성하지 않아도 된다.
- `@Autowired`는 필드와 메서드 둘 다 붙일 수 있다.
  - 필드나 세터 메서드에 붙여주면 스프링은 **타입이 일치하는 빈** 객체를 찾아서 주입해준다.
- 생성자를 통해 자동 주입 설정을 하였을 경우, configuration에서 빈을 등록할 때 setter로 주입하는 빈을 변경하여도 자동 주입 설정이 된 빈이 할당된다. (자동 주입 설정을 하였으면 setter를 하여도 적용되지 않는다.)
- `@Autowired(required = false)`와 같이 설정할 경우 주입할 빈이 존재하지 않아도 예외를 발생시키지 않는다.
  - 스프링 5부터는 required 속성을 false로 하는 대신, 의존 주입 대상에 Optional 을 사용해도 된다.
    - 자동 주입 대상이 Optional인 경우, 일치하는 빈이 존재하지 않으면 값이 없는 Optional을 인자로 전달하고 일치하는 빈이 존재하면 해당 빈을 값으로 갖는 Optional을 전달한다. (예외 발생 x)
```java
    @Autowired
    public void setDateTimeFormatter(Optional<DateTimeFormatter> formatterOpt) {
        if (formatterOpt.isPresent()) {
            this.dateTimeFormatter = formatterOpt.get();
        } else {
            this.dateTimeFormatter = null;
        }
    }
```
- 주입 필수 여부를 설정하는 방법에는 `@Autowired(required = false)`와 `Optional`말고 `@Nullable`을 사용하는 방법도 있다.
  - `@Autowired(required = false)`와 다른 점은 `@Autowired(required = false)`는 주입할 빈이 없으면 setter메서드가 호출되지 않아 값 할당 자체를 하지 않다. 반면에 `@Nullable`은 주입할 빈이 존재하지 않으면 인자로 null을 전달하고 메서드가 호출된다.
    - 기본 생성자에서 자동 주입 대상이 되는 필드를 초기화 할 때는 해당 내용을 유의해야한다.
```java
    @Autowired(required = false)
    public void setDateTimeFormatter(@Nullable DateTimeFormatter dateTimeFormatter) {
        this.dateTimeFormatter = dateTimeFormatter;
    }
```
## Autowired 관련 예외
- 자동 주입할 빈이 존재하지 않을 경우 아래와 같은 예외가 발생한다. (NoSuchBeanDefinitionException)
```
Caused by: org.springframework.beans.factory.NoSuchBeanDefinitionException: No qualifying bean of type 'ch4.dao.MemberDao' available: expected at least 1 bean which qualifies as autowire candidate. Dependency annotations: {@org.springframework.beans.factory.annotation.Autowired(required=true)}
```
- 자동 주입할 빈이 같은 타입으로 2개가 존재하는 경우 아래와 같은 예외가 발생한다. (NoUniqueBeanDefinitionException)
  - **빈으로 등록된 클래스의 자식 클래스도 빈으로 등록할 경우도 주입 가능한 빈을 2개라고 판단해 아래와 같은 에러가 발생한다.**
    - 상속 관계에 의해 동일 타입의 빈이 2개라 자동 주입을 못하는 문제는 `@Qualifier`또는 클래스의 생성자, 또는 setter에서 외부로부터 받는 타입을 하위타입으로 구체적으로 지정해주면 해결할 수 있다.
```
Caused by: org.springframework.beans.factory.NoUniqueBeanDefinitionException: No qualifying bean of type 'ch4.printer.MemberPrinter' available: expected single matching bean but found 2: memberPrinter,memberPrinter2
```

## @Qualifier
- 해당 어노테이션은 자동 주입 가능한 빈이 2개 이상일 경우 자동 주입할 빈을 한정할 수 있다.
- `@Qualifier`는 두 위치에 붙이며 사용한다.
  - `@Bean`을 붙인 빈 설정 메서드 위와 `@Autowired`를 하는 메서드나 필드 위에 아래와 같이 지정을 하며 사용한다.

```java
@Configuration
@Import(AppCtx2.class)
public class AppCtx1 {
    ...
    @Bean
    @Qualifier("printer")
    public MemberPrinter memberPrinter() {
        return new MemberPrinter();
    }

    @Bean
    public MemberPrinter memberPrinter2() {
        return new MemberPrinter();
    }
}

public class MemberInfoPrinter { 
    private MemberDao memberDao;
    
    @Autowired
    @Qualifier("printer")
    private MemberPrinter printer;
  ...
}
```

- 일반적으로 `@Qualifier`이 없으면 `@Bean`이 붙은 메서드 이름이 해당 빈의 한정자(빈의 이름)가 된다. (`@Qualifier`가 빈의 한정자(이름)을 정해준다고 보면 된다.)
  - `@Autowired`도 `@Qualifier`이 없다면 변수의 이름을 한정자로 사용하여, 같은 타입의 빈이 2개 이상일 경우 자신의 변수 이름과 같은 빈을 사용하게 된다. (p.117)
