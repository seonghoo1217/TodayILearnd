# OncePerRequestFilter 란

공식레퍼런스 상에는 Filter base class that aims to guarantee a single execution per request dispatch, on any servlet container.
It provides a doFilterInternal method with HttpServletRequest and HttpServletResponse arguments.라고 적혀있다.

어느 서블릿 컨테이너에서나 요청 당 한번의 실행을 보장하는 것을 목표로한다. doFilterInternal 메소드와 HttpServletRequest와 HttpServletResponse 인자(파라메터)를 제공한다.

중요한 점은 요청 당 한번은 실행을 반드시 보장한다.

기존의 필터는 A 서블릿이 실행되는 동안 다른 B 서블릿에서 요청이 오게 되면 기존에 A 서블릿을 진행하며 실행된 필터가 5개라면 B서블릿을 다시 실행할 때 5개를 실행해야하는
불필요한 과정이생긴다.

하지만 OncePerRequestFilter를 사용할경우 앞선 필터들을 또 한번 거치지 않고 쓸데없는 자원낭비를 줄일 수 있게된다.

결국 동일한 request안에서 한번만 필터링을 할 수 있게 해주는 것이 OncePerRequestFilter의 역할이고 보통 인증 또는 인가와 같이 한번만 거쳐도 되는 로직에서 사용한다.

## 일반 Filter 와의 차이

Filter와 OncePerRequestFilter 모두 사용자의 서블릿 리퀘스트를 가장 최전선에서 받을 때 사용된다.
하지만 두 객체간에도 차이점은 존재한다.

우선 Filter는 아래코드를 보면 Before라는 로그는 서블릿 실행전 After 로그는 서블릿 실행후에 찍히게된다. 여기 까지는 일반적인 Filter의 개념이다.

```kotlin
class LoggingFilter : Filter {
    override fun doFilter(request: ServletRequest?, response: ServletResponse?, chain: FilterChain?) {
        // do Filter 실행 전 로직
        println("Before")
        chain?.doFilter(request, response) ?: throw Exception()
        // do Filter 실행 후 로직
        println("After")
    }
}
```

이러한 Filter를 조금 더 확장한 GenericFilterBean 이다.

GenericFilterBean은 기존 Filter에서 얻어올 수 있는 정보인 Spring의 설정 정보를 가져올 수 있게 확장된 추상 클래스이다.

이 두 필터는 한 가지 공통점이 있는데 매 서블릿 마다 호출이 된다는 것이다.

이말의 뜻은 서블릿은 사용자의 요청을 받으면 서블릿을 생성해 메모리에 저장해주고 같은 클라이언트의 요청을 받으면 생성해둔 서블릿 객체를 재활용하여 
요청을 처리한다는 말이다.

문제는 다른 서블릿으로 dispatch되는 경우가 있을 수 있다.

가장 대표적으로 Spring Security에서 인증과 접근 제어 기능이 Filter로 구현되어진다.

이러한 인증과 접근 제어는 RequestDispatcher 클래스에 의해 다른 서블릿으로 dispatch되게 되는데, 이 때 이동할 서블릿에 도착하기 전에 다시 한번 filter chain을 거치게 된다.

바로 이 때 또 다른 서블릿이 우리가 정의해둔 필터가 Filter나 GenericFilterBean로 구현된 filter를 또 타면서 필터가 두 번 실행되는 현상이 발생할 수 있다.

이런 문제를 해결하기 위해 등장한 것이 바로 이번 글의 주인공인 OncePerRequestFilter이다.

OncePerRequestFilter는 그 이름에서도 알 수 있듯이 모든 서블릿에 일관된 요청을 처리하기 위해 만들어진 필터이다.

이 추상 클래스를 구현한 필터는 사용자의 한번에 요청 당 딱 한번만 실행되는 필터를 만들 수 있다. (이름 참 잘 지었다)

## Filter와 Interceptor의 차이?

필터는 DispatcherServlet 앞에서 먼저 동작하고 인터셉터는 DispatcherServlet 과 Controller 사이에서 작동한다.

필터

- 웹 어플리케이션의 Context 기능
- 스프링 기능을 활용하기 어려움
- 일반적으로 인코딩 CORS,XSS,LOG,인증,권한 등을 구현할 때 사용된다.

인터셉터

- 스프링의 Spring Context의 기능이며 일종의 빈이다.
- 스프링 컨테이너이기에 다른빈을 주입하여 활용성이 좋다.
- 다른 빈을 활용하기에 인증, 권한등을 구현할 수 있다.

