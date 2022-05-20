# Chp 7. AOP 프로그래밍

---

1. 프록시는 핵심 기능을 가지지 않는다. 핵심 기능은 대상 객체가 수행하고 프록시는 부가적인 공통 기능을 수행한다.<br>


2. AOP는 여러 객체에 공통으로 적용할 수 있는 기능을 분리해서 재사용성을 높여주는 프로그래밍 기법이다.
   <br>Spring이 제공하는 AOP방식은 런타임에 프록시 객체를 생성해서 공통 기능을 삽입하는 방식이다.


3. AOP에서 중요한 공통 용어들은 다음과 같다<br>

   ```text
       1. Advice: 언제 공통 기능을 핵심 로직에 적용할지를 정의한다.
       2. Joinpoint: Advice를 적용한 지점을 나타낸다(Spring의 경우 메서드 호출에 대한 JoinPoint만 지원한다)
       3. Pointcut: Joinpoint의 부분 집합으로 실제 Advice가 적용되는 Joinpoint를 나타낸다.
       4. Weaving: Advice를 핵심 로직 코드에 적용하는 것을 Weaving이라고 한다.
       5. Aspect: 여러 객체에 공통으로 적용되는 기능을 Aspect라고 한다.  
   ```

4. Advice(적용 시점)은 여러 가지가 있는데 대부분 Around Advice를 주로 사용한다.


5. @Aspect, @PointCut, @Around 를 이용한 AOP 구현<br>
   ```text
      - Aspect 애노테이션을 이용해서 공통 기능 구현 클래스를 만든다.
      - PointCut 애노테이션을 이용해서 적용할 대상을 설정한다.
      - Around(Advice) 애노테이션을 이용해서 어느 시점에 공통 구현을 넣을지와 공통 기능 구현을 한다.
   ```


6. 프록시 생성 방식<br>
   빈 객체가 인터페이스를 상속하면 인터페이스를 이용해서 프록시가 생성된다. 만약 구현체를 이용해서 프록시를 생성하고 싶다면
   ```text@EnableAspectJAutoProxy(proxyTargetClass = true)```를 이용한다.


7. Advice 적용 순서<br>
   한 대상 객체에 여러 Aspect를 적용할 수 있다. 그런데 이때, 순서가 상황에 따라 달라질 수 있기 때문에, @Order 애노테이션을 이용해서 적용 순서를 정할 수 있다.


8. @Pointcut을 따로 빼서 지정하고 해당 Pointcut을 이용하는 Around에서 정확히 주소를 가르쳐주면 공통적으로 사용할 수 있다.
