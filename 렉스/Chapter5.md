# 컴포넌트 스캔
- 컴포넌트 스캔은 스프링이 직접 클래스를 검색해서 빈으로 등록해주는 기능이다.
  - 설정 클래스에 빈으로 등록하지 않아도 원하는 클래스를 빈으로 등록할 수 있다.
- 스프링이 검색하여 빈으로 등록할 수 있게 하려면 해당 클래스에 `@Component`를 붙여줘야 한다.
  - `@Component`를 통한 빈의 이름은 기본적으로 클래스 이름의 첫 글자를 소문자로 바꾼 이름을 사용하며 다른 이름을 설정하고 싶을 경우 `@Component("newName"")`과 같이 value값을 설정해줘야 한다.
- `@Component`가 붙은 클래스를 빈으로 등록하려면 설정 클래스에서 `@ComponentScan`을 설정해줘야한다.
  - `@ComponentScan`를 설정해주면 `basePackages`로 설정한 패키지의 하위 클래스 중 `@Component`가 붙은 클래스를 검색하고 생성하여 빈으로 등록해준다.
    
  
## @ComponentScan 설정값
### basePackages
- `basePackages`로 설정한 패키지의 하위 클래스 중 `@Component`가 붙은 클래스를 **자동으로** 검색하고 생성하여 빈으로 등록해준다.
- ex) `@ComponentScan(basePackages = {"ch5"})`

### excludeFilters
- 스캔할 때 설정을 한 특정 대상을 자동 등록 대상에서 제외해준다.
- `type = FilterType.REGEX` : 정규 표현식을 통해 제외 대상을 지정한다.
  - ex) `@Filter(type=FilterType.REGEX, pattern="spring\\..*Dao")` <- spring.으로 시작하고 Dao로 끝나는 클래스는 컴포넌트 스캔에서 제외한다.
- `type = FilterType.ASPECTJ` : AspectJ 패턴을 사용해 제외 대상을 지정한다.
  - AspectJ를 사용하려면 dependency에 `aspectjweaver`모듈을 추가해줘야한다.
  - ex) `@Filter(type = FilterType.ASPECTJ, pattern="spring*Dao)`
- `type = FilterType.ANNOTATION` : 설정된 annotation이 붙은 컴포넌트를 스캔 대상에서 제외한다.
  - ex) `@Filter(type = FilterType.ANNOTATION, classes = {NoProduct.class, NanualBean.class})` <- `@NoProduct`, `@NanualBean`이 붙은 클래스를 컴포넌트 스캔 대상에서 제외시킨다.
- `type = FilterType.ASSIGNABLE_TYPE` : 특정 타입과 그 하위 타입을 제외 대상으로 지정한다.
  - ex) `@Filter(type = Filter.ASSIGNABLE_TYPE, classes = MemberDao.class)` <- MemberDao 타입과 그 하위 타입을 컴포넌트 스캔에서 제외한다.
- 설정 필터가 두개 이상이라면 아래와 같이 지정하면 된다.
```java
 @ComponentScan(basePackages = {"ch5"},
        excludeFilters = {
                @ComponentScan.Filter(type = FilterType.ANNOTATION, classes = ManualBean.class),
                @ComponentScan.Filter(type = FilterType.REGEX, pattern = "spring\\..*")
        })
```

## 컴포넌트 스캔의 기본 대상
- @Component
- @Controller
- @Service
- @Repository
- @Configuration
- @Aspect

`@Aspect`를 제외한 다른 어노테이션들은 모두 `@Component`를 포함하는 어노테이션이다.

## 빈 충돌 처리
- 컴포넌트 스캔 과정에서 서로 다른 타입인데 같은 빈 이름을 사용하여 중복된 빈 이름 충돌이 발생한다면 둘 중 하나에 명시적인 빈 이름을 지정해서 충돌을 피해야한다.
- 자동으로 등록되는 빈과 수동으로 등록되는 빈의 이름이 같다면, 수동으로 등록한 빈이 우선적으로 등록된다. (자동 등록 빈은 생성되지 않는다.)
  - 같은 타입이지만 다른 이름의 빈을 생성한다면 자동 주입 코드(Autowired)는 `@Qualifier`를 통해 알맞는 빈을 선택해야 해야한다.
