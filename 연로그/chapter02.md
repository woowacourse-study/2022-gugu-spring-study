# Chapter 2 - 스프링 시작하기

### 빌드와 의존 전이

빌드 도구에는 크게 `Ant`, `Maven`, `Gradle`이 있다.  
요즘에는 `Maven`과 `Gradle`을 많이 사용하는 편이다.

#### Maven

- 설정 정보 관리 파일: pom.xml
- 코드 컴파일/실행 시 `<dependency>`로 설정한 아티팩트 파일 사용
- 로컬 레포지토리에 .jar 파일이 존재하면 해당 파일 사용
- 로컬 레포지토리에 존재하지 않으면 메이븐 원격 중앙 레포지토리에서 다운받음

> 의존 전이; Transitive Dependencies  
> `<dependency>`를 통해 아티팩트 파일을 다운 받을 때, 해당 아티팩트가 의존하는 아티팩트까지 다운로드 한다.  
> 의존하는 대상 뿐만 아니라 의존 대상이 의존하는 대상까지 함께 다운로드되므로 의존 전이라고 표현한다.

#### Gradle

- 설정 정보 관리 파일: build.gradle
- 명령어 실행에 성공하면 프로젝트 루트 폴더에 gradlew.bat, gradlew 래퍼 파일 생성

> 더 자세한 설명은 [블로그 글](https://yeonyeon.tistory.com/89) 로 대체한다.

### Spring bean 등록하기

- 스프링이 생성하는 객체
- `@Configuration`와 `@Bean`을 이용해 등록할 수 있음
- `@Component`를 이용해 등록할 수 있음

> `@Configuration`: 스프링 설정 클래스로 지정하는 어노테이션  
> `@Bean`: 메서드에 붙여 스프링이 해당 메서드로 객체를 생성해 빈으로 등록  
> `@Component`: 클래스에 붙이면 스프링이 객체를 생성해 빈으로 등록

#### example1

```java

@Configuration
public class AppContext {
    @Bean
    public CustomClass customClass() {
        return new CustomClass();
    }
}
```

#### example2

```java

@Component
public class CustomClass {
    // ...
}
```

### Spring은 객체 컨테이너

![메이븐의 의존 그래프 일부](./img/maven_injection_graph.png)

#### BeanFactory

- 객체 생성과 검색에 대한 기능 제공  
  ex: 객체 검색에 필요한 `getBean()` 메서드
- 싱글톤 / 프로토타입 빈 확인 기능 제공

#### ApplicationContext

- 메시지, 프로필 / 환경 변수 등 처리할 수 있는 기능 추가 정의
- 빈 객체의 생성, 초기화, 보관, 제거 등을 관리해 `Container`라고도 불림
- 다양한 구현체 존재
- AnnotationConfigApplicationContext: 자바 어노테이션을 이용한 클래스로부터 객체 설정 정보 가져오기
- GenericXmlApplicationContext: xml로부터 객체 설정 정보 가져오기
- GenericGroovyApplicationContext: 그루비 코드를 이용해 객체 설정 정보 가져오기

> 컨테이너란?
> - 객체를 생성 및 관리하며 의존 관계를 연결해주는 것.
> - 이 책에서는 `BeanFactory`와 `ApplicationContext` 등을 스프링 컨테이너라고 표현한다.

### 싱글톤 객체

- 싱글톤: 단일 객체(single object)
- 스프링은 **Bean 객체 단 하나만 생성**하는 것이 디폴트 설정
- 이를 싱글톤 범위를 갖는다고 표현

> 이 외에 프로토타입 범위도 존재하는데 이는 6장에서 다룬다.
