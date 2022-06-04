# Chapter 11 MVC 1: 요청 매핑, 커맨드 객체, 리다이렉트, 폼 태그, 모델

### @RequestMapping

```java
@RequestMapping(value = "/hello", method = RequestMethod.GET)
```

- 요청 경로를 특정 메서드와 매핑하기 위해 사용하는 어노테이션
- 클래스에 적용하면 클래스 안의 모든 메서드에 기본 경로를 지정할 수 있다.
- 별도 설정이 없으면 요청 방식과 상관 없이 지정한 경로와 일치하는 요청을 처리한다.
- 특정 방식의 요청만 처리하고 싶은 경우 @GetMapping, @PostMapping, @PutMapping, @DeleteMapping, @PatchMapping 등을 사용할 수 있다.

### @RequestParam

```java
@PostMapping("/register/step2")
public String handleStep2(@RequestParam(value="agree", defaultValue="false") Boolean agreeVal) {
    if (!agree) {
        return "register/step1";
    }
    return "register/step2";
}
```

- 위 코드에서는 `agree` 요청 파라미터의 값을 읽어와 `agreeVal` 파라미터에 할당한다. 요청 파라미터의 값이 없으면 “false” 문자열을 값으로 사용한다.
- 스프링 MVC는 파라미터 타입에 맞게 String의 값을 변환해준다.

### 리다이렉트

- “redirect:경로"를 뷰 이름으로 리턴한다.
- 경로가 “/”로 시작하면 웹 어플리케이션을 기준으로 이동 경로를 생성한다.
- 경로가 “/”로 시작하지 않으면 현재 경로를 기준으로 상대 경로를 사용한다.

### 커맨드 객체

- Request Parameter의 값을 커맨드(command) 객체에 담아주는 기능을 제공한다.
- 값을 전달받을 수 있는 setter 메서드를 포함하는 객체를 커맨드 객체로 사용할 수 있다.
- 커맨드 객체가 리스트 타입의 프로퍼티를 가졌거나 중첩 프로퍼티를 가진 경우에도 Request Parameter의 값을 알맞게 커맨드 객체에 설정해준다.
    - HTTP Request Parameter의 이름이 `프로퍼티이름[인덱스]` 형식이면 List 타입 프로퍼티의 값 목록으로 처리한다.
    - HTTP Request Parameter의 이름이 `프로퍼티이름.프로퍼티이름` 형식이면 중첩 프로퍼티 값을 처리한다.

### Model

```java
@RequestMapping("/hello")
public String hello(Model model, @RequestParam(value="name", required=false) String name) {
    model.addAttribute("greeting", "안녕하세요, " + name);
    return "hello";
}
```

- 컨트롤러에서 뷰가 응답 화면을 구성하는데 필요한 데이터를 생성해서 전달할 때 사용한다.

### ModelAndView

```java
@GetMapping
public ModelAndView form() {
    List<Question> questions = createQuestions();
    ModelAndView mav = new ModelAndView();
    mav.addObject("questions", questions);
    mav.setViewName("survey/surveyForm");
    return mav;
}
```

- Model을 이용해서 View에 전달할 데이터 설정 + 결과를 보여줄 뷰 이름을 리턴, 이 두 가지를 한 번에 처리할 수 있다.

### @ModelAttribute

```java
@GetMapping
public String form(@ModelAttribute("login") LoginCommand loginCommand) {
    return "login/loginForm";
}
```

- 입력 폼과 폼 전송 처리에서 사용할 커맨드 객체의 속성 이름이 클래스 이름과 다르다면 `@ModelAttribute` 를 붙인 커맨드 객체를 파라미터로 추가할 수 있다.
