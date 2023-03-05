## **소개 배경**

![yaml](https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcQbOVCKk6SQF3bQd2APXCldD3J7a2YoDS6JEA&usqp=CAU)

최근 진행중인 프로젝트의 경우 `OAuth`와 `JWT` 환경에서 동작되고 있다. 

그에 따라 yaml 파일의 가독성 저하와 `profiles`를 사용하고 있음에도 목적에 맞게 사용하지 않는다고 생각하여 포스팅하게 되었다.

우선 yaml에서 파일은 분리하기 위한 설정 요소들에 대한 설명과 사용하는 법에 대해 설명하고자한다.

## 엥 Profiles?
Profiles은 Spring Framework의 고유한 설정 방식 중 하나로 나의 Application이 어떤 환경에서 동작할지를 설정하는 것이다.

여러 개의 환경에 대한 설정을 관리하기 위한 기능으로 구성하고자 하는 환경에 관한 설정파일을 생성한 후

해당 환경이 동작할 때 해당환경에 맞는 설정파일에서 요소들을 활성화시켜 애플리케이션을 구동한다.

예를들어 개발환경과 운영환경에서의 DB 연결 또는 로깅 설정에 차이가 날 수 있기에 설정파일을 분리할 수 있는 기능을 지원한다.

## include

그 다음단계로는 `include`가 있다.

include는 profiles로 분리한 설정 정보가 상호 필요한 경우 사용한다.

나의 경우 jwt와 oauth의 정보가 각각 필요하기에 설정값을 넣었다.

```yaml
include:
      - jwt
      - oauth
```

예시와 같이 여러개의 yaml 파일에 대하여 사용할 때는 이렇게 열거형으로 사용할 수 있다.

## active
`profiles`와 `include` 설정이 모두 완료되었다 사용한다는 것을 명시적으로 선언해줘야한다.

그 작업이 바로 `active`이다.

profile과 active를 사용할 때는 별도의 yaml 파일을 생성하여 사용할 수 있다.

yaml 파일의 이름 형식은 **`application-${profile_name}.yml`**을 준수하여야한다.

만약 별도의 yaml 파일이 존재하지 않을 경우 컴파일 단계에서 에러가 발생하게 되며, 애플리케이션이 실행되지 않는다.

```yaml
    active: jwt, oauth
```

active까지 사용한다면 yaml 파일을 분리하여 사용할 수 있는 조건은 모두 갖추게된다.

## 그래서 그거 왜 그렇게 하는데?

크게 보자면 세가지 정도의 이점이 존재한다.

**1. 설정 파일의 명시성과 가독성 증가**

- 앞서 말했듯이 우선 가장 큰 이점으로는 가독성 향상이 존재한다.
- 각 profile 별로 설정이 서로 분리되어 있기 때문에, 설정 파일의 가독성이 증가한다.
- 또한 메인 yaml 파일인 `application.yml`에서 개발 / 운영 환경의 설정이 섞여 있으면 파일의 용도가 명시적이지 못하다.

**2. 설정 파일의 유연성 향상**
- 각 프로파일에 대한 설정 파일을 별도로 분리하면, 다양한 프로파일에 대한 설정을 더욱 쉽게 관리할 수 있다.
- 동일한 애플리케이션을 여러 환경에서 사용하는 경우, 각각의 환경에 맞게 설정 파일을 쉽게 변경할 수 있다.

**3. 설정 파일의 보안 향상**
- 애플리케이션의 설정 정보는 보통 중요한 정보를 포함하고 있기 때문에, 보안에 대한 고려가 필요함
- 이 경우, 프로파일 별로 설정 파일을 분리하면, 보안에 민감한 설정 정보를 개발자와 운영팀 등이 서로 다른 파일에서 관리할 수 있기 때문에, 보안에 대한 위험을 줄일 수 있다.

## 변경된 코드

```yaml
spring:
  security:
    oauth2:
      client:
        registration:
          github:
            client-id: ${github-id}
            client-secret: ${github-secret}
            redirectUri: ${github-callback}
            scope: user
  
  datasource:
    driver-class-name: org.mariadb.jdbc.Driver
    url: ${jdbc-url}
    username: ${username}
    password: ${password}

  jpa:
    hibernate:
      ddl-auto: update
    generate-ddl: true
    properties:
      hibernate:
        format_sql: true
        default_batch_fetch_size: 100
jwt:
  secret: ${token-secret}

  access:
    expiration: 86400 #3600*24
    header: Authorization

  refresh:
    expiration: 604800 #3600*24*7
    header: Authorization_refresh

aws:
  secretsmanager:
    name: mogakco
    enabled: true

cloud:
  aws:
    stack:
      auto: false
    region:
      static: ap-northeast-2
```

해당 코드의 경우 profile 별로 분리하지 않고 메인 설정파일인 `application.yml`에 작성되어있던 코드이다.

물론 동작에는 문제가 없으나 가독성과 유연성이 상당히 떨어지게 된다. 

이를 개선하기 위하여 profiles를 적용하면 다음과 같다.

```yaml
spring:
  profiles:
    include:
      - jwt
      - oauth
    active: jwt, oauth
```
우선 해당 코드를 추가한다.

그리고 그에 맞는 yaml 파일들을 추가 생성하여 준다.

우선 **`application-jwt.yml`**부터 보자면 기존 application.yml에 존재하던 설정 값을 분리한다.

```yaml
## application-jwt.yml
jwt:
  secret: ${token-secret}

  access:
    expiration: 86400 #3600*24
    header: Authorization

  refresh:
    expiration: 604800 #3600*24*7
    header: Authorization_refresh
```

oauth설정과 aws 설정 또한 마찬가지이다.

oauth 설정의 경우 `application-oauth.yml`으로 Naming 후 분리한다.

aws 설정값의 경우 아래 dependency로 인해 `bootstrap.yml`으로 Naming 해준다.

```yaml
    implementation 'org.springframework.cloud:spring-cloud-starter-aws-secrets-manager-config:2.2.6.RELEASE'
```

```yaml
##application-oauth.yml
spring:
  security:
    oauth2:
      client:
        registration:
          github:
            client-id: ${github-id}
            client-secret: ${github-secret}
            redirectUri: ${github-callback}
            scope: user
```

```yaml
##bootstrap.yml
aws:
  secretsmanager:
    name: mogakco
    enabled: true

cloud:
  aws:
    stack:
      auto: false
    region:
      static: ap-northeast-2
```

파일을 scope 단위로 분리후의 application.yml파일이다.

```yaml
spring:
  profiles:
    include:
      - jwt
      - oauth
    active: jwt, oauth
  datasource:
    driver-class-name: org.mariadb.jdbc.Driver
    url: ${jdbc-url}
    username: ${username}
    password: ${password}

  config:
    name: application,application-jwt,application-oauth
    location: classpath:/application.yml,classpath:/application-jwt.yml,classpath:/application-oauth.yml


  jpa:
    hibernate:
      ddl-auto: update
    generate-ddl: true
    properties:
      hibernate:
        format_sql: true
        default_batch_fetch_size: 100
```

결론적으로 분리 이후의 가독성과 명시성이 상당히 높게 작용된다.

## 만약

만약 프로젝트의 전체규모가 작거나 단순한 프로젝트라면 굳이 profile 별 분리를 할필요가 없다고 생각되나

프로젝트의 크기가 있고 지금과 같이 jwt, oauth와 같은 별도의 전략을 사용한다면 반드시 yaml 파일을 profile 단위로 분리하는 것을 권고한다.

