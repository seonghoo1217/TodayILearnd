# Intellij와 Git

> 출처 [메타코딩-Youtube](https://www.youtube.com/watch?v=W3RAX9ugrUU)을 보고 정리한 내용이며 본영상에서는 Node.js를 이용한 연동을 강의하기에 구글링을 통하여 영상이외의 필요정보를 얻었습니다.

## intellij와 Git의 연동

Intellij는 IDEA로서 통합개발환경이다. 우리가 Git을 사용하는 이유 중 하나인 나의 기록을 남기는것 또한 Intellij 내부에서 연동하여 사용가능하다.

![intellij-git](../TIL-img/intellij-git.png)

위와 같이 git 내부에서 연동하는방법과 intellij에서 프로젝트 오픈을 git clone 하는 HTTPS주소를 입력하여서 Git 프로젝트를 여는방법도 있다.

## Branch

Git에는 Branch라는 개념이 존재한다.

Branch는 여러 개발자들이 동시에 다양한 작업을 할 수 있도록 도와주는 기능이다. 또한 독립적으로 어떤 작업을 진행하기 위한 개념으로 각각의 브랜치는 서로에게 영향을 끼치지않는다.

![branch](../TIL-img/branch.png)

### Branch의 종류

Branch의 종류에는 배포할 때 사용되는 Main과 실제 개발이되는 Dev가 있다.

#### Branch에 관한 의문점

그렇다면 소규모 프로젝트의 인원이 4명이라고 가정하였을 때 모든 개발자가 Dev에 commit,push를 하게되면 되는것일까?

정답은 **"아니다"** 이다.

만약 당신이 팀장이라고 가정하였을 때 팀원들의 실제 개발 소스코드가 하나의 Branch에서 섞이게 되는경우 이를 Code Review 하고 문제점 발견시에 Rebase를 지시할 때 작업은 상당히 까다로워 질것이다. 

#### 그렇다면 이런 문제점은 어떻게 해결해야 좋을까?

**1. 팀장의 경우**

당신이 팀장이라면 개발단계에선 Main과 Dev 모든 Branch를 보호 상태에 두고 팀원들에게 몇가지를 지시하여야한다.

1.Topic Branch를 생성할것.
2.개발진행하고 로그 rebase 한후에 Topic을 push 시킬 것. 
3.Dev Branch에 pr요청을 하여 팀장에게 comment 날릴 것.
4.팀장은 확인 후 rebase 요청시에는 Request changes로 반환시키고 코드 이상이없다면 Approve후 comment 날릴 것.
5.승인된 상태라면 팀장,팀원중에 한명이 merge 시키기.
6.merge 완료된 상태라면 기존 branch 삭제 시키기.
7.Dev에 최종적으로 pull 시키기

**2. 팀원의 경우**

팀원일 경우 Topic을 생성하여 Dev에 pr요청을 보내야한다.
또한 코드 작성후 push전에 rebase 할 부분은 수정

pr요청시에 문서작업 필요시 문서작성
