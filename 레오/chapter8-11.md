# 스프링 프로그래밍 입문

## 8장 DB연동

### DataSource Configurations

스프링이 제공하는 DB 연동 기능은 DataSource를 사용해서 DB Connection을 구한다. DB연동에 사용할 DataSource를 스프링 빈으로 등록하고 DB연동 기능을 구현한 Bean 객체는 DataSource를 주입받아 사용한다. Tomcat JDBC 모듈의 DataSource 클래스를 Spring Bean으로 등록해서 DataSource로 사용할 수 있다.

### Connections’ States

커넥션 풀은 커넥션을 생성하고 유지한다. 커넥션 풀에 커넥션을 요청하면 해당 커넥션은 active(활성) 상태가 되고, 커넥션을 다시 커넥션 풀에 반환하면 idle(유휴) 상태가 된다. 예를 들어 DataSource.getConnection을 실행하면 커넥션 풀에서 커넥션을 가져와 커넥션이 활성 상태가 된다. 커넥션을 close하면 커넥션은 풀로 들어가 유휴 상태가 된다. 커넥션 풀을 사용하는 이유는 성능 때문이다. 커넥션 풀에 생성된 커넥션은 지속적으로 재사용된다. 데이터베이스 설정에 따라 일정 시간 내에 쿼리를 실행하지 않으면 연결을 끊기도 한다. 이때 해당 커넥션의 연결을 끊지만 커넥션은 풀 속에 남아있고 이 상태에서 해당 커넥션을 가져오면 예외가 발생한다.

### Tomcat JDBC DataSource Properties

- setInitalSize(int): 커넥션 풀 초기화 시 생성한 초기 커넥션 개수를 지정
- setMaxActive(int): 커넥션 풀에서 가져올 수 있는 최대 커넥션 개수를 지정
- setMaxIdle/setMinidle(int): 커넥션 풀에서 커넥션을 가져올 때 대기할 최대/최소 시간을 밀리초 단위로 지정
- setMaxWait(int): 커넥션 풀에서 커넥션을 가져올 때 대기할 최대 시간을 밀리초 단위로 지정
- setMaxAge(long): 최초 커녁션 연결 후 커넥션의 최대 유효 시간을 밀리초 단위로 지정
- setValidationQuery(String): 커넥션이 유효한지 검사할 때 사용할 쿼리를 지정. null은 검사를 하지 않음
- setValidationQueryTimeout(int): 검사 쿼리의 최대 실행 시간을 초 단위로 지정. 0이하는 비활성화
- setTestOnBorrow(boolean): 풀에서 커넥션을 가져올 때 검사 여부를 지정
- setTestOnReturn(boolean): 풀로 커넥션을 반환할 때 검사 여부를 지정
- setTestWhileIdle(boolean): 커넥션이 풀에 유휴 상태로 유지할 최소 시간을 밀리초 단위로 지정
- setMinEvictableIdleTimeMillis(int): 커넥션 풀에 유휴 상태로 유지할 최소 시간을 밀리초 단위로 지정
- setTimneBetWeenEvictionRunsMillis(int): 커넥션 풀의 유휴 커넥션을 검사할 주기를 밀리초 단위로 지정. 1초 이하로 설정 할 수 없음

### JDBC 프로그래밍의 단점을 보완하는 스프링

JDBC API를 이용하면 DB연동을 위한 Connection을 구하고 PreparedStatement를 생성한다. 이후 finally 블록에서 자원을 닫는다. SQLexception은  checked exception이기 때문에 catch로 처리해줘야 한다. Finally와 Catch문은 사실상 데이터 처리와는 상관없는 코드지만 JDBC 프로그래밍을 할 때 구조적으로 반복된다. 구조적인 반복을 줄이려면 템플릿 메서드 패턴과 전략 패턴을 함께 사용하는 JdbcTemplate을 이용하면 된다.

### JdbcTemplate

스프링이 제공하는 DB접근을 위한 클래스이다. 반복되는 코드 길이를 효과적으로 줄일 수 있다. 트랜잭션 관리가 쉽다는 특징도 있다. @Transctional을 붙이면 커밋과 롤백 처리를 스프링이 알아서 처리한다. 스프링을 사용하면 DataSource나 Connection, Statemement, ResultSet을 직접 사용하지 않고 JdbcTemplate을 이용해서 편리하게 쿼리를 실행할 수 있다.

### RowMapper

ResultSet에서 데이터를 읽어와 객체로 변환해주는 기능을 제공한다.

### query() vs queryForObject()

SELECT 쿼리로 데이터를 조회하려면 query메서드를 사용한다. 결과가 없으면 빈 리스트를 반환한다. 결과가 하나의 레코드일 경우 queryForObject를 사용할 수 있다. 쿼리 실행 결과 행이 없거나 두 개 이상이면 IncorrectResultSizeDataAccessException이 발생한다. 행의 개수가 0이면 하위 클래스인 EmptyResultDataAccessException이 발생한다.

### update()

INSERT, UPDATE, DELETE 쿼리는 update()를 사용한다. 쿼리 실행 결과로 변경된 행의 개수를 반환한다.

### @Transactional 트랜잭션 처리

두 개 이상의 쿼리를 한 작업으로 실행해야 할 때 사용한다. 트랜잭션은 여러 쿼리를 논리적으로 하나의 작업으로 묶고 쿼리 중 하나라도 실패하면 전체 쿼리를 실패로 간주하고 실패 이전에 실행한 쿼리를 취소한다. 스프링이 제공하는 @Transactional 애노테이션을 사용하여 실행하고 싶은 메서드를 트랜잭션 범위로 지정할 수 있다.

@Transactional을 사용한 트랜잭션 처리는 AOP이다. AOP는 프록시를 통해서 구현된다. 따라서 트랜잭션은 프록시를 통해서 구현된다. 트랜잭션은 전파되는 특징이 있다. 한번 트랜잭션 범위에 들어오면 처음으로 트랜잭션 시작지점이 적용된다.

## 9장 스프링 MVC 시작하기 / 10장 MVC 프레임워크 동작 방식

### 스프링 MVC 핵심 구성 요소

스프링 MVC 프레임워크는 DispatcherServlet을 중심으로 HandlerMapping, HandlerAdapter, 컨트롤러, ViewResolver, View, JSP가 각자 역할을 수행해 요청을 처리한다.

DispatcherServlet은 중앙에 위치해 있고 모든 연결을 담당한다. 웹 브라우저로부터 요청이 들어오면 HandlerMapping이라는 Bean 객체에게 컨트롤러 검색을 요청한다.  HandlerMapping은 컨트롤러 Bean 객체를 검색해 DispatcherServlet에 전달한다. 이때 클라이언트의 요청결과를 이용한다.

HandlerAdapter는 Handler Bean에게 컨트롤러의 알맞은 메서드를 호출해서 요청을 처리하고 그 결과를 리턴하여 DispatcherServlet에 반환한다.

ViewResolver Bean 객체는 HandlerAdapter로부터 컨트롤러의 요청 처리 결과를 ModelAndView로 받은 뷰 이름에 해당하는 View 객체를 찾거나 생성해서 반환한다.

DispatcherServlet은 ViewResolver가 반환한 View 객체에게 응답 결과 생성을 요청한다. JSP를 사용하는 경우 View 객체는 JSP를 실행함으로써 웹 브라우저에 전송할 응답을 생성한다.

### 컨트롤러

스프링 MVC 프레임워크에서 컨트롤러란 웹 요청을 처리하고 그 결과를 뷰에 전달하는 스프링 빈 객체이다. 클래스에 @Controller 어노테이션을 붙이고 클래스 또는 메서드에 @RequestMapping을 사용하여 처리할 경로를 지정한다.

클라이언트의 요청을 실제로 처리하는 것은 컨트롤러이지만 DispatcherServlet은 HandlerMapping을 이용해서 컨트롤러를 찾아준다. 이 때 HandlerMapping은 ControllerMapping 타입이 아니라 HandlerMapping이다.

스프링 MVC는 범용 프레임워크여서 @Controller 어노테이션을 붙인 클래스 외에 HttpRequestHandler와 같이 다른 타입으로도 클라이언트의 요청을 처리할 수 있다. 이런이유로 스프링 MVC가 웹 요청을 실제로 처리하는 객체를 핸들러라고 부른다. HandlerAdapter는 실행결과를DispatcherServlet이 받는 ModelAndView로 불리는 타입으로 바꾸어준다.

## 11장 요청 매칭, 커맨드 객체, 리다이렉트, 폼 태그, 모델

### 요청 매핑 어노테이션을 이용한 경로 매핑

관련한 요청 경로를 한 개의 컨트롤러 클래스에서 처리하여 코드를 관리한다. @RequestMapping 어노테이션은 클래스나 메서드에 적용하여 경로를 설정할 수 있다.

### @GetMapping @PostMapping

스프링 MVC는 별도 설정이 없으면 Get과 Post 방식에 상관없이 @RequestMapping에 지정한 경로와 일치하는 요청을 처리한다. 원하는 방식으로 설정하려면 @GetMapping @PostMapping과 같이 원하는 방식의 어노테이션을 사용한다.

### 요청 파라미터 접근

1. HttpServletRequest을 메서드의 파라미터로 사용하고 getParameter()를 이용해서 파라미터의 값을 구한다.
2. @RequestParam 어노테이션을 사용한다.
    - value: Http 요청 파라미터의 이름을 지정한다. 기본값은 객체이름이다.
    - required: 필수 여부를 지정한다. 기본값은 True이다.
    - defaultValue: 요청 파라미터가 값이 없을 때 사용할 문자열 값을 지정한다. (String 타입)
3. 커맨드 객체를 사용한다.
    - 스프링 MVC가 Request 객체를 생성하고 그 객체의 세터 메서드를 이용해서 일치하는 요청 파라미터의 값을 전달한다.

### 리다이렉트

컨트롤러 메서드의 반환 타입으로 String을 사용하고 “**redirect: 주소”**를 반환하면 된다.

### @ModelAttribute 어노테이션으로 커맨드 객체 속성 이름 변경

커맨드 객체에 접근할 때 사용할 속성 이름을 변경할 수 있다.

### Model을 통해 컨트롤러에서 뷰에 데이터 전달하기

컨트롤러는 뷰가 응답 화면을 구성하는데 필요한 데이터를 생성해서 전달해야 한다. 컨트롤러는 요청 매핑 어노테이션이 적용된 메서드의 파라미터로 Model로 추가하는 역할을 한다. 또 Model 파라미터의 addAttribute() 메서드로 뷰에서 사용할 데이터 전달하는 역할도 한다. ModelAndView를 사용하면 모델과 뷰 이름을 함께 제공한다.
