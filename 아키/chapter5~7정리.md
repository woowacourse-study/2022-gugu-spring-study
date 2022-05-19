# 5장

컴포넌트 스프링이 직접 클래스를 검색해서 빈으로 등록해주는 기능이다. 스프링이 애노테이션을 찾아 자동으로 빈을 등록해주기 때문에 설정파일에서 따로 빈으로 등록해줄 필요가 사라진다.

<br/>
<hr/>

### @Component 애노테이션으로 스캔 대상 지정

스프링은 @Component 애노테이션이 붙은 클래스를 찾아 자동으로 빈으로 등록한다. @Component에 따로 값을 주지 않았을 경우 클래스 이름의 첫글자를 소문자로 바꾼 이름을 빈 이름으로 사용한다.

> 따로 값을 주지 않았을 경우 : MemberDao -> memberDao  
  따로 값을 주었을 경우 : @Component("listPrinter") -> listPrinter

### @ComponentScan 애노테이션으로 스캔 설정

설정클래스에 @ComponentScan 애노테이션을 붙여 어떤 클래스를 스캔해 빈으로 등록할지 설정할 수 있다.

> @ComponentScan(basePackages = {"spring"})

위 예시에서 `(basePackages = {"spring"})`는 spring 패키지 하위 클래스를 스캔하겠다는 의미이다.

### 스캔 대상에서 제외하거나 포함하기

excludeFilters 속성을 사용하면 스캔할 때 특정 대상을 자동 빈 등록 대상에서 제외할 수 있다.

### 기본 스캔 대상

@Component 애노테이션 이외에도 스프링은 다음 애노테이션이 붙은 클래스를 스캔한다.

- @Component
- @Controller
- @Service
- @Repository
- @Aspect
- @Configuration

@Aspect를 제외한 나머지 애노테이션은 @Component의 특수 애노테이션이다.

# 6장

스프링 컨테이너는 초기화와 종료라는 라이프사이클을 갖는다. 스프링 컨테이너는 초기화할 때 설정클래스에서 정보를 읽어와 알맞은 빈 객체를 생성하고 각 빈을 연결하는 작업을 수행한다.
  
컨테이너 사용이 끝나면 컨테이너를 종료한다. 컨테이너를 종료할 때 사용하는 메서드가 close() 메서드이다.

- 컨테이너 초기화 -> 빈 객체의 생성, 의존 주입, 초기화
- 컨테이너 종료 -> 빈 객체의 소멸

<br/>
<hr/>

### InitializingBean, DisposableBean

빈 객체가 InitializingBean 인터페이스를 구현하면 스프링 컨테이너는 초기화 과정에서 빈 객체의 afterPropertiesSet() 메서드를 실행한다. 
   
빈 객체가 DisposableBean 인터페이스를 구현하면 스프링 컨테이너는 소멸 과정에서 빈 객체의 destroy() 메서드를 실행한다. 

InitializingBean, DisposableBean 인터페이스를 구현하고 싶지 않다면 @Bean 태그에서 initMethod 속성과 destoryMethod 속성을 사용하면 된다.

> @Bean(initMethod = "connect", destroyMethod = "close")

### 싱글톤과 프로토타입

빈 객체는 기본적으로 싱글톤이다. 그러나 Scope를 prototype으로 지정하면 매번 새로운 객체를 생성하도록 만들 수 있다.

# 7장

모든 메서드마다 실행 시간을 측정할 수 있는 기능을 넣어야 한다면 어떻게 할까? 수많은 코드 중복이 발생할 것이다. 애초에 실행 시간을 측정하는 것이 객체가 맡을 책임인지도 의문이다. 이 경우 사용할 수 있는 것이 AOP이다.

<br/>
<hr/>

### 프록시와 AOP

프록시(proxy)란 부가적인 기능을 제공하는 객체를 말한다. 실제 핵심 기능을 실행하는 객체는 대상 객체라고 부른다.  
  
프록시의 특징은 핵심 기능은 구현하지 않는다는 점이다. 대신 여러 객체에 공통으로 적용할 수 있는 기능을 구현한다.  
  
AOP의 기본 개념은 핵심 기능에 공통 기능을 삽입하는 것이다. 핵심 기능에 공통 기능을 삽입하는 방법에는 다음 세 가지가 있다.

1. 컴파일 시점에 코드에 공통 기능을 삽입하는 방법
2. 클래스 로딩 시점에 바이트 코드에 공통 기능을 삽입하는 방법
3. 런타임에 프록시 객체를 생성해서 공통 기능을 삽입하는 방법

여기에서 스프링이 제공하는 AOP 방식은 프록시 객체를 이용한 3번째 방법이다.

### AOP 용어1: Advice

Advice란 `언제` 공통 기능을 핵심 로직에 적용할 지를 정의한다. 가장 많이 사용하는 Advice는 Around이다. Around Advice를 통해 핵심메서드 실행 전, 후, 익셉션 발생 시점에 공통 기능을 실행할 수 있다. Advice는 메서드 위에 붙인다.

### AOP 용어2: Aspect

공통기능으로 사용할 클래스 위에 붙인다.

### AOP 용어3: Pointcut

Pointcut은 공통 기능을 `어디에` 적용할지 나타낸다.

### AOP 용어4: ProceedingJoinPoint

ProceedingJoinPoint을 통해 공통 기능 메서드를 호출할 수 있다.

```java
	public Object execute(ProceedingJoinPoint joinPoint) throws Throwable {
        // 중략...
		joinPoint.proceed(); // 공통기능 메서드 실행
    }
```

### AOP 용어5: execution
execution은 @Pointcut과 함께 사용하며 `어느 패키지`에 공통기능을 적용할 지를 정한다.

```java
	@Pointcut("execution(public * chap07..*(long))")
	public void target() {
	}
```

@Pointcut과 쓰지 않고 @Around 애노테이션으 직접 사용할 수도 있다.
```java
    @Around("execution(public * chap07..*(long))")
	public Object execute(ProceedingJoinPoint joinPoint) throws Throwable {
		joinPoint.proceed();
	}
```

### AOP 적용 순서

@Aspect 메서드와 함께 @Order 애노테이션을 클래스에 붙이면 @Order 애노테이션에 지정한 값에 따라 공통기능 적용 순서를 지정할 수 있다.

@Order 애노테이션 값이 작을 수록 공통기능을 먼저 적용한다.
```java
    @Aspect
    @Order(1)
    public class CacheAspect {
	}
```