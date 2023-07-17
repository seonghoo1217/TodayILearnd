# J2EE 디자인패턴?
- Sun Microsystems에서 만들어진 반복되는 설계 문제를 해결하기 위한 모범 사례 모음이다.
  - 패턴은 다양한 문제에 적용할 수 있고, J2EE 개발자들의 성공적인 경험을 활용할 수 있는 솔루션
- 웹 기반의 엔터프라이즈 어플리케이션을  개발하기 위해 만들어졌다.
- J2EE: Java 2 Platform Enterprise Edition, JAVA SE를 확장한 엔터프라이즈 사양

## J2EE 디자인 패턴 종류
대표적인 3가지 레어어로 `Presentation`,`Business`,`Integration` Layer로 나누어진다.

## Presentation Layer Design Pattern

### Intercepting Filter Pattern
- 사용자 요청의 전처리와 후처리를 쉽게 처리