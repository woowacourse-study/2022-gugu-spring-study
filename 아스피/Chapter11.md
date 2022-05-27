# Chp 11. MVC 1: 요청 매핑, 커멘드 객체, 리다이렉트, 모델

---

1. 요청 매핑 애노테이션을 이용한 경로 매핑<br>

```java
@GetMapping("/resgister/step1")
```

위와 같이 경로를 매핑한다. 경로가 같아도 방식(http 메서드)가 다르면 다른 컨트롤러가 인식한다.

2. 요청 파라미터 접근
    1. HTTPServletRequest를 직접 이용: ex) HTTPServlet.getParameter("agree")
    2. @RequestParam을 이용


3. redirect 처리
    - 잘못된 방식으로 접근하면 떄로는 오류 발생보다 리다이렉트시켜 주는 것이 좋을 떄가 있다.
        - "redirect:" 뒤에 '/'로 시작하는 경우엔 웹 어플리케이션을 기준으로 이동 경로를 설정한다.
        - 그렇지 않으면 현재 요청 경로를 기준으로 상대 경로로 이동 경로를 설정한다.


4. 커맨드 객체를 이용해서 요청 파라미터를 사용하기
    - HTTPServlet.getParameter()가 너무 많아지면 코드 길이가 너무 길어진다. 스프링은 이를 위해 커맨드 객체에 담아주는 기능을 제공한다.<br>
      요청 파라미터의 값을 setter로 전달받을 수 있는 객페를 커맨드 객체로 이용하면 된다.


5. View에서 커맨드 캑체를 이용하기
    - 만약 커맨드 객체가 RegisterRequest라면 View에서 ${registerRequest.name}처럼 접근할 수 있다.
    - 만약 커맨드 객체의 이름을 바꾸고 싶다면 @ModelAttribute("formData")처럼 애노테이션을 붙여주면 formData라는 이름을 사용할 수 있다.


6. 컨트롤러 구현 없는 경로 매핑
    - WebMvcConfigurer의 addViewController()를 이용한다.<br>
      ex) "/main"으로 들어왔는데 컨트롤러가 없을때 setViewName()에 설정한 name으로 뷰 이름 사용


7. 요청 매핑 애노테이션과 관련된 주요 익셉션
    - 404 Error: 요청 경로를 처리할 컨트롤러가 없거나 WebConfigurer를 이용한 설정이 없을때
        - 요청 경로가 올바른지
        - 컨트롤러에 설정한 경로가 올바른지
        - 컨트롤러 클래스를 빈으로 등록했는지
        - @Controller를 적용했는지
        - 리턴한 ViewName을 가지는 View가 없는 경우

    - 405 Error: 요청 경로를 없는 방식(POST,GET 등)으로 요청하는 경우
    - 400 Error:
        - 필수 파라미터가 없는 경우
        - 타입 변환이 이루어질 수 없는 경우


8. 중첩, 콜렉션 프로퍼티
    - 스프링 MVC는 커맨드 객체가 리스트 타입의 프로퍼티를 가졌거나 중첩 프로퍼티를 가진 경우에도 요청 프로퍼티를 알맞게 설정해주는 기능을 제공한다.


9. Model을 이용해서 컨트롤러에서 뷰에 데이터 전달하기
    - Controller를 구현할 때
      ```java
      @RequestMapping
       public String hello(Model model, @RequestParam(value = "name",required = false) String name){
         model.addAttribute("greeting",name);
         return "hello";
      }
      ```
      처럼 구현한다.<br>
      JSP는 ${greeting} 으로 이용한다.


10. ModelAndView 객체를 통해서도 컨트롤러의 결과를 반환할 수 있다.
