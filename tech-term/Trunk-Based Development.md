## 소개 배경
나는 평소, `github branching stategy`를 준수하며, branch 관리 및 PR 방법을 사용하였다.

![img.png](../TIL-img/gitbranching.png)

이슈마다 feature 브랜치를 생성 후 마스터 브랜치에 merge 하는 방식이다.

경우에 따라서 여기에 develop 브런치를 생성하여 개발환경과 배포환경을 분리시키거나, release 브런치를 추가로 만들어 관리하는 경우가 있다.

하지만, 최근들어 `Trunk-Based Development` 즉, 트렁크 기반 개발에 관심을 가지게된 것은 기존 github branching stategy의 경우 절차가 꽤나 많기에 복잡하며, 통합되는 과정까지 시간이 너무 많이 소요된다고 생각한다.

우선 대표적인 GitHub WorkFlow를 살펴보자면 다음과 같다.

1. 개발할 기능을 선택한다.
2. 해당 기능에 대한 새로운 브랜치를 main을 기준으로 생성한다.
3. 개발을 진행하는 과정에서 `commit`과 `push`를 진행하며 history에 내역이 쌓인다.
4. 이후 기능 개발이 완료되었다면 `PR(Pull Request)`를 생성한다.
5. PR이 생성되면, 코드에 대해 오너 혹은 리뷰어가 코드를 검토 후 Merge or Reject을 결정한다.
6. 리뷰가 반복됨에 따라 PR의 수가 계속해서 증가한다.
7. 더 이상 리뷰어가 의견이 없으면 PR이 Merge된다.

이 과정만 보더라도, 개발이 완료되고 해당 사항이 배포환경으로까지 승인되는 과정은 상당히 길고,
이 외에도 Branch의 분리로 인해 이를 PR을 통해 코드를 통합하는 과정에서 상당히 많은 에러와 이슈(conflict)를 유발할 가능성이 높은 방법론이라는 것이다.

또한, 피드백이 늦어지게 된다면, 개발의 속도가 늦어지고 만약 크리티컬이라면 추후 코드와 이전코드에도 모두 영향을 끼치게된다. 만약 핫픽스 이슈가 생긴다면,
이는 상당한 단점으로 작용하게된다.

그래서 이러한 한계점을 극복하고자 브랜칭 모델 중 하나인, 트렁크 기반 개발이 필요한 것이다.

## Trunk-Based Development
`Trunk-Based Development(트렁크 기반 개발, TBD)`란 모든 개발자가 **트렁크(trunk or master)**라는 단일 브랜치에서 직접 모든 작업을 하는 것이다.

해당말이 조금 모호하게 느껴질 수도 있다. **"🤔 그냥 Main 브랜치에서 모든 작업 다하는거 아닌가?"** 라고 생각할 수 있다.

하지만, TBD가 지켜지기 위해선 몇가지 절대적인 규칙이 필요로된다.

### 1. Pair Programming or Mob Programming
페어 프로그래밍(Pair Programming)이란, 두 명의 개발자가 한 곳에서 한 명은 코드를 작성하고 한 명은 각 코드행을 검토하는 방식을 말하며,
이 둘 사이의 역할은 자주 전환되는 프로그래밍을 말한다. 

이 방식을 따르면 작성 중인 코드가 실시간으로 검토되기 때문에 나중에 PR 요청이 필요하지 않다. 

또한 작업은 더 빨리 완료되며, 더 높은 품질, 코딩 스타일 통일, 지식 공유, 문제 해결에 대한 해결책을 같이 찾음 등의 이점이 있다. 

### 2. 신뢰할 수 있는 빌드가 필요
코드 베이스가 항상 릴리즈가 가능한 상태임을 확신할 수 있어야 한다. 그러기 위해서는 충분한 자동 테스트가 실행되어야 하며, 그렇기 때문에 효과적이고 우수한 테스트 전략이 필요하다.

