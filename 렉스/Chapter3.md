# DI (Dependency Injection)
- DI를 우리말로 표현하면 '의존 주입'이다.
  - 의존하는 객체를 직접 생성하는대신 의존 객체를 전달받는 방식을 사용한다.
> 의존이란?
> 의존은 변경에 의해 영향을 받는 관계를 의미한다. 코드의 변경에 따른 영향이 전파되는 관계를 '의존'한다고 표현한다.

- 의존하는 객체를 생성하는 방법에는 클래스 내부에서 직접 생성하는 방법과, 의존 주입 방법이 있다.
  - 의존하는 객체를 클래스 내부에서 생성하면 유지보수 관점에서 문제를 유발할 수 있어서 의존 주입 방법을 사용하는 것이 더 좋다.

## DI를 하는 이유
의존 주입을 하지 않고 의존하는 객체를 클래스 내부에서 생성하면 변경에 취약하다. 반면에 DI를 하면 변경의 유연함을 느낄 수 있다.

- 책의 예시로는 MemberDao를 주입받는 서비스에서 주입 대상이 MemberDao를 상속받은 CachedMemberDao를 사용하려고 하는 예시를 들었다.

## 객체 조립기 (Assembler)
```java
public class Main {
    public static void main(String[] args) {
        MemberDao memberDao = new MemberDao();
        MemberRegisterService registerService = new MemberRegisterService(memberDao);
        ChangePasswordService changePasswordService = new ChangePasswordService(memberDao);
    }
}
```
의존성 주입을 할 때는 위의 코드와 같이 주입할 객체를 만들고 생성자 또는 setter를 통해 주입해줄 수 있다.
하지만 이 방법보다 더 좋은 방법은 객체를 생성하고 의존 객체를 주입해주는 클래스를 따로 작성하는 것이다!!

이 책에서는 의존 객체를 주입하며 서로 다른 두 객체를 조립한다는 의미에서 객체 조립기라고 표현했다.

```java
public class Assembler {
    
    private MemberDao memberDao;
    private MemberRegisterService memberRegisterService;
    private ChangePasswordService changePasswordService;
    
    public Assembler() {
        memberDao = new MemberDao();
        memberRegisterService = new MemberRegisterService(memberDao);
        changePasswordService = new ChangePasswordService(memberDao);
    }

    public MemberDao getMemberDao() {
        return memberDao;
    }

    public MemberRegisterService getMemberRegisterService() {
        return memberRegisterService;
    }

    public ChangePasswordService getChangePasswordService() {
        return changePasswordService;
    }
}
```

# 스프링의 DI
- 스프링은 DI를 지원하는 조립기이다. (스프링은 Assembler와 유사한 기능을 제공한다.)
- 스프링은 위의 Assembler의 생성자 코드처럼 필요한 객체를 생성하고 생성한 객체에 의존을 주입하며 getter를 통한 메서드처럼 객체를 제공해준다.
- Assembler 코드와의 차이점은 Assembler는 특정 타입의 클래스만 생성하는 반면에 스프링은 범용 조립기이다.

```java
public static void main(String[] args) throws IOException{
    ctx=new AnnotationConfigApplicationContext(AppCtx.class);
        ....
}
```
Assembler와 Spring의 차이점은 Assember대신 스프링 컨테이너(Application Context)를 사용했다는 점이다.
- `AnnotationConfigApplicationContext`는 스프링 컨테이너를 생성해준다.

# DI 주입 방법 종류
## 생성자 주입 vs setter 주입
생성자 주입과 setter주입은 각각의 장점이 있다.
- 생성자 방식: 빈 객체를 생성하는 시점에 모든 의존 객체가 주입된다.
  - 단점: 의존 객체가 많을 경우, 각각의 인자들이 어떤 의존객체를 설정하는지 알려면 생성자를 확인해야한다.
- 설정 메서드(setter) 방식: 세터 메서드 이름을 통해 어떤 의존 객체가 주입되는지 알 수 있다.
  - 단점: 의존 객체를 올바르게 주입하지 않을 경우, 사용 시점에 `NullPointerException`이 발생할 수 있다.

## Autowired
- `@Autowired`은 스프링 빈에 의존하는 다른 빈을 자동으로 주입하고 싶을 때 사용한다.
- 해당 어노테이션을 붙여주면 해당 타입의 빈을 찾아서 해당 필드에 할당해준다.
  - 의존하는 빈들을 클래스 코드에서 `@Autowired`를 사용하여 필드에 추가를 하면, `@Bean`메서드를 통해 빈을 생성할 때, 설정 클래스에서 빈의 의존 주입을 위한 코드를 작성하지 않아도 된다.
  - `@Autowired`는 변수와 메서드 둘 다 붙일 수 있다.
```java
public class ChangePasswordService {
    @Autowired
    private MemberDao memberDao;
    ...
}


@Configuration
public class AppCtx2 {
     ...
    @Bean
    public ChangePasswordService changePwdSvc() {
        ChangePasswordService pwdSvc = new ChangePasswordService();
        pwdSvc.setMemberDao(memberDao); // 해당 라인 생략 가능
        return pwdSvc;
    }
    ...   
}
```
## 설정 클래스의 분리 방법
- `@Autowired` 또는 `@Import`를 통해 다른 설정클래스에서 추가한 빈을 불러와서 해당 설정 클래스에서도 사용할 수 있다.
  - `@Import`를 사용할 경우 import된 설정 클래스는 스프링 컨테이너를 생성할 때 지정하지 않아도 import되었기에 자동으로 초기화된다.

## `@getBean()` 메서드의 사용
- `@getBean()`의 첫번째 인자는 빈의 이름이고 두 번째 인자는 빈의 타입이다.
- 해당 메서드를 호출할 때 존재하지 않는 빈 이름을 사용하면 `No bean named~~~`메시지를 가진 예외가 발생한다.
- 빈의 실제 타입이 `@getBean()` 메서드에 지정한 타입이 다르면 `Bean named ~~ is expected to be of type ~~~ but was actually of type ~~`의 메시지를 포함한 예외가 발생한다.
- 메서드에서 첫번째 인자인 빈의 이름을 작성하지 않고 빈의 타입만 인자로 줘도 된다. 이런 경우 해당 타입의 빈이 하나일 경우에만 해당 빈을 가져오고 해당 타입의 빈이 2개 이상이면 `No qualifying bean of type ~~ available: expected single matching bean but found 2:~~`와 같은 메시지를 포함한 예외가 발생한다.

# 주입 대상 객체의 고민
- Bean의 주입 대상은 꼭 Bean이 아니고 일반 객체를 생성하여 주입하여도 된다.
- 객체를 스프링 빈으로 등록할 때와 등록하지 않을 때의 차이는 스프링 컨테이너가 객체를 관리하는지 여부이다.
- 스프링 컨테이너는 빈으로 등록한 객체들에게 자동 주입, 라이프사이클 관리 등 단순 객체 생성 외에 객체 관리를 위한 다양한 기능을 제공한다.
  - 그래서 스프링 컨테이너가 제공하는 관리 기능이 필요 없고, `getBean()`을 통해 빈 객체를 반환받을 필요가 없으면 빈 객체로 등록하지 않아도 된다.

최근에는 의존 자동 주입 기능을 프로젝트 전반에 걸쳐 사용하는 추세라서, 의존 주입 대상은 스프링 빈으로 등록하는 것이 일반적이다.
