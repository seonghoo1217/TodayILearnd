# 자바의 Key 클래스

java.security 패키지에 속하며 토큰을 암호화하는데 사용된다. java.io.Serializable을 상속받는다.

## 주요 메서드

1. getAlgorithm() : 토큰을 HS256 해싱알고리즘을 사용하여 암호화시킨다.

2. getFormat() : 토큰을 포맷한다.

3. getEncoded() : 토큰을 암호화시킨다.