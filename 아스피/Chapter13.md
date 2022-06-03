# Chp 13. MVC 3: 세션, 인터셉터, 쿠키

---

## 로그인 상태를 유지하는 두가지 방법

1. HttpSession
    - 요청 매핑 애노테이션에 HttpSession을 파라미터로 받는 방법과 HttpServletRequest를 받아서 getSession()을 이용한다.
    - session.setAttribute()로 인증정보 객체를 저장한다.
    - 서버가 꺼지면 모든 것이 초기화된다.

2. 쿠키
    - @CookieValue 를 이용한다. ```@CookieValue(value = "Remember", required = false) Cookie cookie```
      를 파라미터로 넘겨 받으면 value는 쿠키의 이름이다.
    - 로그인 시에 쿠키를 만들어주는 부분은 ```HttpServletResponse.addCookie(new Cookie("REMEMBER),email)```처럼 저장해서 넘긴다.

---

## 인터셉터 이용하기

- 기능마다 로그인 여부를 확인하는 코드가 존재하면 너무 많은 중복이 일어난다. 이때, 사용할 수 있는 것이 HandlerInterceptor 이다.
- HanderInterceptor의 구현체를 어디에 적용할지를 위해 WebMvcConfigurer에 addInterceptors()를 이용한다.<br>

```java
    public void addInterceptors(InterceptorRegister registry){
    registry.addInterceptor(authCheckInterCeptor())
    .addPathPatterns("/edit/**");
    }
```

