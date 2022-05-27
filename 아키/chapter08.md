# DB 연동

## JDBC 프로그래밍의 단점을 보완하는 스프링 JdbcTemplate

> **JDBC의 단점**  
> 반복되는 코드가 많다. DB 연동에 필요한 Connection을 구한 다음 쿼리를 실행하기 위한 PreparedStatement, 결과를 가져오기 위한 ResultSet을 생성하고 닫아줘야 한다.

Jdbc API를 이용한 DB 연동 코드

```java
@Override
public Member save(Member member) {
	String sql = "insert into member(name) values(?)";
	Connection conn = null;
	PreparedStatement pstmt = null;
	ResultSet rs = null;
	try {
		conn = getConnection();
		pstmt = conn.prepareStatement(sql, Statement.RETURN_GENERATED_KEYS);
		pstmt.setString(1, member.getName());
		pstmt.executeUpdate();
		rs = pstmt.getGeneratedKeys();
		if (rs.next()) {
			member.setId(rs.getLong(1));
		} else {
			throw new SQLException("id 조회 실패");
		}
		return member;
	} catch (Exception e) {
		throw new IllegalStateException(e);
	} finally {
		try {
			if (rs != null) {
				rs.close();
			}
		} catch (SQLException e) {
			e.printStackTrace();
		}
		try {
			if (pstmt != null) {
				pstmt.close();
			}
		} catch (SQLException e) {
			e.printStackTrace();
		}
		try {
			if (conn != null) {
				close(conn);
			}
		} catch (SQLException e) {
			e.printStackTrace();
		}
	}
}
```

스프링 JdbcTemplate의 장점
1. 중복되는 코드가 줄어든다.
```java
@Override
public Station save(Station station) {
	final String sql = "INSERT INTO station (name) VALUES (?)";
	final KeyHolder keyHolder = new GeneratedKeyHolder();
	jdbcTemplate.update(connection -> {
		PreparedStatement ps = connection.prepareStatement(sql, new String[]{"id"});
		ps.setString(1, station.getName());
		return ps;
	}, keyHolder);

	final Long newStationId = keyHolder.getKey().longValue();

	return new Station(newStationId, station.getName());
}
```

2. 트랜잭션 관리가 쉽다.  

PreparedStatement의 특성상, 자동으로 commit이 이루어 진다. 연결이 AutoCommit이면 해당 연결의 모든 SQL 문이 개별 트랜잭션으로 실행되고 커밋된다.

한 메서드 안에서 두개 이상의 sql문을 실행하고자 할 때, 하나라도 제대로 수행되지 않으면 rollback을 하고싶은 경우 Connection의 setAutoCommit(false) 설정을 통해 AutoCommit을 방지할 수 있다.

이처럼 JDBC API로 트랜잭션을 처리하면 다음과 같이 커밋과 롤백처리를 신경써야 하지만, 스프링을 사용하면 `@Transactional `애노테이션만 붙이면 커밋과 롤백처리를 대신 해준다.

```java
@Override
public Member save(Member member) {
	Connection conn = null;
	PreparedStatement pstmt = null;
	try {
		conn = getConnection();
		conn.setAutoCommit(false);
		//DB 쿼리 실행
	} catch (Exception e) {
		throw new IllegalStateException(e);
	} finally {
		//생략
	}
}
```
```java
@Transactional
public Member save(Member member) {
}
```

## DataSource 설정
JDBC API가 DB 연결을 구하는 방법에는 2가지가 있다.
1. DriverManager를 이용
2. DataSource를 이용

> 궁금한점  
> DriverManager를 이용하는 것과 DataSource를 이용하는 것의 차이점이 뭘까? 커넥션풀?
```java
//1번
connection = DriverManager.getConnection("jdbc:mysql://localhost/order_mgmt", "root", "1234");

//2번
connection = dataSource.getConnection();
```

### DataSource 빈 등록 및 주입
스프링에서는 DataSource를 사용해서 DB connection을 구한다. DB 연동에 사용할 DataSource를 빈으로 등록하고 주입받아 사용한다. AppCtx 설정을 통해 DataSource을 빈으로 등록할 수 있다.

```java
@Configuration
public class AppCtx {

    @Bean(destroyMethod = "close")
    public DataSource dataSource() {
        DataSource ds = new DataSource();
        ds.setDriverClassName("com.mysql.jdbc.Driver");
        ds.setUrl("jdbc:mysql://localhost/spring5fs?characterEncoding=utf8");
        ds.setUsername("spring5");
        ds.setPassword("spring5");
        ds.setInitialSize(2);
        ds.setMaxActive(10);
        return ds;
    }
}
```

우리 프로젝트에선 AppCtx 대신 `application.yml` 설정을 통해 자동으로 DataSource를 빈을 생성할 수 있었다.

```
spring:
  datasource: # datasource 설정
    driver-class-name: org.h2.Driver
    url: jdbc:h2:mem:testdb
    username: sa
    password:
```

### DataSource 클래스
DataSource 클래스는 커넥션 풀과 관련한 기능을 제공한다.  
  
커넥션 풀은 커넥션을 생성하고 유지한다. 커넥션 풀에 커넥션을 요청하면 해당 커넥션은 활성(active) 상태가 되고, 커넥션을 다시 커넥션 풀에 반환하면 유휴(idle) 상태가 된다.

> 커넥션 풀을 사용하는 이유는 성능 때문이다. 매번 커넥션을 생성하면 그때마다 연결 시간이 소모되기 때문에, 미리 커넥션을 생성해두고 필요할때마다 커넥션을 꺼내 응답 시간을 줄인다. maxActive 메서드를 통해 DataSource의 최대 커넥션 개수를 지정할 수 있다.

## JdbcTemplate을 이용한 쿼리 실행

### 조회 쿼리
JdbcTemplate 클래스는 SELECT 쿼리 실행을 위한 query() 메서드를 제공한다.  
query() 메서드는 sql 파라미터로 전달받은 쿼리를 실행하고 RowMapper를 이용해 ResultSet의 결과를 자바 객체로 변환한다.

```java
public Member selectByEmail(String email) {
	List<Member> results = jdbcTemplate
		.query("select * from MEMBER where EMAIL = ?", new RowMapper<Member>(){
			@Override
			public Member mapRow(ResultSet rs, int rowNum) throws SQLException {
				Member member = new Member(
					rs.getString("EMAIL"),
					rs.getString("PASSWORD"),
					rs.getString("NAME"),
					rs.getTimestamp("REGDATE").toLocalDateTime());
				return member;
			}
		}, email);
	return results.isEmpty() ? null : results.get(0);
}
```

> queryForObject():  
> 결과가 1행인 경우 사용할 수 있다.

```java
public int count() {
	Integer count = jdbcTemplate.queryForObject("select count(*) from MEMBER", Integer.class);
	return count;
}
```

### 변경 쿼리

```java
public void update(Member member) {
	jdbcTemplate.update("update MEMBER set NAME = ?, PASSWORD = ? where EMAIL = ?",
		member.getName(), member.getPassword(), member.getEmail());
}
```

### PreparedStatementCreator를 이용한 쿼리 실행

PreparedStatement의 set 메서드를 이용해서 직접 인덱스 파라미터의 값을 설정할 때도 있다. 이 경우 PreapredStatementCreator를 인자로 받는 메서드를 이용해서 직접 preapredStatement를 생성하고 설정해야 한다.

```java
public void update(Member member) {
	jdbcTemplate.update(new PreparedStatementCreator() {
		@Override
		public PreparedStatement createPreparedStatement(Connection con) throws SQLException {
			// 파라미터로 전달받은 Connection을 이용해서 preparedStatement 생성
			PreparedStatement pstmt = con.prepareStatement(
				"inser into MEMBER (EMAIL, PASSWORD, NAME, REGDATE) values (?, ?, ?, ?)");
			pstmt.setString(1, member.getEmail());
			pstmt.setString(2, member.getPassword());
			pstmt.setString(3, member.getName());
			pstmt.setTimestamp(4, Timestamp.valueOf(member.getRegisterDateTime()));
			// 생성한 preparedStatement 객체 리턴
			return pstmt;
		}
	});
```

### INSERT 쿼리 실행 시 KeyHolder를 이용해서 자동 생성 키값 구하기

KeyHolder를 통해 자동으로 생성된 키 값을 구할 수 있다.

```java
public void insert(Member member) {
	KeyHolder keyHolder = new GeneratedKeyHolder();
	jdbcTemplate.update(new PreparedStatementCreator() {
		@Override
		public PreparedStatement createPreparedStatement(Connection con) throws SQLException {
			PreparedStatement pstmt = con.prepareStatement(
				"insert into MEMBER (EMAIL, PASSWORD, NAME, REGDATE)" +
					"values (?, ?, ?, ?)", new String[]{"ID"});
			pstmt.setString(1, member.getEmail());
			pstmt.setString(2, member.getPassword());
			pstmt.setString(3, member.getName());
			pstmt.setTimestamp(4, Timestamp.valueOf(member.getRegisterDateTime()));
			return pstmt;
		}
	}, keyHolder);
	Number keyValue = keyHolder.getKey();
	member.setId(keyValue.longValue());
}
```

## 스프링의 익셉션 변환 처리

SQL 문법이 잘못됐을 때 발생한 메세지를 보면 익셉션 클래스가 org.spring.framework.jdbc 패키지에 속한 BadSqlGrammarException 클래스임을 알 수 있다.  

JDBC API를 사용하는 과정에서 SQLException이 발생하면 이 익셉션을 알맞은 DataAccessException으로 변환해서 발생한다.  

예를 들어 MySQL용 JDBC 드라이버는 SQL 문법이 잘못된 경우 SQLException을 상속받은 MySQLSyntaxErrorException을 발생시키는데 JdbcTemplate은 이 익셉션을 DataAccessException을 상속받은 BadSqlGrammarExcepation으로 변환한다.  

> 스프링은 왜 SQLException을 그대로 전파하지 않고 SQLException을 DataAccessException으로 변환할까?  
> 연동하는 DB에 상관없이 **동일하게 익셉션을 처리**할 수 있도록 하기 위함이다. 덕분에 개발자는 연결하는 DB마다 각각 예외를 다르게 처리할 수고로움이 사라졌다.

## 트랜잭션 처리

트랜잭션(transaction)이란 두 개 이상의 쿼리를 한 작업으로 실행해야 할 때 사용하는 것이다. 트랜잭션은 여러 쿼리를 논리적으로 하나의 작업으로 묶어준다. 한 트랜잭션으로 묶인 쿼리 중 하나라도 실패하면 전체 쿼리를 실패로 간주하고 실패 이전에 실행한 쿼리를 취소한다.  

쿼리 실행 결과를 취소하고 DB를 기존 상태로 되돌리는 것을 `롤백(rollback)`이라고 부른다. 반면에 트랜잭션으로 묶인 모든 쿼리가 성공해서 쿼리 결과를 DB에 실제로 반영하는 것을 `커밋(commit)`이라고 한다.  

JDBC API는 커넥션이 생성되면 바로 Auto Commit 모드로 세팅된다. 그래서 sql문마다 바로바로 커밋이 된다. 그래서 Connection의 setAutoCommit(false)를 통해 Auto Commit을 취소하고
쿼리문마다 커밋과 롤백을 직접 실행해야 한다.

```java
Connection conn = null;
try{
	...
	conn.setAutoCommit(false); // 트랜잭션 범위 시작
	... 쿼리실행
	conn.commit(); // 트랜잭션 범위 종료: 커밋
}
catch(SQLException ex){
	if(conn != null)
		// 트랜잭션 범위 종료: 롤백
		try{ conn.rollback(); } catch (SQLException e){}
}
finally{
	if(conn!= null)
		try{ conn.close(); } catch(SQLException e){}
}
```

### @Transactional을 이용한 트랜잭션 처리
스프링이 제공하는 `@Transactional` 애노테이션을 사용하면 트랜잭션 범위를 매우 쉽게 지정할 수 있다. 트랜잭션을 실행하고 싶은 메서드에 `@Transactional` 애노테이션을 붙이기만 하면 된다.

```java
@Transactional
public void changePassword(String email, String oldPwd, String newPwd) {
	Member member = memberDao.selectByEmail(email);
	if (member == null)
		throw new MemberNotFoundException();

	member.changePassword(oldPwd, newPwd);

	memberDao.update(member);
}
```

changePassword 메서드는 하나의 트랜잭션으로 묶이며, 메서드 내의 쿼리가 하나라도 실패하면 전부 롤백된다.

`@Transactional` 애노테이션이 제대로 동작하려면 다음 두가지 설정을 스프링에 추가해야 한다.
1. PlatformTransactionManager 빈설정
2. @Transcational 애노테이션 활성화 설정( @EnableTransactionManagement)

> 궁금한점 : 
> 위 2가지 설정을 모두 한 적이 없는데 스프링부트에서는 자동으로 설정을 해주는 것일까?


### @Transactional과 프록시

여러 빈 객체에 공통으로 적용되는 기능을 구현하는 방법으로 AOP가 나왔는데, 트랜잭션도 공통 기능 중 하나이다. 스프링은 @Transactional 애노테이션을 이용해서 트랜잭션을 처리하기 위해 내부적으로 AOP를 사용한다

### @Transactional 적용 메서드의 롤백 처리

@Transactional을 처리하기 위한 프록시 객체는 **원본 객체의 메서드를 실행하는 과정에서 RuntimeException이 발생하면 트랜잭션을 롤백**한다.

### @Transactional의 주요 속성
@Transacional에도 속성 값을 줄 수 있다. 그러나 잘 사용하지는 않는다.

### 트랜잭션 전파

@Transactional의 propagation 속성은 기본값이 Propagation.REQUIRED이다. REQUIRED는 현재 진행 중인 트랜잭션이 존재하면 해당 트랜잭션을 사용하고 존재하지 않으면 새로운 트랜잭션을 생성한다.  

만약 propagation 속성값이 REQUIRES_NEW라면 기존 트랜잭션이 존재하는지와 상관없이 새로운 트랜잭션을 생성한다.  

Transaction Propagation의 종류는 다음과 같다. 
- REQUIRED (default) : 이미 시작된 트랜잭션이 있으면 참여하고 없으면 새로 시작한다. 
- REQUIRES_NEW : 항상 새로운 트랜잭션을 시작한다. 이미 진행 중인 트랜잭션이 있으면 트랜잭션을 잠시 보류시킨다.
- SUPPORTS : 이미 시작된 트랜잭션이 있으면 참여하고, 없으면 트랜잭션없이 진행한다.
- NESTED : 중첩된 트랜잭션은 먼저 시작된 부모 트랜잭션의 커밋과 롤백에는 영향을 받지만 자신의 커밋과 롤백은 부모 트랜잭션에게 영향을 주지 않는다. 메인 트랜잭션이 롤백되면 중첩된 로그 트랜잭션도 같이 롤백되지만, 반대로 중첩된 로그 트랜잭션이 롤백돼도 메인 작업에 이상이 없다면 메인 트랜잭션은 정상적으로 커밋된다.
- MANDATORY : REQUIRED와 비슷하게 이미 시작된 트랜잭션이 있으면 참여한다. 반면에 트랜잭션이 시작된 것이 없으면 새로 시작하는 대신 예외를 발생시킨다. 혼자서는 독립적으로 트랜잭션을 진행하면 안 되는 경우에 사용한다.
- NOT_SUPPORTED : 트랜잭션을 사용하지 않게 한다. 이미 진행 중인 트랜잭션이 있으면 보류시킨다.
- NEVER : 트랜잭션을 사용하지 않도록 강제한다. 이미 진행 중인 트랜잭션도 존재하면 안된다 있다면 예외를 발생시킨다.