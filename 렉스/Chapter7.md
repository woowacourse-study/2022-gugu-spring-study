# AOP
## 준비
Spring에서 AOP를 사용하기 위해서는 `aspectjweaver`의존성을 추가해줘야 한다.
- `aspectjweaver` 모듈은 AOP를 설정하는데 필요한 어노테이션을 제공한다.
- 스프링의 AOP 기능은 `spring-aop`모듈이 제공하는데, 해당 모듈은 `spring-context` 모듈을 의존 대상에 추가하면 자동으로 포함된다.
- `aspectjweaver`의존성은 `implementation 'org.springframework:spring-aspects:5.3.19'`를 통해 주입시키면 된다.

- [MavenRepository-aspectjweaver](https://mvnrepository.com/artifact/org.aspectj/aspectjweaver)

## 프록시(Proxy)란?
책에서 소개하는 Calculator예시를 보면 factorial메서드의 핵심 로직들은 Calculator 인터페이스를 상속한 ImpeCalculator와 RecCalculator들이 갖고
ExeTimeCalculator는 Calculator 인터페이스를 상속하지만 factorial의 핵심 로직을 다른 객체에게 위임하고 부가적인 기능을 제공합니다.

우리는 이와 같이 다른 객체에 핵심 기능을 위임하고 부가적인 기능을 제공하는 객체를 프록시(Proxy)라고 합니다. (ExeTimeCalculator가 Proxy이다)

> 책에서 위의 예시들은 프록시보다는 데코레이터(decorator)에 가깝다고 한다. 프록시는 접근 제어 관점에 초점이 맞춰져 있고, 데코레이터는 기능 추가와 확장에 초점이 맞춰져 있다.

## AOP란?
- AOP는 Aspect(관심) Oriented Programming의 약자로, 여러 객체에 공통으로 적용할 수 있는 기능을 분리해서 재사용성을 높여주는 프로그래밍 기법이다.
- 핵심 기능과 공통 기능을 분리함으로써 **핵심 기능을 구현한 코드의 수정 없이 공통 기능을 적용 및 수정할 수 있다.**
- 스프링은 프록시를 이용해 AOP를 구현한다. (AOP를 적용하여 실행되는 메서드들은 전부 생성한 빈 타입이 아니고 프록시 타입으로 실행된다.)

### 핵심 기능에 공통 기능을 넣는 방법
- 방법 1. 컴파일 시점에 코드에 공통 기능을 삽입하는 방법
  - AOP 개발 독가 소스코드 컴파일 전에 공통 구현 코드를 소스코드에 삽입하는 방식으로 동작한다.

- 방법 2. 클래스 로딩 시점에 바이트 코드에 공통 기능을 삽입하는 방법
- 방법 3. 런타임에 **프록시 객체를 생성해서 공통 기능을 삽입하는 방법**

> 1, 2 번 방법은 스프링 AOP에서 지원하지 않아 AspectJ와 같은 AOP전용 도구를 사용해 적용해야 한다.
> 스프링은 3번 방식을 제공하고있다.

> 📌 스프링은 프록시를 이용해 메서드 호출 시점에 Aspect를 적용한다.

### AOP 주요 용어
- **Aspect**: AOP에서의 공통 기능을 의미한다. 트랜잭션이나 보안 등이 좋은 예시이다.
- **Advice**: 언제 공통 관심 기능을 핵심 로직에 적용할지를 정의하고 있다.
- **Joinpoint**: Adcice를 적용 가능한 지점을 의미한다. 메서드 호출, 필드 값 변경 등이 해당된다. (스프링은 프록시를 사용해서 AOP를 구현하기 때문에 메서드 호출에 대한 Joinpoint만 지원한다.)
- **Pointcut**: Joinpoint의 부분 집합으로 실제 Advice가 적용되는 Joinpoint를 나타낸다. (스프링에서는 정규 표현식이나 AspectJ의 문법을 이용하여 Pointcut을 정의할 수 있다.)
- **Weaving**: Advice를 핵심 로직 코드에 적용하는 것을 의미한다.

### Advice의 종류
- **Before Advice**: 대상 객체의 메서드 호출 전에 기능을 실행한다.
- **After Returning Advice**: 대상 객체의 메서드가 예외 없이 실행된 이후에 공통 기능을 실행한다.
- **After Throwing Advice**: 대상 객체의 메서드를 실행하는 도중 익셉션이 발생한 경우에 공통 기능을 실행한다.
- **After Advice**: 예외 발생 여부에 상관없이 대상 객체의 메서드 실행 후에 공통 기능을 실행한다.
- **Around Advice**: 대상 객체의 메서드 실행 전, 후 또는 예외 발생 시점등 다양한 시점에 원하는 기능을 실행할 수 있다. (캐시, 성능 모니터링과 같은 Aspect를 구현할 때 주로 이용한다.)

> Around Advice가 다양한 시점에 사용 가능하여 가장 널리 사용된다. 이후 나올 내용도 모두 Around Advice 내용이다.

## 스프링 AOP 적용 방법
1. Aspect로 사용할 클래스에 @Aspect 어노테이션을 붙인다. 
2. @Pointcut 어노테이션으로 공통 기능을 적용할 Pointcut을 정의한다.
3. 공통 기능을 구현한 메서드에 @Around 어노테이션을 적용한다.

> 프록시는 스프링 프레임워크가 알아서 만들어준다.

```java
@Aspect
public class ExeTimeAspect {
    
    @Pointcut("execution(public * ch7.calculator..*(..))")
    private void publicTarget() {
    }
    
    @Around("publicTarget()")
    public Object measure(ProceedingJoinPoint joinPoint) throws Throwable {
        long start = System.nanoTime();
        try {
            return joinPoint.proceed();
        } finally {
            long finish = System.nanoTime();
            Signature sig = joinPoint.getSignature();
            System.out.printf("%s.%s(%s) 실행 시간 : %d ns\n",
                    joinPoint.getTarget().getClass().getSimpleName(),
                    sig.getName(), Arrays.toString(joinPoint.getArgs()),
                    (finish - start));
        }
    }
}

@Configuration
@EnableAspectJAutoProxy
public class AppCtx {

    @Bean
    public ExeTimeAspect exeTimeAspect() {
      return new ExeTimeAspect();
    }
  ...
}
```

위의 코드를 보면 `@Advice`를 통해 해당 클래스가 Aspect라는 것을 명시한 후, `@Pointcut`에 명시한 메서드의 실행 전/후(`@Around`)에 `measure`메서드를 실행하겠다는 것을 명시한다.

- `@Pointcut`
  - 공통 기능을 적용할 대상을 설정한다. 
    - 위 코드에서는 ch7.calculator 패키지와 그 하위 패키지에 위치한 타입의 public메서드를 Pointcut으로 설정하겠다는 의미이다.
  - 패키지 위치 설정할 때, 순환 참조를 조심해야한다.
- `@Around`
  - value값으로 설정한(`publicTarget()`)메서드에 정의한 메서드 전/후에 해당 메서드를 실행한다. 
    - 위의 Pointcut에서 지정한 ch7.calculator 하위에 속한 빈 객체의 Public메서드 전후에 @Around가 붙은 아래의 measure()메서드를 적용한다.
- `@ProceedingJoinPoint`
  - measure메서드에 파라미터로 있는 `@ProceedingJoinPoint`는 프록시 대상 객체 메서드를 호출할 떄 사용한다.
    - 호출은 `joinPoint.proceed();`와 같이 하면 된다.
    - `joinPoint.proceed();`를 통해 대상 객체 메서드를 호출하기에 원하는 공통 기능들을 해당 메서드 전후에 올바르게 위치시켜야 한다.
    - `joinPoint.getSignature();`: 호출되는 메서드에 대한 정보를 구한다.
    - `joinPoint.getTarget();`: 대상 객체를 구한다.
    - `joinPoint.getArgs();`: 파라미터 목록을 구한다.
> **Signature**
> - 자바에서는 메서드 이름과 파라미터를 합쳐서 메서드 시그니처라고 한다.
> - `joinPoint.getSignature();`를 통해서는 호출한 메서드의 시그니처를 호출할 수 있다.

- `@EnableAspectJAutoProxy`
  - Aspect를 적용하기 위해서는 해당 어노테이션을 설정 클래스에 붙인 후 Aspect클래스를 Bean으로 등록해줘야 한다.
  
> 위의 코드를 통해 aspect가 적용되는 메서드를 실행해보면, 해당 객체는 원본 타입의 클래스가 아닌 Proxy 타입으로 실행이 된다.
> 그 이유는 AOP를 적용할 경우 스프링이 실행하고자 하는 메서드를 실행하기 전에 Proxy객체를 생성하여 공통기능 적용 후 실행 메서드를 실행하기 떄문이다.

> - `@Around("execution(public * ch7.calculator..*(..))")`와 같이 `@Pointcut`이 아닌 `@Around`에 execuiton명시자를 직접 지정할 수도 있다.
> - `@Pointcut`은 공통된다면 메서드를 public으로 지정을 하여 재사용할 수도 있다.
>    - 공통으로 사용되는 Pointcut들은 별도의 클래스를 만들어 관리하는 것도 좋은 방법이다. (해당 클래스는 빈으로 등록할 필요가 없다.) 
    


## 프록시 생성방식
📌 프록시는 객체를 생성할 때 빈 객체가 인터페이스를 상속하면 인터페이스를 이용해서 프록시를 생성한다.

그래서 인터페이스를 상속받은 하위 클래스를 이용해 `getBean()`을 사용할 경우, 다음과 같은 에러가 발생한다.
```shell
Exception in thread "main" org.springframework.beans.factory.BeanNotOfRequiredTypeException: Bean named 'calculator' is expected to be of type 'ch7.calculator.RecCalculator' but was actually of type 'com.sun.proxy.$Proxy18'
```

빈 객체가 인터페이스를 상속할 떄 인터페이스가 아닌 클래스를 이용해서 프록시를 생성하고 싶다면 다음과 같이 `EnableAspectJAutoProxy`의 속성을 설정해야 한다.
`@EnableAspectJAutoProxy(proxyTargetClass = true)`와 같이 `proxyTargetClass`속성을 true로 변경해주면 인터페이스가 아닌 자바 클래스를 상속받아 프록시를 생성해줘서 구현(실제, 하위)클래스를 이용해 빈 객체를 구할 수 있습니다.

## Pointcut의 execution
> execution(수식어패턴? 리턴타입패턴 클래스이름패턴? 메서드이름패턴(파라미터패턴))

execution의 기본 형식은 다음과 같다. 
- 수식어패턴
  - 생략 가능하며 public, protected등이 온다. 하지만 스프링 AOP에서는 public에서만 가능하다.
- 리턴타입패턴
  - 반환 타입을 명시한다.
- 클래스이름패턴, 메서드이름패턴, 파라미터패턴
  - 클래스 및 메서드 이름을 패턴으로 명시한다.

책 169페이지 예시 참조할 것.

## 여러 Advice 적용
스프링에서는 한나의 Pointcut에 여러 Advice를 적용할 수 있다.

Advice를 적용하는 방법은 위와 동일하게 Aspect를 생성하고 Bean을 등록해주면 된다. 
하지만 어떤 Aspect가 먼저 적용될지는 스프링 프레임워크나 자바 버전에 따라 달라질 수 있기에 적용 순서가 중요하면 `@Order`를 통해 직접 순서를 지정해줘야 한다.
- `@Order`은 값이 작을수록 먼저 적용된다.
