## Chapter 15 - 간단한 웹 어플리케이션의 구조

### 간단한 웹 어플리케이션의 구조

- `Front Servlet`
    - 웹 브라우저의 모든 요청을 받는 창구 역할
    - Spring의 DispatcherServlet
- `Controller` + `View`
    - 서비스 호출
    - 응답 결과 생성에 필요한 모델 생성
    - 응답 결과 생성할 뷰 선택
- `Service`
    - 클라이언트가 요구한 기능에 대한 로직을 제공
- `DAO`
    - DB와 웹 어플리케이션 간에 데이터 이동시켜 주는 역할

> (+)  
> [프론트 컨트롤러](https://yeonyeon.tistory.com/103) 에 대한 간단한 설명

### 궁금한 점
- 각자의 패키지 구성 방식