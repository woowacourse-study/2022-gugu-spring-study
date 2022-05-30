# 8장. DB 연동

### JDBC 프로그래밍의 단점을 보완하는 스프링

JDBC API 를 사용하면 DB 연동에 필요한 Connection 을 구한 다음 쿼리를 실행하기 위한 PreparedStatement 를 생성하고, 쿼리를 실행한 뒤에는 finally 블록에서 ResultSet, PreparedStatement, Connection 을 닫아야한다.

데이터 처리와는 상관없는 코드를 반복해야 하기 때문에 이를 줄이기 위해 템플릿 메서드 패턴과 전략 패턴을 함께 사용한다. 스프링은 이 두 패턴을 엮은 JdbcTemplate 클래스를 제공한다.

### 커넥션 풀(Connection Pool) 이란?

- DBMS 로 커넥션을 생성하는 시간은 전체 성능에 영향을 줄 수 있다.
- 최초 연결에 따른 응답 속도 저하와 동시 접속자가 많을 때 발생하는 부하를 줄이기 위해 사용하는 것이 `커넥션 풀`이다.
- 일정 개수의 DB 커넥션을 미리 만들어 두는 기법이다.
- 커넥션을 미리 생성해두기 때문에 커넥션을 사용하는 시점에서 **커넥션을 생성하는 시간을 아낄 수 있다**.
- 동시 접속자가 많더라도 커넥션을 생성하는 부하가 적기 때문에 **더 많은 동시 접속자를 처리할 수 있다.**
- 커넥션도 일정 개수로 유지해서 **DBMS 에 대한 부하를 일정 수준으로 유지할 수 있다.**
- DB 커넥션이 필요하면 커넥션 풀에서 커넥션을 가져와 사용한 뒤 커넥션 풀에 다시 반납한다.
- DB 커넥션 풀 제공 모듈 : **Tomcat JDBC , HikariCP** , DBCP, c3p0 등
    - * 해당 책에서는 Tomcat JDBC 기반으로 설명

### Tomcat JDBC 의 주요 프로퍼티

- Tomcat JDBC 모듈의 **DataSource 클래스**
    - 커넥션을 몇 개 만들지 지정할 수 있는 메서드 제공

**커넥션의 상태**

- 활성 상태
    - 커넥션 풀에 커넥션을 요청한 경우
    - `dataSource.getConnection()`
- 유휴 상태
    - 커넥션을 다시 커넥션 풀에 반환한 경우
    - `connection.close()`

- `maxActive` : 활성 상태가 가능한 최대 커넥션 수
- `maxWait` : 커넥션이 모두 사용중이라서 반환되길 기다리는 대기 시간 (대기 시간이 지나면 익셉션 발생)
- `initialSize` : 커넥션 풀 초기화시 최소 생성할 커넥션 개수

커넥션 풀에 있는 커넥션은 지속적으로 재사용된다.

- 일정 시간(ex.5분) 내에 쿼리를 실행하지 않으면 DB 연결을 끊는데, 어떤 커넥션이 5분이상 유휴 상태였다면 해당 커넥션 연결을 끊지만 커넥션은 여전히 풀 속에 남아있다.
- 이 상태에서 해당 커넥션을 풀에서 가져와서 사용하면 연결이 끊어진 커넥션이므로 익셉션이 발생한다.
- 커넥션 풀의 커넥션이 유효한지 주기적으로 검사해야 함
    - `minEvictableIdleTimeMillis`
    - `timeBetweenEvictionRunsMillis`
    - `testWhileIdle`
    

### 스프링의 익셉션 변환 처리

- JDBC API 를 사용하는 과정에서 `SQLException` 이 발생하면, 이 익셉션을 알맞는 `DataAccessException` 으로 변환해서 발생한다.

**스프링은 왜  `SQLException` → `DataAccessException` 으로 변환할까?**

- 연동 기술에 상관없이 동일하게 익셉션을 처리할 수 있도록 하기 위함이다.
- 스프링은 JDBC, JPA, Hibernate 에 대한 연동을 지원하고 MyBatis 는 자체적으로 스프링 연동 기능을 제공한다.
- 각각의 구현 기술마다 익셉션을 다르게 처리해야 한다면 개발자는 기술마다 익셉션 처리 코드를 작성해야 할 것이다. 각 연동 기술에 따라 발생하는 익셉션을 스프링이 제공하는 익셉션으로 변환함으로 써 구현 기술에 상관없이 동일한 코드로 익셉션 처리가 가능해졌다.

### 트랜잭션 처리

**트랜잭션이란?**

- 두 개 이상의 쿼리를 한 작업으로 실행해야 할 때 사용하는 것이다.
- **여러 쿼리를 논리적으로 하나의 작업으로 묶어준다.**
- 트랜잭션으로 묶인 쿼리 중 하나라도 실패하면 전체 쿼리를 실패로 간주하고 실패 이전에 실행한 쿼리를 취소한다.
- 쿼리 실행 결과를 취소하고 DB 를 기존 상태로 되돌리는 것을 롤백(rollback)이라고 한다.
- 트랜잭션을 커밋하거나 롤백할 때까지 실행한 쿼리들이 하나의 작업 단위가 된다.
- `setAutoCommit(false)`
- `commit()`
- `rollback()`

**@Transactional**

- 스프링이 제공하는 @Transactional 어노테이션을 사용해서 트랜잭션 범위를 지정한다.
- 트랜잭션 범위에서 실행하고 싶은 메서드에 붙히면 된다.

**@Transactional** 이 제대로 동작하려면 해당 내용을 스프링 설정에 추가해야 한다

- 플랫폼 트랜잭션 매니저(PlatformTransactionManager) 빈 설정
- @Transactional 어노테이션 활성화 설정

**PlatformTransactionManager**

- 스프링이 제공하는 트랜잭션 매니저 인터페이스
- 구현기술에 상관없이 동일한 방식으로 트랜잭션 처리를 하기 위해 해당 인터페이스를 제공한다.
- JDBC 는 `DataSourceTransactionManager` 클래스를 `PlatformTransactionManager` 로 사용한다.

**@EnableTransactionManagement**

- @Transactional 어노테이션이 붙은 메서드를 트랜잭션 범위에서 실행하는 기능을 활성화한다.
- 설정 클래스에 붙혀준다?

### 트랜잭션을 시작하고, 커밋하고, 롤백하는 것은 누가 어떻게 처리하는 걸까?

### @Transactional 과 프록시

- 트랜잭션도 공통 기능 중 하나이다.
- 스프링은 @Transactional 어노테이션을 이용해서 트랜잭션을 처리하기 위해 내부적으로 AOP 를 사용한다.
- 스프링은 @Transactiona 어노테이션이 적용된 빈 객체를 찾아서 알맞은 프록시 객체를 새성한다.

![이미지]( https://www.google.com/url?sa=i&url=https%3A%2F%2Fblog.hax0r.info%2F2021-03-01%2Fspring-transaction-management%2F&psig=AOvVaw3MaKcNWNgMoQRSS8REqCl6&ust=1653714173546000&source=images&cd=vfe&ved=0CAwQjRxqFwoTCMDyrMbz_vcCFQAAAAAdAAAAABAJ](https://www.google.com/url?sa=i&url=https%3A%2F%2Fblog.hax0r.info%2F2021-03-01%2Fspring-transaction-management%2F&psig=AOvVaw3MaKcNWNgMoQRSS8REqCl6&ust=1653714173546000&source=images&cd=vfe&ved=0CAwQjRxqFwoTCMDyrMbz_vcCFQAAAAAdAAAAABAJ )

### @Transactional 적용 메서드의 롤백 처리

- 커밋을 수행하는 주체가 프록시 객체였던 것 처럼 롤백을 처리하는 주체 또한 프록시 객체이다!
- 발생한 익셉션이 RuntimeException 일 때 트랜잭션을 롤백한다.

### 트랜잭션 전파

Propagation 열거 타입 값 목록에서 `REQUIRED` 값

- 메서드를 수행하는 데 트랜잭션이 필요하다는 것을 의미한다.
- 현재 진행중인 트랜잭션이 존재하면 해당 트랜잭션을 사용한다.
- 존재하지 않으면 새로운 트랜잭션을 생성한다.

JdbcTemplate 은 트랜잭션이 진행 중이면 트랜잭션 범위에서 쿼리를 실행한다.

# 10장. 스프링 MVC 프레임워크 동작 방식

### 스프링 MVC 핵심 구성 요소

`**DispatcherServlet**` 이 모든 연결을 담당한다.

- 웹 브라우저로부터 요청이 들어오면 DispatcherServlet 은 그 요청을 처리하기 위한 컨트롤러 객체를 검색한다.
- 이 때 DispatcherServlet 은 직접 컨트롤러를 검색하는 것이 아니라 HandlerMapping 이라는 빈 객체에게 컨트롤러 검색을 요청한다.
- HandlerMapping 은 클라이언트의 요청 경로를 이용해서 이를 처리할 컨트롤러 빈 객체를 DispatcherServlet 에 전달한다.
    - 컨트롤러 객체를 DispatcherServlet 이 전달받았다고 해서 바로 컨트롤러 객체의 메서드를 실행할 수 있는 것은 아니다.
    - @Controller를 이용해서 구현한 컨트롤러, Controller 인터페이스를 구현한 클래스, HttpRequestHandler 인터페이스를 구현한 클래스를 동일한 방식으로 실행할 수 있도록 만들어졌다.

@Controller, Controller 인터페이스, HttpRequestHandler 를 동일한 방식으로 처리하기 위해 중간에 사요되는 것이 **`HandlerAdapter`** 빈이다.

- DispatchServlet 은 HandlerMapping 이 찾아준 컨트롤러 객체를 처리할 수 있는 HandlerAdapter 빈에게 **요청 처리를 위임한다.**
- HandlerAdapter 는 컨트롤러의 알맞은 메서드를 호출해서 요청을 처리하고 그 결과를 DispatchServlet 에 리턴한다.
- 이때 HandlerAdapter 는 컨트롤러의 처리 결과를 ModelAndView 라는 객체로 변환해서 DispatchServlet 에 리턴한다

HandlerAdapter 로부터 컨트롤러의 요청 처리 결과를 ModelAndView 로 받으면 DispatchServlet 은 결과를 보여줄 뷰를 찾기 위해 `**ViewResolver**` 빈 객체를 사용한다.

- ModelAndView 는 컨트롤러가 리턴한 뷰 이름을 담고 있는데 ViewResolver 는 이 뷰 이름에 해당하는 View 객체를 찾거나 생성해서 리턴한다.

→ DispatchServlet 을 중심으로 HandlerMapping, HandlerAdapter, 컨트롤러, ViewResolver, View, JSP 가 각자 역할을 수행해서 클라이언트의 요청을 처리한다. 하나라도 어긋나면 요청을 처리할 수 없게 되므로 각 구성 요소를 바르게 설정하는 것이 중요하다!

### 컨트롤러와 핸들러

- 컨트롤러 : 클라이언트 요청을 실제로 처리하는 것
- DispatchServlet : 클라이언트의 요청을 전달받는 창구 역할

- DispatchServlet 은 클라이언트의 요청을 처리할 컨트롤러를 찾기 위해 `HandlerMapping` 을 사용한다.
- 스프링 MVC 는 웹 요청을 처리할 수 있는 범용 프레임워크이다.
- DispatchServlet 입장에서는 클라이언트 요청을 처리하는 객체의 타입이 반드시 @Controller 를 적용한 클래스일 필요는 없다.
- 스프링 MVC 는 웹 요청을 실제로 처리하는 객체를 `핸들러`라고 부른다.
- @Controller 적용 객체나 Controller 인터페이스를 구현한 객체는 모두 스프링 MVC 입장에서는 핸들러가 된다.
