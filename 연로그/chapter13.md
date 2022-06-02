# Chapter 13 - 세션, 인터셉터, 쿠키

### Session

1. 컨트롤러에서 HttpSession 파라미터 추가

```java
@PostMapping
public void test(LoginRequest loginRequest,Errors errors,HttpSession httpSession){
    // ...
}
```

2. HttpServletRequest 파라미터를 이용해 HttpSession 구하기

```java
@PostMapping
public void test(LoginRequest loginRequest,Errors errors,HttpServletRequest httpServletRequest){
        HttpSession httpSession=httpServletRequest.getSession();
    // ...
}
```

- 세션 이용하기
    - `setAttribute()`를 통해 저장
    - `getAttribute()`를 통해 조회
    - `invalidate()`를 통해 모든 속성 값 제거

### Interceptor

- `HandlerInterceptor`를 구현해 interceptor 생성 가능
  - `preHandle`: 핸들러 실행 전
  - `postHandle`: 핸들러 실행 후, 뷰 실행 전
  - `afterCompletion`: 핸들러 및 뷰 실행 후
- `WebMvcConfigurer`를 구현한 설정 파일에서 설정
  - `addInterceptors()`를 override 해 interceptor 추가

### Cookie

- 쿠키 가져오기

```java
@GetMapping("/find")
public void findCookie(@CookieValue(value = "ID") String value) {
    System.out.println(value);
}
```

```java
RestAssured.given()
        .log().all()
        .contentType(MediaType.APPLICATION_JSON_VALUE)
        .cookie("ID","yeonlog")
        .when()
        .get("/find")
        .then()
        .log().all()
        .extract();
```

- 쿠키 저장하기

```java
@GetMapping("/cookie")
public void addCookie(HttpServletResponse response) {
    response.addCookie(new Cookie("ID", "yeonlog"));
}
```

### 궁금한 점
- 쿠키를 저장해서 언제 쓰는가? 어디서 가져오는가?
- 서버에서 쿠키를 저장할 일이 있는가? 현재 JSP와 연동했기 때문에 사용하는 코드인가?