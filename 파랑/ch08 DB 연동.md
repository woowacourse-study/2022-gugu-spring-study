# Chapter 8 DB 연동

## JdbcTemplate

- DB 연동시 구조적인 반복을 줄이기 위해 템플릿 메서드 패턴과 전략 패턴을 엮어 만든 클래스
- 트랜잭션 관리가 쉽다 → 적용하고 싶은 메서드에 `@Transactional` 을 붙이기만 하면 된다.

### query

- select 쿼리 실행을 위한 메서드
- sql 파라미터로 전달받은 쿼리를 실행하고, RowMapper를 이용해서 결과를 자바 객체로 변환한다
- 쿼리를 실행한 결과가 존재하지 않으면 길이가 0인 리스트를 반환한다

### queryForObject

- 실행 결과가 1행인 경우 사용
- 실행 결과 칼럼이 2개 이상이면 RowMapper를 사용하여 자바 객체로 변환할 수 있다
- 쿼리를 실행한 결과 행의 개수가 0이거나 두 개 이상이면 `IncorrectResultSizeDataAccessException` 이 발생한다
- 쿼리를 실행한 결과 행의 개수가 0이면 하위 클래스인 `EmptyResultDataAccessException` 이 발생한다.

> `EmptyResultDataAccessException` 이 `IncorrectResultSizeDataAccessException` 을 상속하므로 행의 개수가 0인 경우 `EmptyResultDataAccessException` 가 발생한다.

### update

- INSERT, UPDATE, DELETE 시 사용한다
- 쿼리 실행 결과로 변경된 행의 개수를 반환한다

### PreparedStatementCreator

- set 메서드를 사용해서 직접 인덱스 파라미터의 값을 설정해야할 때 사용한다

> 일반적으로는 값을 바로 넣어주지만, 대표적으로 keyHolder를 만들 때 creator를 써야 한다. 


```java
// 기존
jdbcTemplate.update(
        "update MEMBER set NAME=?, PASSWORD=? where EMAIL=?",
        member.getName(), memeber.getPassword(), member.getEmail());

// PreparedStatementCreator 사용
jdbcTemplate.update(new PreparedStatementCreator() {
    @Override
	public PreparedStatement createPreparedStatement(Connection con) throws SQLException {
		PreparedStatement pstmt = con.preparedStatement(
			"insert into MEMBER (EMAIL, PASSWORD, NAME, REGDATE) values (?,?,?,?)");
			pstmt.setString(1, member.getEmail());
			pstmt.setString(2, member.getPassword());
			pstmt.setString(3, member.getName());
			pstmt.setTimestamp(4, Timestamp.valueOf(member.getRegisterDateTime()));
		return pstmt;
	}
});
```

### KeyHolder

- INSERT 쿼리 실행 후 자동으로 생성된 키값을 구할 때 사용한다

## 스프링의 예외 변환

JDBC API를 사용하는 과정에서 `SQLException`이 발생하면 `DataAccessException` 으로 변환해서 예외를 발생시킨다. 예를 들어 MySQL용 JDBC 드라이버는 SQL 문법이 잘못된 경우 `SQLException`을 상속받은 `MySQLSyntaxErrorException` 을 발생시키는데, JdbcTemplate는 이 예외를 `DataAccessException` 을 상속받은 `BadSqlGrammarException` 으로 변환한다.

- 예외를 변환하는 이유: 연동 기술에 상관없이 동일하게 예외를 처리하기 위해서

## 트랜잭션

- 두 개 이상의 쿼리를 한 작업으로 실행해야 할 때 사용한다
- **롤백**(rollback): 한 트랜잭션으로 묶인 퀴리 중 하나라도 실패하면 전체 쿼리를 실패로 간주하고 실패 이전에 실행한 쿼리를 취소한다.
- **커밋**(commit): 한 트랜잭션으로 묶인 쿼리들이 모두 성공해서 결과를 실제 DB에 반영한다.

### @Transactional

- 트랜잭션 범위에서 실행하고 싶은 메서드에 붙이면 쉽게 범위를 지정할 수 있다.
- `@Transactional` 이 제대로 동작하기 위한 두 가지
    - 플랫폼 트랜잭션 매니저 빈 설정
    - `Transactional` 어노테이션 활성화 설정 → `@EnableTransactionManagement`
      - `springboot에서는 어노테이션 필요 없음` ????

    ```java
    @Configuration
    @EnableTransactionManagement
    public class AppCtx {
    
        ...
    	
        @Bean
        public platformTransactionalManage transactionMangager() {
            DataSourceTransactionManager tm = new DataSourceTransactionManager();
            tm.setDataSource(dataSource());
            return tm;
        }
    }
    ```

- 트랜잭션을 처리하기 위해 내부적으로 AOP(프록시)를 사용한다.
- 프록시 객체가 커밋과 롤백을 처리한다.

### 트랜잭션 전파

- 기본적으로 현재 진행 중인 트랜잭션이 존재하면 해당 트랜잭션을 사용한다. 존재하지 않으면 새로운 트랜잭션을 생성한다.
- 트랜잭션이 적용된 메서드 A에서 트랜잭션이 적용된 메서드 B를 호출할 경우
    - A에서 B를 호출한 경우 이미 A에서 시작된 트랜잭션이 존재하므로 B를 호출한 시점에 트랜잭션을 새로 생성하지 않고 존재하는 트랜잭션을 그대로 사용한다. 즉, A와 B메서드를 한 트랜잭션으로 묶어 실행하는 것이다.
- propagation 속성값이 `REQUIRES_NEW` 인 경우
    - 기존 트랜잭션 값이 존재하는지와 상관 없이 항상 새로운 트랜잭션을 생성한다.
        -> `그럼 B에서 B에서 롤백되면 B만 롤백? 아님 A까지 롤백?`
- 트랜잭션이 적용된 메서드 A에서 트랜잭션이 적용되지 않은 메서드 B를 호출할 경우
    - JdbcTemplate는 진행 중인 트랜잭션이 존재하면 해당 트랜잭션 범위에서 쿼리를 실행하므로 하나의 트랜잭션 범위에서 A와 B가 실행된다.
