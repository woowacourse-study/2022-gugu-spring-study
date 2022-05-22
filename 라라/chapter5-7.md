# 5장. 컴포넌트 스캔

### `@Component` 어노테이션으로 **스캔 대상**을 지정

- 클래스에 `@Component` 어노테이션을 붙여주면 해당 클래스를 스프링에서 검색할 수 있는 빈으로 등록된다.
- `@Component(”이름”)` 형식으로 빈 이름을 등록할 수 있다.
- 만약 지정하지 않으면 클래스 첫 글자를 소문자로 바꾸고 빈이름으로 등록된다. (MemberDao → memberdao)

### `@ComponentScan` 어노테이션으로 **스캔 대상**을 지정

- `@ComponentScan(basePackages = {”spring”})`
- `basePackages` : 스캔 대상 패키지 목록

### 스캔 대상에서 제외하거나 포함하기

- `excludeFilters` 속성을 사용하여 특정 대상을 자동 등록 대상에서 제외
- @ComponentScan(basePackages = {”spring”}, excludeFilters = **@Filter(type = FilterType.REGEX, pattern = “spring\\..*Dao”)**)
    - `FilterType.REGEX`  : 정규 표현식 사용
- @ComponentScan(basePackages = {”spring”}, excludeFilters = **@Filter(type = FilterType.ASPECTJ, pattern = “spring.*Dao”)**)
    - `FilterType.ASPECTJ`  : AspectJ 패턴 사용
- @ComponentScan(basePackages = {”spring”}, excludeFilters = **@Filter(type = FilterType.ANNOTATION, classes = {NoProduct.class, ManualBean.class})**)
    - `FilterType.ANNOTATION` **:** classes 에 작성한 어노테이션을 붙인 클래스를 컴포넌트 스캔 대상에서 제외

### 기본 스캔 대상

- @Component
- @Controller
- @Service
- @Repository
- @Aspect
- @Configuration

@Controller, @Repository 는 컴포넌트 스캔 대상이자 스프링 프레임워크에서 특별한 기능과 연관

- @Controller : 웹 MVC 와 연관
- @Repository : DB 연동과 연관

### 컴포넌트 스캔 충돌 처리

**빈 이름 충돌**

- @ComponentScan(basePackages = {”spring”, “spring2”})
- spring, spring2 패키지에 동일한 이름의 클래스가 존재하고 두 클래스 모두 `@Component` 를 붙였을 경우 `익셉션 발생`
- 둘 중 하나에 명시적으로 빈 이름 지정

**수동 등록한 빈과 충돌**

- 자동 등록된 빈 이름과 수동 등록한 빈 이름이 같은 경우 : **수동 등록한 빈이 우선**
- 설정 클래스에서 수동 등록한 이름과 다를 경우 같은 타입의 빈이 두개가 생성되므로 자동 주입하는 코드는 `@Qualifier` 로 알맞은 빈을 선택해야 함

# 6장. 빈 라이프사이클과 범위

### **스프링 컨테이너**

- 초기화 ~ 종료 라이프 사이클을 가짐

```java
// 1. 컨테이너 초기화
AnnotationConfigApplicationContext ctx = new AnnotationConfigApplicationContext(ApplicationContext.class);

// 2. 컨테이너에서 빈 객체를 구해서 사용
Greeter greeter = ctx.getBean("greeter", Greeter.class);
String msg = g.greet("스프링");

// 3. 컨테이너 종료
ctx.close();
```

컨테이너 초기화하고 종료할 때 다음의 작업도 함께 수행

- 컨테이너 초기화 : 빈 객체의 생성, 의존 주입, 초기화
- 컨테이너 종료 : 빈 객체의 소멸

### 스프링 빈 객체의 라이프 사이클

객체 생성 → 의존 설정 → 초기화 → 소멸

- 스프링 컨테이너를 초기화 할 때 스프링 컨테이너는 가장 먼저 빈 객체를 생성하고 의존을 설정함
- 의존 자동 주입을 통한 의존 설정이 이 시점에 수행됨
- 모든 의존 설정이 완료되면 빈 객체의 초기화를 수행함
- 빈 객체를 초기화하기 위해 스프링은 빈 객체의 지정된 메서드 호출
- 스프링 컨테이너를 종료하면 스프링 컨테이너는 빈 객체의 소멸을 처리함 → 지정한 메서드 호출

### 빈 객체의 초기화와 소멸 : 스프링 인터페이스

스프링 컨테이너는 빈 객체의 초기화와 소멸을 위해 지정된 메서드를 호출

해당 메서드는 인터페이스에 정의되어 있음

- `org.springframework.beans.factory.InitializingBean`
    
    ```java
    public interface InitializingBean {
    		// 초기화 과정에서 해당 메서드 호출
    		void afterPropertiesSet() throws Exception;
    }
    ```
    
- `org.springframework.beans.factory.DisposableBean`
    
    ```java
    public interface DisposableBean {
    		// 소멸 과정에서 해당 메서드 호출 (ctx.close() 호출시)
    		void destroy() throws Exception;
    }
    ```
    

**초기화 / 소멸 필요 과정**

1. DB 커넥션 풀
    1. 빈 객체는 초기화 과정에서 DB 연결 생성
    2. 연결 유지
    3. 빈 객체 소멸시 사용중인 DB 연결 끊음
2. 채팅 클라이언트
    1. 시작 : 서버와 연결 (초기화)
    2. 종료 : 연결 끊음 (소멸)
    

### 빈 객체의 초기화와 소멸 : 커스텀 메서드

- 모든 클래스가 `InitializingBean`, `DisposableBean` 인터페이스를 상속받을 수 있는 것은 아님
    - 직접 구현한 클래스가 아닌 외부에서 제공받은 클래스를 스프링 빈 객체로 설정하고 싶을 때
- 스프링 설정에서 직접 메서드를 지정할 수 있음
    - `@Bean` 태그에서 `initMethod` 속성과 `destroyMethod` 속성을 사용해서 초기화 메서드와 소멸 메서드의 이름을 지정하면 됨
    - **@Bean(initMethod = “connect”, destoryMethod = “close”)**
    

### 빈 객체의 생성과 관리의 범위

- 프로토타입 범위의 빈은 싱글톤이 아닌 빈 객체를 구할 때마다 **매번 새로운 객체를 생성**
    - `@Scope(”prototype”)`
    - 완전한 라이프 사이클을 갖지 않고 생성, 초기화만 가짐
    - 자동으로 소멸되지 않기 때문에 소멸 코드를 직접 작성해야함
- 싱글톤 범위로 명시적으로 지정하는 방법
    - `@Scope(”singleton”)`

# 7장. AOP 프로그래밍

### 프록시란?

→ 핵심 기능은 다른 객체에게 위임하고 부가적인 기능을 제공하는 객체

### **AOP란?**

- Spring의 핵심 개념중 하나인 DI가 애플리케이션 모듈들 간의 결합도를 낮춰준다면, AOP는 **애플리케이션 전체에 걸쳐 사용되는 기능을 재사용**하도록 지원하는 것
- AOP (Aspect-Oriented Programming) 란 단어를 번역하면 **관점(관심) 지향 프로그래밍**
