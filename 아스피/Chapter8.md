# Chp 8. DB 연동

---

1. JDBC 프로그래밍을 할 떄 구조적으로 사실상 데이터 처리와 관련없는 코드가 반복된다. 이를 해결하기 위해 스프링이 JdbcTemplate 클래스를 제공한다.<br>


2. JDBC API는 DriverManager외에 DataSource를 이용해서 DB 연결을 구하는 방법을 정의한다.<br>
   스프링은 DataSource를 스프링 빈으로 등록하고 DB 연동 기능을 구현한 객체는 DataSource를 주입받아 사용하게 한다.<br>


3. 커넥션을 매번 생성하여 사용하면 연결 시간이 소모되므로 커넥션 풀에 미리 커넥션들을 만들어두고 필요할 때 꺼내쓴다.<br>


4. JdbcTemplate 클래스
    - select<br>
        - query(): sql파라미터로 전달받은 쿼리를 실행하고 RowMapper 클래스를 이용해 ResultSet의 결과를 자바 객체로 변환한다.<br>
        - queryForObject(): 결과가 1행인 경우 사용할 수 있다.
            - 만약 결과가 0개일 경우: EmptyResultDataAccessException (IncorrectResultSizeDateAccessException을 상속)
            - 만약 결과가 2개 이상일 경우: IncorrectResultSizeDateAccessException

    - insert, update, delete<br>
      update() 메서드를 사용하고 쿼리 실행 결과로 변경된 행의 갯수를 리턴한다.
        - Insert시에 Auto Increment의 키를 가지고 오고 싶을 경우, KeyHolder를 이용해서 가져온다.


5. DB 연동 과정에서 발생 가능한 익셉션
    - DataSource를 잘못 설정한 경우: CannotGetJdbcConnectionException
    - 쿼리문에 실수를 하는 경우: BadSqlGrammerException


6. 이러한 익셉션들은 DataAccessException을 상속받은 하위 타입이다. JDBC는 SQLException들을 RuntimeException인 DateAccessExcpetion으로 변환해준다.<br>
   이 과정이 AOP를 사용한 것인지 찾아보니 update()같은 메서드의 내부에 들어가면 <br>
   ```text
   update() -> execute() -> translateException() ->getExceptionTranslator()으로 SQLExceptionTranslator를 받아서 translate()하면 DataAccessException
   ```
   가 나옴


7. 트랜잭션
    - 두 개 이상의 쿼리를 한 작업으로 실행할 때, 뒤의 쿼리에서 오류가 발생하면 앞의 쿼리의 실행도 롤백이 되어야한다. 이것을 위해 전체 쿼리를 하나의 작업으로 묶어준다.<br>
      이것이 트랜잭션이다. 트랜잭션으로 묶인 모든 쿼리가 성공하면 DB에 반영하는데 이것을 커밋이라한다. 반대로, 하나라도 실패한다면 전채를 취소하는데 이것을 롤백이라한다.<br>
      <br>
    - @Transtional 애노테이션을 이용한 트랜잭션 처리는 별도의 설정을 추가하지 않으면 RuntimeException 일 때, 트랜잭션을 롤백한다.<br>
      만약, 다른 Exception이 발생하는 경우에도, rollback을 하고 싶다면 ```@Transational(rollbackFor = SQLException``` 처럼 속성을 이용한다.<br>


8. 트랜잭션 전파
    - 트랜잭션내부 메서드 중에 또 트랜잭션이 있는 경우가 있으면, @Transactional의 propagation 속성값으로 판단한다.<br>
      기본값인 Propagation.REQUIRED는 현재 진행 중인 트랜잭션이 존재하면 그걸 쓰고 아니면 새로 생성한다.
     

