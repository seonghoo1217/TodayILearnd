작성배경
이번 프로젝트에서 OAuth2.0을 사용하여 인가처리를 사용하였는데 Github의 경우 OAuth Developer 문서를 읽어보면 SpringSecurity에서 제공하는 OAuth API를 사용하기 보다는 직접 API를 호출하는 방식을 권고하고 있다.



물론 방식의 차이일뿐 같은 결과를 가져오지만 인가처리는 SpringSecurity의 Oauth2.0을 사용하였으나 logout을 구현하기 위해 외부 API를 호출해야하는 작업이 필요해졌다.



Github의 logout주소는 아래와 같다

https://api.github.com/applications/{client_id}/token
이 API로 요청을 보내면서 Header에는 client_id와 client_secret을 Body에는 authToken을 넣어 전송해야한다.



이때 자바의 경우 전송하는 방법이 대표적으로 3가지정도 존재하기 때문에 참고하여 상황에 맞는 방식으로 API를 구현해야한다.



물론 Spring에서 지원하는 RestTemplate을 사용하는 것을 필자는 권고한다.



사용방법


1. HttpURLConnection
   java.net 클래스에서 제공하는 URL 요청을 위한 클래스로서 JDK1.1부터 자바에서 HTTP통신을 위해 존재한 상당히 오래된 클래스이다.

HttpURLConnection은 기본적으로 GET 요청방식을 가지고, setRequestMethod() 메소드로  POST,PUT,DELETE와 같은 다른 요청방식을 설정할 수 있다. 또한 HTTP 표준을 준수하며 사용이 간단하다. 하지만 원시적인 API로 auto redirection을 지원하지 않으며 결과값을 Stream으로 하나씩 받아와야하며 그에 따라 코드의 길이가 길어지게된다.



밑은 간단한 예시이다.

public String getAccessToken(String code) throws IOException {
URL url = new URL("https://github.com/login/oauth/access_token");
System.out.println("code="+code);
HttpURLConnection conn = (HttpURLConnection) url.openConnection();
conn.setDoInput(true);
conn.setDoOutput(true);
conn.setRequestMethod("POST");
conn.setRequestProperty("Accept", "application/json");
//    conn.setRequestProperty("User-Agent", "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/96.0.4664.45 Safari/537.36");

      // 이 부분에 client_id, client_secret, code를 넣어주자.
      // 여기서 사용한 secret 값은 사용 후 바로 삭제하였다.
      // 실제 서비스나 깃허브에 올릴 때 이 부분은 항상 주의하자.
      try (BufferedWriter bw = new BufferedWriter(new OutputStreamWriter(conn.getOutputStream()))) {
         bw.write("client_id="+client_id+"&client_secret="+ client_secret +"&code=" + code);
         bw.flush();
      }

      int responseCode = conn.getResponseCode();
      String responseData = getResponse(conn, responseCode);
      System.out.println("responseCode="+responseCode);
      System.out.println("responseData="+responseData);

      conn.disconnect();
      return responseData;
}


사용법이 정말 간단하다.

URL 객체를 생성 후 호출할 API의 도메인을 넣어준후 세팅을 마친후 Stream으로 처리해주면된다.



2. HttpClient
   Apache HttpComponents로 불리며 HttpURLConnection을 사용할 때보다 조금 더 짧고, 직관적으로 코드를 짤수있다는 장점이 존재한다.

무엇보다 Response값을, URLConnection과 달리 exeute메서드로 요청해서 EntityUtils.tostring()으로 바로 문자열로 받아올 수 있어 편하다.

요청방식도 따로 설정해주지 않고, 각각 메소드로 정의되어있어서, 더 직관적으로 볼 수 있다.



또한 HTML, JSON, XML 등 다양한 응답 유형을 처리할 수 있고, 인증, 쿠키, 프락시 등 다양한 기능을 지원하며, 연결 풀링, 커넥션 타임아웃 등 연결 관리 기능을 제공하기에 Restful 서비스에서 유용하게 사용된다.



하지만, URLConnection에서 발전한 방법임을 가만해도 여전히 반복되는 레거시 코드가 존재하게 되며, 응답의 컨텐츠 타입에 따라 별도의 로직이 필요하다.

성능적인 측면에서도 HttpClient는 매번 요청 시마다 연결을 설정하고 끊기 때문에, 많은 요청을 처리하는 경우 성능 문제가 발생할 수 있고 다중 스레드 환경에서 불안정하게 동작하기도 한다.

import java.io.IOException;
import org.apache.http.HttpEntity;
import org.apache.http.client.methods.CloseableHttpResponse;
import org.apache.http.client.methods.HttpGet;
import org.apache.http.impl.client.CloseableHttpClient;
import org.apache.http.impl.client.HttpClientBuilder;
import org.apache.http.util.EntityUtils;

public class HttpClientExample {
public static void main(String[] args) {
String url = "https://www.naver.com";
try (CloseableHttpClient httpClient = HttpClientBuilder.create().build()) {
HttpGet request = new HttpGet(url);

            try (CloseableHttpResponse response = httpClient.execute(request)) {
               HttpEntity entity = response.getEntity();
               if (entity != null) {
                  String result = EntityUtils.toString(entity);
                  System.out.println(result);
               }
            }
         } catch (IOException ex) {
            System.err.println("HTTP request failed: " + ex.getMessage());
         }
      }
}
앞서 말했듯 HttpClient의 장점인 toString을 통해 바로 객체를 문자화 시켜 간편하게 loggin 및 반환받을 수 있다는 점이 존재하며 전송또한 간단하다.



3. RestTemplate
   RestTemplate은 Spring 프레임워크에서 제공하는 HTTP 클라이언트 라이브러리로, RESTful 웹 서비스를 호출하는 데 사용되는 대표적인 라이브러리이다.

RestTemplate은 기본적으로 HttpURLConnection 또는 HttpClient를 사용하여 RESTful 웹 서비스에 HTTP 요청을 보내고, HTTP 응답을 처리합니다.



RestTemplate은 HttpURLConnection 및 HttpClient에서 발전한 형태라고 볼 수 있다.

이전의 HTTP 클라이언트 라이브러리들은 주로 저수준의 HTTP 요청과 응답 처리를 위해 설계되었으며, 개발자가 수동으로 HTTP 요청 및 응답을 처리해야 했지만, RestTemplate은 Spring 프레임워크의 기능과 유연성을 활용하여, HTTP 요청 및 응답 처리를 간편하게 구현할 수 있도록 지원한다.



RestTemplate은 다양한 HTTP 요청 및 응답 형식을 지원하며, HTTP 클라이언트 라이브러리의 일부 단점을 극복하고, 더욱 쉬운 HTTP 클라이언트 작성을 지원한다.



그러나 RestTemplate은 많은 기능을 가지고 있기 때문에 메모리 사용량이 크며, 동시 요청 처리에 제한이 있을 수 있다.



어떻게 HttpURLConnection 및 HttpClient에서 발전하였는가?
기본적으로 앞서말한 HttpURLConnection과 HttpClient 모두 Http 프로토콜을 기반으로 크게 5가지의 같은 방식을 가지고 진행한다.



1. 연결설정 : URL 객체를 만들어 API를 호출할 상태를 만든다.



2. 요청설정: HTTP의 요청에 대한 설정을 진행한다. 요청방식 또는 요청헤더와 요청바디와 같은 것을 설정한다.



3. 요청전송: 요청 대상인 서버로 전송한다.



4. 응답수신: 서버로 부터 응답받은 데이터들을 텍스트,JSON,XML 등으로 파싱하여 응답받는다.



5. 연결 종료: 연결 및 응답을 종료한다.



이 부분 또한 RestTemplate에서 작동하지만 사용자가 간편하게 사용할 수 있게 해준다.



해당작업들을 스프링 프레임워크에서 공식적으로 제공함으로서 객체 생성 및 설정을 간편하게 처리할 수 있다. 또한, 객체 생성과 관련된 복잡한 코드를 줄일 수 있게 발전하였다.



RestTemplate의 동작원리



어플리케이션이 RestTemplate를 생성 -> URI, HTTP메소드 등의 헤더를 담아 요청
RestTemplate는 HttpMessageConverter를 사용하여 requestEntity를 요청메세지로 변환한다.
RestTemplate는 ClientHttpRequestFactory로 부터 ClientHttpRequest를 가져와서 요청을 보낸다.
ClientHttpRequest는 요청메세지를 만들어 HTTP 프로토콜을 통해 서버와 통신한다.
RestTemplate는 ResponseErrorHandler로 오류를 확인하고 있다면 처리로직을 태운다.
ResponseErrorHandler는 오류가 있다면 ClientHttpResponse에서 응답데이터를 가져와서 처리한다.
RestTemplate는 HttpMessageConverter를 이용해서 응답메세지를 java object(class responseType)으로 변환한다.
어플리케이션에 반환된다.
사용코드
RestTemplate restTemplate = new RestTemplate();

HttpHeaders headers = new HttpHeaders();
headers.set("Authorization", "Basic " + Base64Utils.encodeToString((client_id + ":" + client_secret).getBytes()));
headers.set("Accept", MediaType.APPLICATION_JSON_VALUE);

Map<String, String> requestBody = new HashMap<>();
requestBody.put("access_token", git_authToken);

HttpEntity<Map<String, String>> requestEntity = new HttpEntity<>(requestBody, headers);

ResponseEntity<Void> response = restTemplate.exchange(
"https://api.github.com/applications/"+client_id+"/token",
HttpMethod.DELETE,
requestEntity,
Void.class,
client_id
);
System.out.println("Response="+response.getStatusCode());
if (response.getStatusCode() == HttpStatus.NO_CONTENT) {
MemberSocial findM = memberService.getMemberInfoByAuthToken(git_authToken);
findM.updateInfoByLogout(null,null);
}


코드분석
위의 코드를 보면 logout을 위해 필요한 client_id와 secret을 header에 담고 body에는 git의 access_token(authToken이라 칭함)을 넣어 HttpEntity, 즉 요청 개체로 만들어 이를 restemplate의 요청에 같이 담아 보내며 이때 전송방식은 문서에 따라 delete를 이용하여 전송하고 반환값이 필요 없기에 Void를 선택하였다.



이때 옳바른 요청이 도달할 경우 204를 반환하기 때문에 204가 응답결과로 올경우 회원의 refreshToken과 authToken을 제거하는 로직을 구현하였다.





결론
이 처럼 외부 REST API를 호출하는 방법은 여러가지가 존재하지만 스프링 프레임워크를 이용하는 만큼 공식적으로 지원하는 RestTemplate 객체를 이용하는 것이좋은 방법이라고 설명하고 싶다