# Chp 14. MVC 4: 날짜 값 변환, @PathVariable, 익셉션 처리

---

- Data타입 프로퍼티 변환 처리: @DataTimeFormat

- @PathVariable을 이용한 경로 변수 처리<br>
    - ```http://localhost:8080/sp5-chap14/members/10```
      해당 형식의 URL를 쓰면 각 회원마다 경로의 마지막 부분이 달라진다. 이때, 요청 매핑에서는 member/{id}와 같이 받는데 id를 경로 변수라고 한다.<br>

- 컨트롤러 익셉션 처리
    1. 각 컨트롤러마다 핸들링을 하고 싶다면 컨트롤러에 @ExceptionHandler를 적용한 메서드를 이용한다.
    2. 전체적인 컨트롤러에 공통으로 익셉션 처리를 하고 싶다면 @ControllerAdvice를 적용한 클래스를 사용한다.
    3. 만약 컨트롤러와 @ControllerAdvice가 적용된 클래스에 공통 익셉션의 처리가 있다면 컨트롤러 내부의 처리를 따른다.
