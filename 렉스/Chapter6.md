# 빈 라이프 사이클과 범위

## 스프링 컨테이너의 라이프 사이클
스프링 컨테이너는 초기화와 종료의 라이프 사이클을 갖는다.

- 컨테이너 초기화 -> 빈 객체의 생성, 의존 주입, 초기화
- Running -> 빈 객체 사용(`getBean()`)
- 컨테이너 종료 -> 빈 객체의 소멸

다음과 같이 컨테이너의 라비프 사이클에 따라 빈 객체도 자연스럽게 생성과 소멸을 한다.

## 빈의 라이프 사이클
- 객체 생성
- 의존 결정 
  - 의존 자동 주입을 통한 의존 설정과 설정 클래스에 있는 의존 주입들이 모두 수행된다.
- 초기화 
  - 의존 결정이 완료되면 스프링 빈은 빈 객체의 지정된 메서드를 호출하여 빈을 초기화해준다.
- 소멸
  - 스프링 컨테이너가 종료되면 스프링 컨테이너가 빈 객체를 소멸시킨다.

### 빈 라이프 사이클 관련 스프링 인터페이스
> - org.springframework.beans.factory.InitializingBean;
>  - `afterPropertiesSet()` 재정의
> - org.springframework.beans.factory.DisposableBean;
>    - `destroy()` 재정의

- 빈의 라이프 사이클에서 초기화 및 소멸을 하는 과정에 특정 로직의 실행이 필요하다면 위의 인터페이스들을 상속받아 각각의 메서드를 재정의해주면 된다.
- 초기화와 소멸 과정이 필요한 예시로는 데이터베이스 커넥션 풀과 클라이언트 채팅 등이 존재한다.
- 외부에서 제공받은 클래스와 같이 InitializingBean과 DisposableBean 인터페이스를 구현하기 어려운 경우 아래와 같이 `@Bean` 태그의 `initMethod`속성과 `destroyMethod`속성을 통해 초기화/소멸 메서드의 이름을 지정하면 된다.
```java
    @Bean(initMethod = "connect", destroyMethod = "close")
    public Client2 client2() {
        Client2 client2 = new Client2();
        client2.setHost("host");
        return client2;
    }
    // connect와 close는 Clients에 정의되어있는 메서드 이름
```
- `initMethod`의 경우 빈을 초기화 과정에서 시행되는 것이어서 빈 생성 메서드에서 직접 실헹을 하여도 된다.
  - 하지만 `InitializingBean`를 상속하여 `afterPropertiesSet()`를 재정의한 경우에는 빈의 초기화 과정에서 `afterPropertiesSet()`를 호출하기 때문에 중복된 호출이 발생하지 않도록 주의해야한다.

> `initMethod`, `destroyMethod`에서 호출되는 메서드는 파라미터가 없어야한다. 파라미터가 있을 경우 spring이 예외를 발생시킨다.

# 프로토타입 빈
일반적으로 스프링의 빈은 싱글톤으로 생성되어 사용된다.
싱글톤으로 사용되는 빈을 `getBean()`을 할 때마다 새로운 객체를 생성하도록 하려면 빈의 범위(Scope)을 프로토타입으로 설정해주면 된다.

사용 예시는 다음과 같습니다.
```java
    @Bean
    @Scope("prototype")
    public Client client() {
        Client client = new Client();
        client.setHost("host");
        return client;
    }
```
(`@Scope`의 기본값은 싱글톤으로 생성하도록 되어 있습니다.)

위와 같이 프로토 타입 빈을 사용하면 빈은 객체 생성, 프로퍼티 설정 및 초기화 작업까지는 수행하지만, 컨테이너를 종료한다고 해서 생성한 빈의 프로토타입 빈의 소멸 메서드를 실행하지는 않습니다.
즉, 소멸 과정이 실행되지 않아 프로토타입 범위의 빈을 사용할 때는 빈 객체의 소멸 처리를 코드에서 직접 해줘야 합니다.
