Git commit —amend -m “CommitMessage” 최신 커밋 수정하기

amend를 사용하여 커밋을 수정할 경우 새로운 커밋이 생성

amend 사용후 remote repository에 커밋을 시도할 경우  충돌이 일어난다. 그렇기에 push —force를 사용 이로인한 동기화 문제 유발가능성

git diff HEAD 로 변경사항 확인

새로운 기능에 대해 remote repository에 부분적으로 반영하기 위해 branch라는 단위를 사용

**branch**

- 내 수정 사항을 특정 공간에 고립시켜 다른 사용자의 변경사항과 섞이지 않도록 만듦
- 이를 통해, 서로 간에 영향이 없게 만들어 줌
- 나무에서 가지가 뻗어나가는 현상을  branch라고 한다.
- 중심이 되는 나뭇가지 == Main Branch라고한다.
- branch는 특정 commit을 가르키는 단순한 포인터 역할

Log 확인 시 log를 하나의 line을 출력

- git log —oneline

내가 현재 어떤 branch에서 작업하는지 알기위해 HEAD라는 개념 사용

HEAD는 branch에 대한 포인터이며, checkout 명령어 사용 시 포인터가 가리키는 branch가 변경

commit 을 새로이 수행할 경우 Main branch의 HEAD 포인터는 옮겨감

각 branch는 특정 commit의 포인터 그 이상 그이하도 아님

**branch의 생성**

git branch [생성할 branch이름] [branch가 가리킬 commitID]

독립된 branch에서 작업한 내용을 다른 branch로 통합하는 것을 **Merge**라고 함

병합이란 서로 다른 branch의 changeset을 결합하는 새로운 commit을 만드는 작업

**Merge의 종류**

1. Fast-Forward Merges 병합하려는 branch들 사이에 분기가 존재하지 않을 경우 사용

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/30db4385-3ec9-43a6-a884-8adb2abb87bc/Untitled.png)

브랜치 삭제 방법

- git branch -D [branch 이름]

브랜치 Fast-Forward 병합

- git merge [병합하려는 대상 branch명]

병합하려는 branch에 존재하는 changeset들이 특정 branch에만 존재하면 Three-way Merge고 이미 main branch에 존재한다면 fast-forward

three-way-merge

- 여러명의 개발자들이 하나의 base에서 여러 branch

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/4f6a9a5d-811a-4591-b9ce-b6229ecf177a/Untitled.png)

Three-way Merges시에는 새로운 커밋이 생성되게된다.

- Log 출력 시 graph 형태로 출력
  • git log --oneline --graph
  – Log 출력 시 모든 branch의 내용을 출력
  • git log --oneline --all
  – Log 출력 시 모든 branch의 내용을 출력하면서, 분기를 graph로 표현
  • git log --oneline --all --graph

**Merge Conflicts**

- Merge를 수행하는 과정에서 분기가 발생한 base를 기준으로두 branch에 동시에 존재하는 file의 특정 내용을 서로 다른 내용으로 수정한 경우 git에 의해 자동으로 merge가 되지 않음
- Git 입장에서 소스코드의 의미(Semantic)를 알지 못하므로, 어느 것을 선택해야 할지 판단을 할 수 없음
- 단, conflict가 발생하지 않기 위해서는 변경이 발생한 부분과 최소 한 줄 이상 떨어져야 함

Fast-Forward Merges에서는 merge conflict가 일어날 수 없다.

**Merge Conflicts 해결방법**

가장 간단한 방법은 merge를 수행하는 사람이 직접 두 변경사항 중 적절한 것을 선택하고, 필요 시 추가적으로 수정하여 git에 반영하는 것
– Merge conflict가 발생한 경우, git에서 marker를 해당 파일에 추가하여 서로 다른 변경사항을 표시함
– 이를 바탕으로, 서로 다른 변경사항을 적절히 합쳐서 conflict를 해결해야 함

Rebase 중요커밋을 최신커밋의 앞에 끼워넣을 경우 사용하는 명령어

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/3957f8f3-4a3c-4e3b-9a44-542c79a8c656/Untitled.png)

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/ac19f4cf-4e0d-41ba-8a69-23ad124f7304/Untitled.png)

**Squash and Merge**

임시 커밋들을 하나의 commit으로 만들어 merge 해주는 것을 merge —squash라고 하며 이전 commit들은 log에서 확인되지 않는다.

squash merge는 joint commit이아닌 완전히 새로운 별도의 commit을 생성

**Merge와 Conflict**

- Fast-Forward Merge외의 Merge는 모두 Conflict를 유발할 수 있음
- Merge 과정에서 사용자가 conflict를 직접 해결해야함

Local Repository에서 clone을 수행하면 remote repository를 origin이라는 이름으로 참조하게된다.

- Clone 직후에는 workspace가 clean 상태이며, main branch는 remote와 동일한 상태를 갖고 있음
- git log 명령어를 통해 clone 할 당시 시점의 origin(==remote repository)의 main branch의 commit을 확인 가능

Local Repository에서 작업한 결과를 remote repository에 upload(동기화) 하는 방법은 git push

git push 시점에 origin과 local의 상태를 비교

만약 내가 local에서 작업하는 동안 다른 개발자가 origin에 커밋하는 경우 remote의 실제 main branch가 가르키는 commit은 변경되지만, 본인의 local의 origin main은 유지

이럴경우 **동기화**를 거쳐야함

git fetch로 remote의 최신 포인터 상태를 가져올 수 있다. 하지만 변경된 작업 내용까지 가져오지는 않으므로 파일까지 가져오고 싶을땐 git merge origin을 실행한다.

git rebase의 경우 HEAD가 가리키는 branch와 <target branch>의 base를 기준으로 HEAD가 가리키는 branch에 생성된 commit들을 <target branch>의 commit 뒤로 이동

local branch 삭제는 git branch -D  [branch name]

remote의 branch를 삭제하기 위해선 git push origin —delete [branch name]

git push의 경우 branch 단위로 동기화가 진행

git fetch의 경우 모든 branch에 대하여 새로운 내용을 update 수행

기본적으로 동기화를 시도할 때 Fast forward merge를 시도한다. 이때 merge가 되지 않으면 push는 실패 다시 push연산으로 동기화 해야함 즉,Fast-forward merge가 되도록 상태를 만들어주는 것

동기화 실패시 해결방법 두가지

1. origin의 main을 local의 main으로 merge 수행 그렇게 three way merge 실행한 후 push
2. git fetch후 origin main을 rebase한후 push

git pull= git fetch + git merge origin/main 오로지 main의 내용만 읽어온다.

⇒ conflict를 유밞할 수 있고 현재 작업 중인 내용에 문제가 생길 수 있으므로 사용하지 않는것을 권장

**협업 방법론**

- 메인 기반 워크플로우

⇒ 개발자가 새로운 기능을 개발할 때 branch를 생성해서 작업을 수행한 후 기능 개발이 완료된 후에 local repository의 main branch로 병합 후, 이를 remote repository와 동기화하는 방식

⇒ remote repository와 동기화 실패 시 merge 수행 후 다시 동기화해야함

- 풀 리퀘스트 기반 워크플로우

⇒ github를 통해 remote repository의 feature branch에서 직접 main branch로 병합 수행

⇒ 병합을 위해 다른 사용자로부터 변경내용에 대해 코드 리뷰를 받고 지정된 사람들로 부터 승인이 필요

⇒ remote repository에서 merge 이후에 remote repository의 main과 local의 main은 서로다름

풀리퀘스트에서의 squash and Merge와 rebase and merge의 동작방식을 확인할 수 있다.

풀 리퀘스트에서의 rebase

- 풀 리퀘스트에서의 rebase는 feature에 있는 commit을 main으로 rebase를 수행한다.

Local에서 main을 타겟으로한 feature의 리베이스

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/883228bc-5de7-48d3-b8d9-882957682e9e/Untitled.png)

풀 리퀘스트에서의 rebase 사용방식

- 풀리퀘스트는 feature branch의 commit들을 main에 똑같이 복사하는 방식으로 rebase를 수행한다.
- rebase and merge 이후 feature branch는 삭제한다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/60c62fed-65f3-425a-ab00-4ee77a1d583d/Untitled.png)

실제로 local 에서 rebase 수행 시, base 이후에 feature에서 생성된 commit들은 새로운 base 이후에 복제되는 형태로 처리

복제 후에 기존 commit들은 feature branch에서 보이지 않음

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/249076e0-1e80-4c5c-a489-3f45ae6633d0/Untitled.png)

rebase and merge 이후 branch를 삭제해야하는 이유

- 동일한 commit들을 또 복제 후 rebase를 시도하여 conflict를 일으킨다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/c7b62716-3801-4f62-932f-cecf0c2c613e/Untitled.png)

풀 리퀘스트 기반 squash merge는 똑같다.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/8ea043fb-38bb-483a-9abc-21327a6b35f1/Untitled.png)

메인기반 워크플로우 vs 풀 리퀘스트 기반 워크 플로우

- 메인기반 워크플로우
    - 협업을 위한 절차가 매우 간편
    - 개인 개발자가 직접 feature branch의 내용을 main에 merge할 수 있어 작업 진행이 빠르다.
    - 단, 개인 개발자가 merge하는 과정에서 다른 사람의 작업 내용을 잘못 수정하여 문제를 일으킬 가능성이 존재한다.
- 풀 리퀘스트 기반 워크플로우
    - 메인 기반 워크플로우에 비해 협업을 위한 절차가 추가된다.
    - Feature Branch의 개발내용을 리뷰 후 merge 할때까지 시간이 길어져 전체적인 개발 시간이 길어질 수 있다.
    - merge과정에서 다른사람의 승인이 필요하여 메인 기반 워크플로우에 비해 문제가 발생할 가능성을 줄일수 있다.

특정 commit을 가리키는 ID ⇒ revision

origin /Head는 origin의 default branch를 뜻한다.

origin /main은 clone시점의 remote의 main branch를 뜻함

remote에서 삭제된 branch 정보를 local에서도 동기화 : git fetch —prune

remote의 포인터를 최신화 : git fetch

local에서 branch 삭제 : git branch -D branchName

local에서 remote의 branch삭제 git push origin —delete branchName