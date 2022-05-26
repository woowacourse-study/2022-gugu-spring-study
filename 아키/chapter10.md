# 스프링 MVC 프레임워크 동장 방식

## 스프링 MVC 핵심 구성 요소

[](image/10장.jpeg)

1. 웹 브라우저로부터 요청이 전송되면 `DispatcherServlet`이 요청을 받는다.
2. `DispatcherServlet`은 `HandlerMapping`에 컨트롤러 검색을 요청한다.
3. `HandlerMapping`이 컨트롤러를 찾으면 `DispatcherServlet`은 `HandlerAdapter` 빈에게 요청 처리를 위임한다.
4. `HandlerAdapter`는 요청에 대응하는 컨트롤러 빈을 찾아 실행하고 결과를 받는다.
5. `HandlerAdapter`는 처리 결과를 ModelAndView로 변환해서 `DispatcherServlet`에게 반환한다.
6. `DispatcherServlet`는 컨트롤러 실행 결과를 보여줄 View를 찾는 과정을 `ViewResolver`에게 위임한다.
7. `DispatcherServlet`은 `ViewResolver`가 찾은 View 객체에 응답 결과를 생성을 요청하고, View는 웹 브라우저에게 최종 결과를 전송한다.  
