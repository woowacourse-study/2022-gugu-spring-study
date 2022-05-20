#스프링 프로그래밍 입문

## 5장 컴포넌트 스캔
### @Component("name")
스프링이 Bean으로 등록할 스캔 대상에 붙이는 Annotation이다.
- 속성값으로 빈 이름을 지정할 수 있다. 기본 값은 첫 글자가 소문자인 클래스 이름이다.

### @ComponentScan
@Configuration이 붙은 설정 클래스가 @Component가 붙은 클래스를 Bean으로 등록하게 하는 Annotation이다.
- basePackages 속성으로 포함할 package를 지정할 수 있다.
- excludeFilters 속성에 @Filter를 사용하면 제외할 클래스를 지정할 수 있다.
- @Qualifier로 이름을 지정할 수 있다.

## 6장 빈 라이프사이클과 범위
###컨테이너 초기화와 종료
빈 설정에서 직접 메서드를 지정할 경우 초기화와 종료가 한 번만 실행되게 해야 한다.
- 컨테이너 초기화: 빈 객체의 생성, 의존 주입, 초기화
  - InitializingBean 인터페이스
  - Bean 태그의 initMethod 속성에서 메서드 이름 지정
- 컨테이너 종료: 빈 객체의 소멸
  - DisposableBean 인터페이스
  - Bean 태그의 destroyMethod 속성에서 메서드 이름 지정

###빈 객체의 생성과 관리 범위
Bean은 기본적으로 Singleton scope를 갖는다.
@Scope("prototype")를 Bean과 같이 사용하면 프로토타입 범위의 빈을 사용할 수 있다.
이때 Bean 객체의 소멸처리는 코드에서 직접 해야한다.

## 7장AOP프로그래밍
### Proxy
프록시란 핵심 기능의 실행은 다른 객체에 위임하고 부가적인 기능을 제공하는 객체이다. 공통 기능 구현을 프록시에 위임한다.

### Aspect Oriented Programming AOP
여러 객체에 공통으로 적용할 수 있는 기능을 분리해서 재사용성을 높여주는 프로그래밍 기법이다.
- 코드의 수정 없이 공통 기능을 적용 할 수 있게 만들어 준다. 즉, 핵심 기능의 코드를 수정하지 않으면서 공통 기능의 구현을 추가할 수 있다.

#### AOP적용 방법
1. 컴파일 시점
2. 클래스 로딩 시점
3. 런타임 시점
스프링은 프록시를 이용한 AOP를 지원한다. 프록시 객체를 자동으로 만들어준다.

### AOP용어
- Advice: 공통 관심 기능을 핵심 로직에 적용할 때를 정의한다.
- Joinpoint: Advice를 적용 가능한 지점을 의미한다.
- Pointcut: Joinpoint의 부분 집합이다.
- Weaving: Advice를 핵심 로직 코드에 적용하는 것이다.
- Aspect: 여러 객체에 공통으로 적용되는 기능이다.

### Advice의 종류
주로 **Around Advice**를 사용한다.
- Before: 대상 객체의 호출 전에 공통 기능을 실행한다.
- After Returning: 대상 객체의 메서드가 예외 없이 실행된 이후에 공통 기능을 실행한다.
- After Throwing: 대상 객체의 메서드를 실행하는 도중 예외가 발생한 경우에 공통 기능을 실행한다.
- After: 익센션 발생 여부에 상관없이 대상 객체의 메서드 실행 후 공통 기능을 실행한다.
- Around: 대상 객체의 메세드 실행 전, 후 또는 예외 발생 시점에 공통 기능을 실행한다.

### 스프링AOP구현
1. Aspect로 사용할 클래스에 @Aspect를 붙인다.
2. @Pointcut으로 공통 기능을 적용할 Pointcut을 정의한다.
3. 공통 기능을 구현한 메서드에 @Around를 적용한다.
