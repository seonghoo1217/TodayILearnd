# raw password cannot be null 에러에 대한 정리

## 에러 발생이유

대부분의 학생 개발자들이 회원가입 로직을 작성하고 구현하는 과정에서 비밀번호를 암호화하여 DB에 저장하려고 할것이고
그러한 과정에서 우리는 WebSecurityConfigurerAdapter를 extends하여서 구현한 Config파일에
보통 @Bean 클래스로 PasswordEncoder나 BCryptPasswordEncoder를 등록하여 이를 비밀번호 암호화,복호화하는데
사용할것이다. 이 과정에서 password를 encode할 때 password 매개변수에 null 값이 들어갈때 일어나는 에러이다.

* 실제 암호화와 복호화를 시켜주는 BCryptPasswordEncoder 구조체를 한번 살펴보자

```java
    @Override
	public String encode(CharSequence rawPassword) {
		if (rawPassword == null) {
			throw new IllegalArgumentException("rawPassword cannot be null");
		}
		String salt = getSalt();
		return BCrypt.hashpw(rawPassword.toString(), salt);
	}
```
암호화 시켜주는 로직의 일부분이다. 가장 중요한 부분이 rawPassword가 파라메터로 넘어올 때 rawPassword이 null일 경우
IllegalArgumentException이 발생하면서 개발자입장에서는 난생처음보는 오류인 rawPassword cannot be null이라는 황당한 에러로그만 보게된다.

회원가입 로직을 설계하는 경우 많은 인원들이 html Form전송 방식을 이용할 것이다.
물론 password또한 입력받는 과정에서 parameter로 넘길 값의 이름이 잘못되었거나 매칭되지않는 경우 발생하는 오류이기 때문에 
다음과같은 오류 해결 방식을 권장한다.

1. html 코드 확인
2. Service 로직확인
3. Entity값 확인
4. Security Config 확인
5. DB 테이블 구조 확인 