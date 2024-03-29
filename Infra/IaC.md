# Infrastructure As Code

IaC의 최대 장점은 수동으로 설정하던 인프라 구조를 코드를 통해 작성함으로서 획일화 된 작업으로 설정 및 제거 할 수 있다는 것다.

즉, 코드를 통해서 인프라를 관리하고 프로비저닝할 수 있다.

## 어떤 상황에 효율적일까? 🤔
- 인프라를 완전히 다른 리전이나 IaaS에 똑같이 복제하고 싶을 경우
- 해당 리전이 사용할 수 없는 상황에 직면했을 경우
- 기존과는 다른 새로운 아키텍처를 빠른 시간 내에서 적용해야할 경우

### 다른 이점은 뭐가있을까?
GUI 환경에서는 쉽게 서비스를 제공하고, 아키텍처를 빠르게 실험해볼 수 있다는 장점이 있었다.

하지만, 아래와 같은 단점을 가지고 있다.
- 설정값이 인프라 환경에 어떠한 영향 주는지 파악하기에 명시적이지 않다.
- 환경 설정에 대한 내용을 다른 팀 멤버에 전달하기 어렵다.

## IaC의 주요가치
인프라를 구축하는데 있어 가장 중요한 가치는 자동화이며, 이는 IaC의 사용이유와도 직결된다.
GUI환경의 경우 각 요소를 별도로 지정해주어야하는 반면, 코드를 통해 클라우드 인프라스트럭처의 생성/수정/삭제를 자동화할 수 있다는 이유이다.

즉 코드를 이용하여 서버, DB, Network, Test등의 인프라를 설계할 수 있는것에 주요가치를 가진다.

## IaC의 장점
- 인프라를 만드는 과정이 자동화되므로, 오류가 훨씬 덜 발생(만드는 과정에서 발생)하고 안전하다.
- IaC는 쉽게 공유할 수 있고, 버전 관리에도 용이(코드로 관리하기에 환경변수 변경)
- 코드와 현재 상태를 비교하여, 추후 인프라 상태의 변경에 따르는 위험을 분석하고 검증할 수 있다.
- 배포 과정을 소수의 시스템 관리자만 진행하는 것이 아닌, 개발자 스스로가 배포하고 인프라를 통제할 수 있다는 것이 강점

## 프로비저닝 vs. 배포 vs. 오케스트레이션

### 프로비저닝
- 시스템, 데이터 및 소프트웨어로 서버를 준비하고 네트워크 작동을 준비
- Puppet, Ansible, Terraform 등과 같은 구성 관리 도구를 사용하여 서버를 프로비저닝할 수 있다
- 클라우드 서비스를 시작하고 구성하는 것을 "프로비저닝"이라고한다.

### 배포
배포는 프로비저닝된 서버를 실행하기 위해 애플리케이션 버전을 제공하는 작업
지속적 배포는(Continuous Delivery) AWS CodePipeline, Jenkins, Github Actions를 통해 구축할 수 있다.

### 오케스트레이션
- 오케스트레이션은 여러 시스템 또는 서비스를 조정하는 작업
- 마이크로서비스, 컨테이너 및 Kubernetes로 작업할 때 일반적인 용어

## Configuration Drift
이미 구축되어있는 인프라 환경에서 만약 프로덕션 레벨에 있는 어떤 특정한 인스턴스를 권한을 갖고 있는 누군가가 실수로 삭제해서 제품에 영향을 미칠 경우 이를 어떻게 알아내고 해결할 수 있는가?

또는 보안 그룹 설정이 도중에 변경이 된다면?

이러한 인프라 변경에 대한 사고와 미치는 영향을 Configuration Drift 라고한다.

## 해결법

### AWS Config & AWS CloudFormation Drift Detection
- AWS에서 제공하는 도구로서 인프라 에러시 이를 감지하고 정보를 반환한다.

## IaC 종류

### 절차형 IaC
프로그래밍 언어를 이용해서 직접 순차적으로 인프라를 생성하도록 코드를 작성하는 방법
선언형에 비해 더 강력한 일들을 할 수 있으나, 실제 적용된 결과를 가늠하기 어렵고, 코드를 읽기에 직관적이지 않다는 단점이 존재한다.

### 절차형 IaC 종류
- Cloud Development Kit(CDK)
- Pulumi
- Ansible

### 선언형 IaC
선언형 언어 JSON, YAML 등을 사용
실제 인프라가 적용된 결과(기대하는 상태)와 적용할 내용(YAML 등)이 직관적으로 매핑

### 선언형 IaC 종류
- CloudFormation
- Blueprint
- Cloud Deployment Manager
- Terraform (어떤 클라우드 서비스에도 적용되는 범용 IaC 도구)