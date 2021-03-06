# Chapter 6 - 빈 라이프 사이클과 범위

### 컨테이너의 라이프 사이클

- 스프링 컨테이너는 초기화와 종료라는 라이프 사이클을 가짐
- `초기화`: 빈 객체 생성, 의존 주입, 초기화 작업 포함
- `종료`: 빈 객체의 소멸 작업 포함

### 스프링 빈 객체의 라이프 사이클

> 스프링 컨테이너가 관리

1. 객체 생성
2. 의존 설정
3. 초기화 (방법 3가지)
    - `InitializingBean`의 `afterPropertiesSet()` 실행
    - `@PostConstruct` 어노테이션이 있는 메서드 실행
    - `@Bean`의 `initMethod` 속성에서 지정한 메서드 실행
4. 소멸 (방법 3가지)
    - `DisposableBean`의 `destroy()` 실행
    - `@PreDestroy` 어노테이션이 있는 메서드 실행
    - `@Bean`의 `destroyMethod` 속성에서 지정한 메서드 실행

- 스프링 컨테이너가 관리
- 초기화 메서드가 두 번 호출되지 않도록 주의
- ex: DB Connection Pool - 커넥션 풀을 위한 빈 객체는 초기화 과정에서 DB 생성하고 소멸 시 DB 연결 끊음

### 프로토타입 빈의 라이프 사이클

> 프로토타입 빈: 빈 객체를 구할 때마다 매번 새로운 객체를 생성

- 컨테이너를 종료해도 프로토타입 빈 객체의 소멸 메서드를 실행하지는 않음
- 소멸 처리를 코드에서 직접 해야함

> 프로토타입 빈은 왜 쓸까?  
> 의존 관계를 편하게 하기 위해 사용할 수도 있다.  
> 순수 자바 코드로도 해결할 수 있는 부분이기도 하고 실무에서 자주 사용하지는 않는 듯 하다.
> [참고 링크1](https://www.inflearn.com/questions/167692), [참고 링크2](https://www.inflearn.com/questions/415649)
