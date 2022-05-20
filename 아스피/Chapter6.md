# Chp 6. 빈 라이프사이클과 범위

---

1. 스프링 컨테이너는 초기화와 종료라는 라이프사이클을 갖는다.
    1. 초기화<br>
    ```java
        AnnotationConfigApplicationContext ctx = 
            new AnnotationConfigApplicationContext(AppContext.class); 
    ```

    2. 종료<br>
    ```
    java ctx.close();
   ```

2. 스프링 컨테이너는 초기화시에 *빈 객체의 생성, 의존 주입, 초기화* 를 하고 종료시에 *빈 캑체의 소멸*을 한다.<br>
   이때, 빈 객체의 라이프사이클은 아래의 순서로 이루어진다.<br>
   ```text
        객체 생성 -> 의존 설정 -> 초기화 -> 소멸
    ```


3. 스프링은 빈 객체가 생성되거나 소멸될 때, 빈 객체의 지정한 메서드를 호출할 수 있다.<br>
    1. InitializingBean, DisposableBean 인터페이스를 상속해서 @Overiding을 이용해서 구현할 수 있다.
       <br><br>
    2. 커스텀 메서드 이용<br>
   ```java
   @Bean(initMethod = "connect",destroyMethod = "close")
   ```
   위의 형태로 초기화와 소멸 과정에서 실행시킬 메서드를 지정할 수 있다.(이때, 해당 메서드들에 파라미터가 존재하면 안된다.)


4. 빈 객체의 생성과 관리 범위 빈 객체는 기본적으로 싱글턴으로 생성된다. 만약 매번 새로운 객체를 생성해서 사용하고 싶다면 아래와 같은 방법으로 가능하다.<br>
   ```java
      @Bean
      @Scope("prototype")
   ```
