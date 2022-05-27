## Get과 Post의 구분
- @RequestMapping 으로 경로 지정을 할 경우, Spring MVC는 별도의 설정이 없으면 Get, Post 방식에 상관없이 지정한 경로와 일치하는 요청들을 처리한다.
- Spring 4.3버전 이후로는 @PostMapping, @GetMapping과 같이 요청 method별로 어노테이션이 있어 요청 제한을 할 수 있다.

## 요청 파라미터의 접근
### HttpServletRequest
- HttpServletRequest를 통해서도 paramter의 값을 가져올 수 있다. 
- HttpServletRequest의 `getParameter()`를 통해 파라미터의 값을 가져오면 된다.
```java
    @RequestMapping("/step2")
    public String handleStep2(HttpServletRequest request) {
        String agreeParam = request.getParameter("agree");
        if (agreeParam == null || !agreeParam.equals("true")) {
            return "register/step1";
        }
        return "register/step2";
    }
```

### @RequestParam
- 요청 파라미터의 개수가 몇 개 안된다면 해당 어노테이션을 통해 파라미터 값을 가져올 수 있다.
- 스프링 MVC는 파라미터 타입에 맞게 값을 변환해준다. (primitive type, wrapper type모두 지원)
- 
```java
    @RequestMapping("/step2")
    public String handleStep2(@RequestParam(value = "agree", defaultValue = "false") boolean agree) {
        if (!agree) {
            return "register/step1";
        }
        return "register/step2";
    }
```
- **속성들**
  - value: HTTP 요청 파라미터의 이름을 지정한다
  - required: 필수 여부를 지정하며 해당 값이 true인데 요청 값이 없다면 예외를 발생한다. 기본값은 true이다.
  - defaultValue: 요청 파라미터가 값이 없을 때 사용할 문자열 값을 지정한다. 기본값은 없다.

### @ModelAttribute (커맨드 객체)
- @ModelAttribute를 사용하면 dto로 사용되는 객체에 request param들을 자동으로 mapping할 수 있다.
- 자동으로 매핑이 되기 때문에 파라미터의 수가 많아질 경우에는 `@RequestParam`보다는 `@ModelAttribute`를 사용하는 것이 좋다.
- 해당 어노테이션은 생략도 가능하다.
- 스프링 MVC는 커맨드 객체가 중첨, 컬렉션 프로퍼티를 가져도 파라미터 값을 알맞게 매핑해주는 기능을 제공하고 있다. (View에서 Form data를 사용하는 방법)
  - Http 요청 파라미터 이름이 `프로퍼티이름[인덱스]`형식이면 List타입의 프로퍼티 값 목록으로 처리한다.
  - Http 요청 파라미터 이름이 `프로퍼티이름.프로퍼티이름`의 형식이면 중첩 프로퍼티 값을 처리한다.

## 컨트롤러의 리다이렉트
- 컨트롤러에서 특정 페이지로 리다이렉트를 시키려면 `"redirect:경로(주소)"`를 뷰의 이름으로 반환하면 된다.
- Spring은 `@RequestMapping`, `@GetMapping` 등의 요청 매핑 관련 어노테이션을 가진 메서드가 `redirect:`로 시작하는 경로로 반환하게 된다면 나머지 경로를 이용해서 경로를 구한다.
  - `/`로 시작한다면 웹 어플리케이션의 기본 주소를 기준으로 경로를 만든다.
  - `/`로 시작하지 않을 경우 현재 경로를 기준으로 상대적인 경로를 만든다.
    - ex) `redirect:step1`: localhost:8080/register/step2 -> localhost:8080/register/step1

## 주요 발생 예외
### 요청 매핑 관련 예외
- 404 (가장 흔하게 발생)
  - 요청 경로가 올바른지
  - 컨트롤러에 설정한 경로가 올바른지
  - 컨트로러 클래스를 빈으로 등록했는지
  - 컨트롤러 클래스에 @Controller 어노테이션을 적용했는지
- 405
  - 지원하지 않는 전송 method를 사용한 경우 발생한다.
  - ex) Post method경로만 존재하는데 해당 주소로 Get request를 보낸 경우
- 400
  - @RequestParam이나 커맨드 객체(ModelAttribute)객체에 필수 입력으로 값을 설정하였는데 요청 값이 없는 경우
  - @RequestParam의 타입으로 특정 값을 매핑할 수 없는 경우
    - ex) "true1"이라는 값을 boolean으로 변환할 수 없다.
    - ex) "abc"를 int 형으로 변환할 수 없다.

## 
