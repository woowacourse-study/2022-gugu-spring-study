# Chp 5. 컴포넌트 스캔

---

1. 스프링이 검색해서 빈으로 등록하려면 클래스에 @Component 애노테이션이 붙어있어야 한다.<br>
   이때, 애노테이션에 속성값을 주면 해당 값이 빈 객체의 이름이 된다.


2. @ComponentScan 애너테이션을 설정 클래스에 붙이면 @Component가 붙은 클래스들을 찾아내어 빈 객체로 등록한다. 이때, @ComponentScan에 basePackages 속성값을 주면 해당
   패키지 아래에 있는 @Component들을 찾아내어 등록한다.


3. 스캔에서 제외하고자 한다면 @ComponentScan 애노테이션의 excludeFilters 속성을 이용한다.<br>
   ```excludeFilters = @Filter()``` 를 사용하고 @Filter 애너테이션의 type, pattern, classes 속성등의 값을 이용해 제외할 수 있다.<br>


4. @Component 애너테이션 뿐아니라 해당 애너테이션이 붙어있는 특수 애너테이션, @Aspect 애너테이션 또한 스캔 대상이 된다.


5. 컴포넌트 스캔에 따른 충돌 처리
   <br><br>
    1. 수동 등록한 빈과 충돌<br>
    ```java
   @Component
   public class MemberDao{
   }    
   ```
   위의 MemberDao가 자동 등록된 상황이다.<br>

    ```java
    @Bean
    public MemberDao memberDao(){
        MemberDao memberDao = new MemberDao();
        return memberDao;
    }
    ```
   이 상황에서 설정 클래스에 위의 빈 객체를 수동 등록하면 memberDao라는 이름이 겹친다. 이 경우, 수동 등록한 빈이 우선 등록되고 MemberDao 타입 빈은 하나만 존재한다.
   <br><br>

    2. 빈 이름 충돌
    ```java
    @Configure
    @ComponentScan(basePackages = {"spring","spring2"})
   public class AppCtx{}
    ```
   위의 상황에서 spring과 spring2의 패키지에 같은 이름의 빈 객체가 있다면 익셉션이 발생한다. 이런 경우, 최소 하나에 꼭 이름을 지정해주어야한다.
