# SpringBoot의 출발점
SpringBoot는 Spring이 `Containerless` web application architecture를 지원해줬으면 좋겠다는 한 개발자의 요청으로 부터 시작되게 되었다.

여기서 우리가 주목해야할 키워드는 **Containerless**이다. 나는 맨처음에 Contain + less를 단순히 생각하여, 컨테이너가 존재하지 않는 상태의 프레임워크를 지칭하는 용어인줄 알았다.

하지만, 그렇다면 WAS의 개념은? 톰캣은 어디에서 구동되는 것인가? 의문이 있었지만, 다행 컨테이너의 유무가 아니었다.

## Containerless
- 서버의 설치(톰캣 직접 설치) 및 관리가 별도로 필요없는 상태를 뜻한다.

## Container
- 웹 컴포넌트가 하나 있을 때 컴포넌트가 일을 하기 위해서는 웹 클라이언트의 요청이 필요하다.
- 웹 클라이언트의 요청이 들어오면 동적 컨텐츠를 만들어 응답을 웹 클라이언트에게 보내준다.
- 웹 컴포넌트는 항상 웹 컨테이너 안에 들어있어야 한다.

### Web Container
- 웹 컴포넌트의 라이프 사이클 관리
- 웹 클라이언트의 요청이 들어오면 정해진 규칙에 따라 어느 컴포넌트가 요청을 처리할지 넘겨준다. (라우팅, 매핑)
- Java 개념에서 **컴포넌트**는 `서블릿(Servlet)` 컨테이너 `서블릿 컨테이너(Servlet Container)`로 비유할 수 있다.

## Spring과 SpringBoot

### Spring에서
Spring Container는 웹 컨테이너에서 여러 개의 컴포넌트(Bean)를 관리한다.
스프링 컨테이너는 웹컨테이너를 대체할 수 없다.

### SpringBoot에서 
Web Container(Servlet Container)를 신경쓰지 않고 개발만! 집중할 수 있게 해준다.
이를 독립 실행형(standalone) 애플리케이션이라고 한다.