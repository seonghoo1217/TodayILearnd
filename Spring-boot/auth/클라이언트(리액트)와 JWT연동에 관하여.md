# 클라이언트(리액트)와 JWT 연동에 관하여

* 최근들어 클라이언트와 협업을 자주하여 API 서버를 구축할 일이 많아짐에 따라 인증방식을 Form 방식에서 JWT 토큰인증 방식으로 바꾸게 되었다.
* 그에 있어서 지켜야하는 점과 유용한 팁을 주려고한다.


## 백엔드(SpringBoot)

1. API 서버측에서는 별도의 JWT관련된 yml 형태의 파일을 생성하여 관리한다.
2. yml 파일에는 AccessToken과 RefreshToken에 대한 설정값들을 지정해준다.
3. 설정값은 아래 코드와 같다.
```yaml
jwt:
  secret: JWT 시크릿 키 값을 넣어줍니다.64bit이상의 값이어야합니다.

  access:
    expiration: AccessToken만료 시간
    header: AccessToken 발급시 이름

  refresh:
    expiration: RefereshToken만료 시간
    header: RefreshToken 발급시 이름
```

여기서 Secret 키값은 일반 String형태의 값을 **Base64** 알고리즘으로 인코딩하기 때문에 인코딩된 값을 넣어도 되고 아래와같이 코드적으로 알고리즘을 사용하여 암호화하여 주어도 된다.
(Secret키 값을 토대로 알고리즘을 사용하여 JWT를 암호화 시키기 떄문에 노출시키지 않는 편이 제일좋으며 AWS SecretManager로 숨기는방법을 추천)

AccessToken의 만료기한은 RefreshToken보다 적게 설정한다.(RefreshToken으로 AccessToken을 재발급 받기 떄문)클라이언트 개발자(프론트엔드 개발자)와 협의하여 네이밍하면 좋다.

토큰의 발급이름은 클라이언트 개발자(프론트엔드 개발자)와 협의하여 네이밍하면 좋다.

* 코드적으로 Base64로 시크릿 키 값을 인코딩해주는 코드
```java
    
private static String SECRET_KEY = "";

@PostConstruct
public void secretKeyEncode() throws UnsupportedEncodingException {
        byte[] secretBytes = secret.getBytes("UTF-8");
        Base64.Encoder encoder = Base64.getEncoder();
        byte[] enCodeBytes = encoder.encode(secretBytes);
        SECRET_KEY = new String(enCodeBytes);
        }
```

4. 토큰의 유효 기간은 현재 진행중인 프로젝트의 특성에 따라 다르게 설정하는 것이 좋다.

(본인의 경우 쇼핑카트 프로젝트는 AccessToken을 10분에 RefreshToken을 2시간으로 설계하였다.)

5. 반드시 RefreshToken을 통해 AccessToken을 재발급하여주는 API를 제작하여야한다.