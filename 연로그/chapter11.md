# Chapter 11 - 요청 매핑, 커맨드 객체, 리다이렉트, 모델

### 웹 어플리케이션 개발하기

웹 어플리케이션은 크게 두 종류의 코드로 나눌 수 있다.

- 특정 요청 URL을 처리할 코드
- 처리 결과를 html과 같은 형식으로 응답하는 코드

### @RequestMapping

- 요청 URL 지정 가능
- 공통 부분은 클래스 상단에 설정 가능
- `@GetMapping`, `@PostMapping`, `@PutMapping`, `@DeleteMapping` 등으로 전송 방식 구분 가능

#### AS-IS

```java

@Controller
public class UserController {
    @RequestMapping("/users/step1")
    public String step1() {
        // ...
    }

    @RequestMapping("/users/step2")
    public String step2() {
        // ...
    }
}
```

#### TO-BE

```java

@Controller
@RequestMapping("/users")
public class UserController {
    @RequestMapping("/step1")
    public String step1() {
        // ...
    }

    @RequestMapping("/step2")
    public String step2() {
        // ...
    }
}
```

### 요청 / 응답

- 요청
    - @RequestParam
    - @ModelAttribute
    - HttpServletRequest
    - InputStream
    - HttpEntity
    - @RequestBody
- 응답
    - String
        - "redirect:/URL경로" 식으로 리다이렉트 구현 가능
    - HttpServletResponse
    - Writer
    - ModelAndView
    - @ResponseBody

> 해당 내용에 대해서는 학습한 적이 있어 자세한 내용은 블로그 링크로 대체한다.  
> 👉 [요청 파라미터](https://yeonyeon.tistory.com/134), [응답 데이터](https://yeonyeon.tistory.com/135)