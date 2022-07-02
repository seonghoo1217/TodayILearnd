# E: Sub-process /usr/bin/dpkg returned an error code (1) 에러

* 혹시라도 ubuntu 환경의 리눅스 셸 에서 apt install 명령어를 입력하였을때 이런 에러가 난경우가 있다면 이글이 도움이 될것이다.

이 에러의 경우 dpkg 및 apt install 기능이 모두 죽게된다. 따라서 install 자체가 작동하지않는다.

install 자체가 실행되지 않음으로 인해서 패키지들간에 의존성 문제가 발생하게 되어 이를 해결해야 apt install 기능을 사용할 수 있다.


## 해결방법

해결방법은 의외로 간단하다

첫번째로 에러가 나는 위치인 dpkg 경로 밑에 있는 info 디렉터리에 담긴 모든 파일을 제거하여준다.

그 다음으로 --configure -a 옵션으로 dpkg 명령을 실행하여주면 정상작동 된다. 코드는 아래와 같다


```markdown
sudo rm /var/lib/dpkg/info/*
sudo dpkg --configure -a
sudo apt update -y
```
