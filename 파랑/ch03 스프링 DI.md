# Chapter 3 스프링 DI

## DI : Dependency Injection 의존 주입

- 의존
    - 객체 간의 의존
    - 한 클래스가 다른 클래스의 메서드를 실행할 때 ‘의존'한다고 표현
    - 변경에 의해 영향을 받는 관계
- 객체 직접 생성 vs DI
    - 클래스 내부에서 객체를 직접 생성하면 생성이 쉽긴 하지만 유지보수가 힘들다.
    - DI를 통해 의존을 처리하면 변경에 유연한 코드를 구현할 수 있다. 객체를 사용하는 클래스가 여러개라도 변경할 곳은 의존 주입 대상이 되는 객체를 생성하는 코드 한 곳 뿐이기 때문이다.

## 객체 조립기(assembler)

```java
public class Main {
	public static void main(String[] args) {
		MemberDao memberDao = new MemberDao();
		MemberRegisterService regSvc = new MemberRegisterService(memberDao);
		ChangePasswordService pwdSvc = new ChangePasswordService();
		pwdSvc.setMemberDao(memberDao);
		...
    }
}
```

- 위 코드처럼 main 메서드에서 의존 대상 객체를 생성하고 주입하는 것보다 좀 더 나은 방법은 객체를 생성하고 의존 객체를 주입해주는 클래스를 따로 작성하는 것이다.
- 조립기: 의존 객체를 주입한다는 것은 서로 다른 두 객체를 조립한다고 생각할 수 있기 때문에 이 클래스를 조립기라고 표현한다.

```java
public class Assembler {
    
    private MemberDao memberDao;
    private MemberRegisterService regSvc;
    private ChangePasswordService pwdSvc;
    
    public Assembler() {
        memberDao = new MemberDao();
        regSvc = new MemberRegisterService(memberDao);
        pwdSvc = new ChangePasswordService();
        pwdSvc.setMemberDao(memberDao);
    }

    // ... getter
}
```

- 객체 조립기를 사용하면 의존 객체를 변경시 조립기의 코드만 수정하면 된다.

## 스프링의 DI

- 스프링이 DI를 지원하는 조립기의 역할을 한다.

```java
@Configuration
public class AppCtx {

	@Bean
	public MemberDao memberDao() {
		return new MemberDao();
	}
	
	@Bean
	public MemberRegisterService memberRegSvc() {
		return new MemberRegisterService(memberDao());
	}
	
	@Bean
	public ChangePasswordService changePwdSvc() {
		ChangePasswordService pwdSvc = new ChangePasswordService();
		pwdSvc.setMemberDao(memberDao());
		return pwdSvc;
	}
	...
```

- `AnnotationConfigApplicationContext` 클래스를 이용해 스프링 컨테이너를 생성할 수 있다.
- 생성한 컨테이너의 `getBean()` 메서드를 통해 사용할 객체를 구할 수 있다.

    ```java
    MemberRegisterService regSvc = ctx.getBean("memberRegSvc", MemberRegisterService.class);
    ```


### DI 방식

1. **생성자 방식**
    - 장점: bean 객체를 생성하는 시점에 모든 의존 객체가 주입되어 완전한 상태로 사용할 수 있다.
    - 단점: 생성자의 파라미터 개수가 많을 경우 각 인자가 어떤 의존 객체를 설정하는지 알아내기 위해 생성자의 코드를 확인해야 한다.
2. **setter 메서드 방식**
    - 장점: setter 메서드 이름을 통해 어떤 의존 객체가 주입되는지 알 수 있다.
    - 단점: setter 메서드를 사용해서 필요한 의존 객체를 전달하지 않아도 bean 객체가 생성되기 때문에 객체를 사용할 때 NullPointerException이 발생할 수 있다.

### 두 개 이상의 설정 파일 사용

- `@Autowired` 을 사용하면 생성자나 setter 메서드를 사용하지 않아도 스프링 빈에 의존하는 다른 빈을 자동으로 주입할 수 있다.
- `@Import` 는 클래스 위에 붙여 함께 사용할 설정 클래스를 지정한다. 클래스 사용시 import된 설정 클래스도 자동으로 함께 초기화된다.

### getBean() 메서드 사용

- 일반적인 사용 방식
- 빈 이름을 지정하지 않고 타입만 넣어줘도 가능

    ```java
    // 일반적인 사용 방식
    VersionPrinter versionPrinter = ctx.getBean("versionPrinter", VersionPrinter.class);
    
    // 해당 타입의 bean 객체가 하나라면 이름을 지정하지 않아도 가능
    VersionPrinter versionPrinter = ctx.getBean(VersionPrinter.class);
    
    ```
