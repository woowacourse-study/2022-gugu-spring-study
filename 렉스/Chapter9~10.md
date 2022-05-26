## Spring MVC의 어노테이션 및 클래스
- `@EnableWebMvc`: 스프링 MVC 설정을 활성화해주어 스프링 MVC를 사용하는데 필요한 다양한 (빈)설정을 생성해준다.
해당 어노테이션을 직접 설정하려면 수십 줄의 코드를 작성해야한다.
- `WebMvcConfigurer`: 스프링 MVC의 개별 설정을 조정할 때 사용한다.
- `Model`: 컨트롤러의 처리 결과를 view로 전송할 때 사용한다.
- `@RequestParam`: http 요청 파라미터 값을 메서드의 파라미터로 전달할 때 사용한다.


컨트롤러란?
- 웹 요청을 처리하고 결과를 뷰에 전달하는 스프링 빈 객체이다

# Chapter 10
![](https://s1.md5.ltd/image/b091f0b66d2dec79cdcbf9d7532a1465.png)

- DispatcherServlet은 모든 연결을 담당한다. 
1. 요청이 들어오면 해당 요청을 처리할 수 있는 HadlerMapping을 통해 요청 처리를 위한 컨트롤러 객체를 스프링 컨테이너에서 검색한다. 
2. HandlerMapping는 해당 요청을 해결할 수 있는 컨트롤러 빈 객체를 스프링 컨테이너에서 찾아 DispatcherServlet에 다시 반환한다.
3. DispatcherServlet이 HandlerMapping에서 찾은 컨트롤러 빈에게 요청 처리를 위임한다.
4. HandlerAdaptor는 컨트롤러의 알맞은 메서드를 호출하여 요청을 처리한다.
5. 만들어진 결과를 ModelAndView로 변환 후에 DispatcherServlet에 반환한다.
6. DispatcherServlet은 결과를 결과로 받은 ModelAndView를 보여줄 View를 찾기위해 스프링 컨테이너에서 ViewResolver 빈 객체를 찾아 사용한다.
7. ViewResolver는 뷰 이름에 해당하는 View객체를 찾거나 생성해서 반환한다.
   - JSP를 사용하는 ViewResolver는 매번 새로운 View객체를 생성하여 DispatcherServlet에 반환한다.
8. ViewResolver가 반환한 View객체에 응답 결과 생성을 요청한다.
   - JSP를 사용하는 경우 View 객체는 JSP를 실행함으로써 웹 브라우저에 전송할 응답 결과를 생성한다.

## Handler
- HandlerApaptor는 @Controller, Controller인터페이스, HttpRequestHandler 인터페이스를 동일한 방식으로 처리하 수 있게 해준다.
- HandlerMapping은 요청을 처리하기 위한 컨트롤러 객체를 검색하여 반환을 해준다. 하지만 왜 ControllerMappping이 아닌 HandlerMapping일까?? 
  - 이는 DispatcherServlet 입장에서 클라이언트 요청을 처리하는 객체의 타입이 @Controller를 적용한 클래스 외에도 HttpRequestHandler이 존재하기 때문이다.
- Handler: 스프링 MVC에서 웹 요청을 실제로 처리하는 객체를 의미한다.
- DispatcherServlet은 HandlerAdator로부터 받아야하는 객체의 타입이 ModelAndView타입이어야 한다. 
  - 핸들러들의 반환값이 ModelAndView가 아닐 수도 있는데 HandlerAdaptro는 핸들러의 결과값들을 ModelAndView로 변환해주는 일도 한다.
- 핸들러 객체의 타입마다 그에 맞는 HandlerMapping과 HandlerAdaptor가 존재하여 핸드러 종류에 따라 각각 스프링 빈으로 등록하여야 한다.
  - 스프링이 제공하는 설정 기능을 사용하면 빈을 수동으로 등록하지 않아도 된다. (바로 다음 설명)

## WebMvcConfigurer, @EnableWebMvc
```java
@Configuration
@EnableWebMvc
public class MvcConfig implements WebMvcConfigurer {

    @Override
    public void configureDefaultServletHandling(DefaultServletHandlerConfigurer configurer) {
        configurer.enable();
    }

    @Override
    public void configureViewResolvers(ViewResolverRegistry registry) {
        registry.jsp("/WEB-INF/view/", ".jsp");
    }
    // configureViewResolvers는 ViewResolver의 설정을 해준다.
}
```
WebMvcConfigurer타입의 빈(클래스)에 `@EnableWebMvc`을 설정클래스에 붙이면 `@Controller` 어노테이션이 붙은 컨트롤러들을 위한 설정을 생성해준다.

WebMvcConfigurer는 스프링 5부터는 자바 8버전부터 지원한 디폴트 메서드를 사용하여 기본적인 설정 구현들이 만들어져있으며 우리가 설정을 변경하고 싶으면 override해주면 된다.


