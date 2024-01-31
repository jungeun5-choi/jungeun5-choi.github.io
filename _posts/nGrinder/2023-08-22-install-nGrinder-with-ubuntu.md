---
layout: single
title: "nGrinder; Ubuntu Linux에 nGrinder 설치"
categories: [nGrinder]
tag: [ngrinder, ubuntu, linux]
---

원래는 간단하게 Docker에 설치했었는데, 필수 파일이 삭제되는 문제가 발생해서 리눅스에 설치해 보기로 했다.

## Windows10에서 Ubuntu 설치
**설치 버전:** ubuntu v20.04.6 LTS (이거 아니어도 v18.04 등 LTS 버전이면 가능)

{% include link.html
    url="https://webdir.tistory.com/541"
    title="WEBDIR::윈도우즈에서 리눅스 설치 - WSL"
    description="윈도우즈10에서 WSL 설치방법에 대하여 알아봅니다."
%}
Windows10에서는 별다른 문제없이 설치가 잘 되는데, Windows11에서는 추가적으로 설정을 더 해줘야한다. (복잡함...)

### Ubuntu에 jdk 설치
jdk 1.8 혹은 11만 지원한다는 이야기가 있어서 jdk 11로 설치했다.
```shell
# sudo 업데이트
$ sudo apt update
# jdk 11버전 설치
$ sudo apt install openjdk-11-jre-headless
```
###### jdk 설치 확인
```shell
# jdk 설치 확인
$ java -version

#response
openjdk version "11.0.20" 2023-07-18
OpenJDK Runtime Environment (build 11.0.20+8-post-Ubuntu-1ubuntu120.04)
OpenJDK 64-Bit Server VM (build 11.0.20+8-post-Ubuntu-1ubuntu120.04, mixed mode, sharing)
```

### Ubuntu에 nGrinder 설치
##### ngrinder-controller 설치
```shell
# 파일 다운로드
# 직접 github에 방문하여 최신 버전 설치를 권장.
$ wget https://github.com/naver/ngrinder/releases/download/ngrinder-3.5.8-20221230/ngrinder-controller-3.5.8.war
```
```shell
# 실행
$ java -jar ngrinder-controller-3.5.8.war --port=8080
```
설치 및 실행이 끝나면,
1. `http://localhost:8080/login`를 입력해 ngrinder 콘솔로 접근
2. ID: `admin`/ PW: `admin` 으로 로그인한다.

##### ngrinder-agent 설치
controller를 실행한 창은 실행 후에는 더이상 명령문을 입력받지 않아서 새로운 Ubuntu 창을 켜서 진행했다.
```shell
# 파일 다운로드
# 정확한 경로는 [Agent Management] 페이지 > download 경로 확인 가능
$ wget http://localhost:8080/agent/download/ngrinder-agent-3.5.8.tar

# 파일 압축 해제
$ tar -xvf ngrinder-agent-3.5.8.tar
```
```shell
# ngrinder-agent 폴더로 이동 후,
$ cd ngrinder-agent

# run_agent.sh 실행
~/ngrinder-agent$ ./run_agent.sh
```

##### agent가 성공적으로 추가된 모습
![complete-install-ngrinder-agent]({{site.url}}/images/2023-08-22-install-nGrinder-with-ubuntu/complete-install-ngrinder-agent.png){: width="80%" height="80%"}

![complete-download-files]({{site.url}}/images/2023-08-22-install-nGrinder-with-ubuntu/complete-download-files){: width="80%" height="80%"}

## localhost 접속 문제 해결