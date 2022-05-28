# Chapter 08 - DB 연동

### Java와 DB 연동

> JDBC에 대해서는 [블로그 글](https://yeonyeon.tistory.com/217) 로 대체한다.

1. DB 연동에 필요한 Connection 생성
2. 쿼리를 실행하기 위한 PreparedStatement 생성
3. 2의 실행 결과 반환
4. ResultSet, PreparedStatement, Connection 닫기

```java
class BoardDao {

    public int create(Team team) {
        String sql = "insert into board (turn) values (?)";

        try (PreparedStatement preparedStatement = connection.prepareStatement(sql, Statement.RETURN_GENERATED_KEYS)) {
            preparedStatement.setString(1, team.name());
            preparedStatement.executeUpdate();

            ResultSet resultSet = preparedStatement.getGeneratedKeys();
            return resultSet.getInt(1);

        } catch (SQLException e) {
            e.printStackTrace();
            throw new DataAccessException();
        }
    }
}
```

### Spring과 DB 연동

- 템플릿 메서드 패턴 + 전략 패턴을 도입한 JdbcTemplate 등장
- 중복되는 코드를 제거하고 핵심 로직에 좀 더 집중할 수 있음
- [@Transactional](https://yeonyeon.tistory.com/223) 을 통해 간편한 트랜잭션 관리
- 커넥션 풀 활용
    - : 일정 개수의 DB 커넥션을 미리 만들어두는 기법
    - DB 커넥션을 생성하는 시간 -> 전체 성능에 영향
    - 커넥션 풀에서 커넥션 가져와 사용한 뒤 반납
    - ex: Tomcat JDBC, HikariCP, DBCP, c3p0, ...

```java
class BoardDao {

    public int create(Team team) {
        String sql = "insert into board (turn) values (:turn)";

        MapSqlParameterSource source = new MapSqlParameterSource("turn", team.name());
        KeyHolder keyHolder = new GeneratedKeyHolder();

        jdbcTemplate.update(sql, paramSource, keyHolder);
        return Objects.requireNonNull(keyHolder.getKey()).intValue();
    }
}
```

> Spring의 커넥션 풀 기본 옵션
> - 2.0 이전: Tomcat JDBC
> - 2.0 이후: HikariCP
>
> [HikariCP benchmarking page](https://github.com/brettwooldridge/HikariCP-benchmark) 를 보면 히카리가 월등한 성능을 보인다.

### DataSource

- Connection 생성할 때 이용됨
- 여러가지 옵션
    - `maxActive`: 활성 상태가 가능한 최대 커넥션 수
    - `maxWait`: 커넥션 요청 시 다른 커넥션이 반환될 때까지 대기하는 시간
    - `initialSize`: 최소 커넥션 개수
    - `minEvictableIdleTime`: 최소 유휴 시간
    - `timeBetweenEvictionRunsMillis`: 유휴 커넥션 검사 주기
    - `testWhileIdle`: 유휴 커넥션 검사 여부 (T/F)

### JdbcTemplate

|메서드|설명|반환값|
|---|-----|---|
|query|select 쿼리 실행 (결과 n개)|List<T>|` 
|queryForObject|select 쿼리 실행 (결과 1개)|T|
|update|insert/update/delete 실행 (결과 변경된 행의 개수)|int|

#### KeyHolder

- MySQL의 AUTO_INCREMENT 같은 자동 생성 키 값을 구할 수 있게 도움
- Spring과 DB 연동의 예제 참고

### DB 연동 시 발생 가능한 Exception

> [블로그](https://yeonyeon.tistory.com/175) 로 대체한다.

### Spring의 Exception 변환 처리

- SQL 문법이 잘못됐을 때 `BadSqlGrammarException` 발생
- JdbcTemplate에서는 `DataAccessException`으로 변환  
  👉 연동 기술에 상관없이 동일하게 익셉션을 처리하기 위함

### Transaction 처리

1. 직접 Connection의 commit(), rollback() 호출
2. @Transactional 이용

### @Transactional

> [블로그](https://yeonyeon.tistory.com/223) 참고

- value: 트랜잭션 관리에 사용할 PlatformTransactionManager 빈의 이름 지정
- propagation: 트랜잭션 전파 타입
    - REQUIRED: 기존 트랜잭션 있으면 사용, 없으면 생성
    - MANDATORY: 기존 트랜잭션 있으면 사용, 없으면 예외
    - REQUIRES_NEW: 항상 새 트랜잭션 시작
    - SUPPORTS: 기존 트랜잭션 있으면 사용, 없어도 메서드 실행
    - NOT_SUPPORTED: 기존 트랜잭션 있으면 메서드 실행 동안만 일시 중지
    - NEVER: 트랜잭션 필요 X. 기존 트랜잭션 있으면 익셉션
    - NESTED: 기존 트랜잭션에 중첩된 트랜잭션에서 실행, 없으면 REQUIRED처럼 동작
- isolation: 트랜잭션 격리 레벨
    - DEFAULT: 기본 설정
    - READ_UNCOMMITTED: 커밋 안된 데이터 읽기 가능
    - READ_COMMITTED: 커밋된 데이터만 읽기 가능
    - REPEATABLE_READ: 처음 읽은 데이터와 두번재 읽은 데이터가 동일한 값 찾기
    - SERIALIZABLE: 동일 데이터에 대해 여러 트랜잭션 수행 불가
- timeout: 트랜잭션 제한 시간 지정

### @EnableTransactionManagement

- proxyTargetClass: 프록시 생성 여부 결정 (default = false)
- order: AOP 적용 순서 지정