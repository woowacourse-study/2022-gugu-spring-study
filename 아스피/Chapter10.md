# Chp 10. 스프링 MVC 프레임워크 동작 방식

---

1. 스프링 MVC 핵심 구성 요소
    1. 요청 전송: 웹 브라우저가 요청을 보내면 DispatcherServlet이 해당 요청을 받는다
    2. HandlerMapping에게 요청을 처리할 적합한 컨트롤러를 찾아주길 요청한다.
    3. 찾은 컨트롤러를 실행할 수 있도록 HandlerAdapter에게 처리를 요청한다.
    4. HandlerAdapter는 컨트롤러가 리턴한 결과를 ModelAndView 객체의 형태로 변환해서 반환한다.
    5. 컨트롤러의 실행결과를 보여줄 View를 찾기위해 빈 등록된 ViewResolver를 이용한다.
    6. ViewResolver가 찾거나 생성한 View를 반환받아 응답 결과 생성을 요청하고 View는 웹 브라우저에 전송할 응답을 생성하고 끝이 난다.


2. DispatcherServlet은 전달받은 설정 파일을 통해 스프링 컨테이너를 생성하는데<br>
   ```HandlerMapping, HandlerAdapter, 컨트롤러 빈, ViewResolver```는 해당 컨테이너에서 구한다.


3. @Controller를 위핸 핸들러

``` @EnableWebMvc``` 애노테이션을 추가하면 @Controller 타입의 핸들러 객체를 처리하기 위한 두 클래스도 추가해준다.

4. WebMvcConfigurer 인터페이스와 설정
    - @EnableWebMvc를 사용하면 ViewResolver 값이 자동으로 등록된다. 그런데 WebConfigurer를 구현하면 거기에 사용사가 추가 구현을 할 수 있다.
      configureViewResolver()를 이용한다. 이렇게 되면 따로 Bean 등록을 할 필요가 없다.


5. 디폴트 핸들러와 HandlerMapping의 우선순위
    - '/index.html'과 같이 매핑된 컨트롤러가 없다면 WebMvcConfigurer가 제공하는 configureDefaultServletHandling()를 사용하면 편하다.<br>
    - 메서드에 SimpleUrlHandler는 모든 경록에 대해 디폴트핸들러를 반환한다. 그러면 DispatcherServlet은 처리를 요청한다.
