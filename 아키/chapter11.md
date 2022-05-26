# MVC1: 요청 매핑, 커맨드 객체, 리다이렉트, 폼 태그, 모델

## 요청 매핑 애노테이션을 이용한 경로 매핑

컨트롤러 클래스는 요청 매핑 애노테이션을 사용해서 메서드가 처리할 요청 경로를 지정한다. 요청 매핑 애노테이션에는 @RequestMapping, @GetMapping, @PutMapping 등이 있다.

여러 단계를 거쳐 하나의 기능이 완성되는 경우 관련 요청 경로를 한 개의 컨트롤러 클래스에서 처리하면 코드 관리에 도움이 된다.

회원 가입 과정을 예로 들면 `약관 동의 => 회원 정보 입력 => 가입 완료`라면, 다음과 같이 컨트롤러 클래스를 작성할 수 있다.

```java
@Controller
public class RegistController {

  @RequestMapping("/register/step1")
  public String handleStep1() {
    return "register/step1";
  }
  
  @RequestMapping("/register/step2")
  public String handleStep2() {
    return "register/step2";
  }
  
  @RequestMapping("/register/step3")
  public String handleStep3() {
    return "register/step3";
  }
}
```

이 때, 요청 매핑 url이 모두 `/register`로 시작한다. 이 경우 공통되는 부분을 @RequestMapping 애노테이션으로 처리할 수 있다.

```java
@Controller
@RequestMapping("/register")
public class RegistController {

  @RequestMapping("/step1")
  public String handleStep1() {
    return "register/step1";
  }
  
  @RequestMapping("/step2")
  public String handleStep2() {
    return "register/step2";
  }
  
  @RequestMapping("/step3")
  public String handleStep3() {
    return "register/step3";
  }
}
```

## GET과 POST 구분: @GetMapping, @PostMapping

@GetMapping을 사용하면 GET 방식으로만 들어오는 요청만 처리하도록 제한할 수 있다.

## 요청 파라미터 접근

만약 프론트엔드에서 폼으로 요청 파라미터값을 넘기는 경우를 생각해보자.

```html
<form action="step2" method="post">
<label>
	<input type="checkbox" name="agree" value="true"> 약관 동의
</label>
<input type="submit" value="다음 단계" />
</form>
```

컨트롤러 메서드에서 요청 파라미터를 사용하는 방법에는 2가지가 있다.

1. HttpServletRequest를 직접 이용
컨트롤러 처리 메서드의 파라미터로 `HttpServletRequest` 타입을 사용하고, HttpServletRequest의 `getParameter()` 메서드를 이용해 파라미터의 값을 구할 수 있다.

```java
// RegisterController.java 
@Controller
public class RegisterController {
	...
	@PostMapping("/register/step2")
	public String handleStep2(HttpServletRequest request) {
				String agreeParam = request.getParameter("agree");
		if (agreeParam == null || !agreeParam.equals("true")) {
			return "register/step1";
		}
		return "register/step2";
	}
		...
}
```

2. @RequestParam 애노테이션 사용
요청 파라미터의 개수가 많지 않다면 이 애노테이션을 사용해 간단하게 요청 파라미터의 값을 구할 수 있다.

```java
// RegustController.java
@Controller
public class RegisterController {

	@PostMapping("/register/step2")
	public String handleStep2(
			@RequestParam(value = "agree", defaultValue = "false") Boolean agreeVal){
		if (!agree) {
			return "register/step1";
		}
		return "register/step2";
	}
}
```
위 코드에서는 agree 파라미터를 읽어 agreeVal이라는 변수에 할당한다.

> **@RequestParam 어노테이션의 속성**  
> 1. value(String): HTTP 요청 파라미터의 이름을 지정
> 2. required(boolean): 필수 여부 지정
> 3. defaultValue(String): 값이 없을 때 사용할 문자열 지정

## 리다이렉트 처리

```java
// RegisterController.java
@Controller
public class RegisterController {
	// regist/step2 경로로 들어오는 요청중 POST 방식만 처리 
	@PostMapping("/register/step2")
	public String handleStep2() {
	}

	// regist/step2 경로로 들어오는 요청중 GET 방식만 처리, step1로 리다이렉트 시킴
	@GetMapping("/register/step2")
	public String handleStep2Get() {
		return "redirect:/register/step1";
	}
}
```

## 커맨드 객체를 이용해 요청 파라미터 사용

프론트엔드에서 전송하는 파라미터가 많을 경우 커맨드 객체를 통해 전달받을 수 있다.

```java
@PostMapping("/register/step3")
public String handleStep3(RegisterRequest regReq){
}
```

커맨드 객체로 사용하는 `RegisterRequest`에는 setter 메서드가 있어야만 한다.

```java
// RegisterRequest.java
public class RegisterRequest {
	//생략
	public void setEmail(String email) {
		this.email = email;
	}

	public String getPassword() {
		return password;
	}

	public void setPassword(String password) {
		this.password = password;
	}
	//생략
}
```

## 컨트롤러 구현 없는 매핑

단순히 뷰 이름만 리턴할 경우 WebMvcConfigurer 인터페이스의 addViewControllers() 메서드를 이용해 처리할 수 있다.

```java
@Controller
public class MainController{
	@RequestMapping("/main")
	public String main(){
			return "main";
	}
}
```
```java
// MvcConfig.java
@Configuration
@EnableWebMvc
public class MvcConfig implements WebMvcConfigurer {
	@Override
	public void addViewControllers(ViewControllerRegistry registry) {
		registry.addViewController("/main").setViewName("main");
	}
}
```

## Model을 통해 컨트롤러에서 뷰에 데이터 전달

뷰에 데이터를 전달하는 컨트롤러는 다음 두가지를 해야만 한다

1. 요청 매핑 어노테이션이 적용된 매서드의 파라미터로 Model을 추가
2. Model 파라미터의 addAttribute() 매서드로 뷰에서 사용할 데이터 전달

`addAttribute`의 첫번째 파라미터는 속성이름이고, 두번째 파라미터는 전달하려는 객체이다.

```java
// SurveyController.java
@Controller
@RequestMapping("/survey")
public class SurveyController {

	@GetMapping
	public String form(Model model) {
		List<Question> questions = createQuestions();
		model.addAttribute("questions", questions);
		return "survey/surveyForm";
	}

		// 설문항목
	private List<Question> createQuestions() {
		Question q1 = new Question("당신의 역할은 무엇입니까?",
				Arrays.asList("서버", "프론트", "풀스택"));
		Question q2 = new Question("많이 사용하는 개발도구는 무엇입니까?",
				Arrays.asList("이클립스", "인텔리J", "서브라임"));
		Question q3 = new Question("하고 싶은 말을 적어주세요.");
		return Arrays.asList(q1, q2, q3);
	}
}
```