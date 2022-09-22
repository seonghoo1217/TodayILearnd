# OAuth 란? 유래 및 발전과정

* Oauth는 프로토콜 규약으로서 2006년도에 Twitter와 Google이 정의한 개방형 Authorization이다.

* 서버의 API 요청 허락을 목적으로 JSON 형태로 개발되었다.

사용자는 한번의 인가 절차로 웹 서비스의 리소스를 접근할 때 로그인 자격증명(가장 많이 사용되는 방법은 ID와 PW이다.)과 같은 개인정보를
전송하지 않고도 자신의 접근 권한 또는 기타 권한을 부여 할 수 있도록하는것

예시로 많은 웹사이트에서 소셜 로그인을 지원하는 것이 예로, 소셜로그인에 접근 권한을 요청하고 요청이 통과되면 사용자가 웹서비스를 별도의 회원가입
로그인을 하지않아도 된다.

## OAuth 탄생배경

OAuth 등장이전에 A사이트와 B사이트가 있다고 가정한다. A사이트에서 B사이트의 리소스를 가져오기 위해선 다른사이트의 ID와 PW를 직접
입력받아 저장하여 필요할때마다 불러와서 사용해야한다. 이때 단점이 발생한다.

사용자 : A사이트에서 B사이트로 ID와 PW를 넘겨주는것에 신뢰할 수 없음

A사이트 : ID와 PW를 받기 때문에 보안부분에 있어 문제 발생시 책임을 져야한다.

B사이트 : A사이트를 신뢰할 수 없음

위와 같은 문제를 해결하기 위해 2006년 트위터 개발자와 Gnolia의 개발자가 안전한 인증방식에 대한 논의를 하면서 OAuth가 등장했고 2010년에 OAuth1.0이 발표되었다.
현재는 OAuth의 세션 고정 공격을 보완한 OAuth 1.0a를 거쳐, OAuth의 구조적인 문제점을 해결하고,
핵심요소만을 차용한 유사 프로토콜 WRAP(Web Resource Access Protocol)을 기반으로 발표한 OAuth 2.0가 많이 사용되고 있다.

## OAuth2.0 용어

1. Resource Server : OAuth2.0 서비스를 제공하고 자원을 관리하는 서버(보통 google, naver 같은 다른 사이트)

2. Resource Owner(자원 소유자) : Resource Server의 계정을 소유하고 있는 사용자 (사용자)

3. Client : Resource Server의 API를 사용하여 데이터를 가져오려고 하는 사이트 (개발 사이트)

4. Authorization Server(권한 서버) : Client가 Resource Server의 서비스를 사용할 수 있게 인증하고 토큰을 발생해주는 서버(인증 서버, google, naver)

5. Access Token	자원 서버에 자원을 요청할 수 있는 토큰

6. Refresh Token	권한 서버에 접근 토큰을 요청할 수 있는 토큰

## 인증 절차 종류

1. Authorization Code Grant
- Client가 다른 사용자 대신 특정 리소스에 접근을 요청할 때 사용
- 리소스 접근을 위해, Authorization Server에서 받은 권한 코드로 리소스에 대한 액세스 토큰을 받는 방식
- 다른 인증 절차에 비해 보안성이 높기에 주로 사용된다.

2. Implicit Grant

- Authorization Code Grant과 다르게 권한 코드 교환 단계가 있음
- 액세스 토큰을 즉시 반환받아 이를 인증에 이용하는 방식
 
3. Resource Owner Password Credentials Grant

- Client가 암호를 사용하여 액세스 토큰에 대한 사용자의 자격 증명을 교환하는 방식
- Resource Owner에서 ID, Password를 전달 받아 Resource Server에 인증하는 방식으로 신뢰할 수 있는 Client야 가능

4. Client Credentials Grant	

- Client가 컨텍스트 외부에서 액세스 토큰을 얻어 특정 리소스에 접근을 요청할 때 사용하는 방식

## 현재 많이 사용중인 Oauth2.0 Frame work Version

![](https://www.ssemi.net/content/images/size/w1000/2021/12/image-1.png)

* 다양한 클라이언트 환경에 적합한 인증 및 인가의 부여 방법(Grant Type)을 제공하고 그 결과로 클라이언트에 접근 토큰을 발급하는것에 대한 구조


