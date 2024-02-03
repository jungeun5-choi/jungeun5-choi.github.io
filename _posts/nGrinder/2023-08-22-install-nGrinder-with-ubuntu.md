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

### 1. Ubuntu에 jdk 설치
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

### 2. Ubuntu에 nGrinder 설치
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

파일 경로에서도 진행하면서 다운받은 파일들을 확인해볼 수 있다.
![complete-download-files]({{site.url}}/images/2023-08-22-install-nGrinder-with-ubuntu/complete-download-files.png){: width="80%" height="80%"}

<br>

## 🦀 localhost 접속 문제 해결
### 1. 발생한 문제
테스트용 groovy 스크립트에서 localhost를 접속하려고 하면 아래와 같은 Connection Error가 발생.
```
2023-08-23 02:49:52,314 ERROR Cannot invoke method GET() on null object
java.lang.NullPointerException: Cannot invoke method GET() on null object
	at TestRunner.test(createProduct2.groovy:40)
	at java.base/jdk.internal.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at java.base/jdk.internal.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at java.base/jdk.internal.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at net.grinder.scriptengine.groovy.junit.GrinderRunner.run(GrinderRunner.java:164)
	at net.grinder.scriptengine.groovy.GroovyScriptEngine$GroovyWorkerRunnable.run(GroovyScriptEngine.java:147)
	at net.grinder.engine.process.GrinderThread.run(GrinderThread.java:118)
```

### 2. 해결 방안
1. portproxy 추가
2. 접근 url을 local hostname → window hostname으로 변경
3. 테스트 실행

#### portproxy 추가
{% include link.html
    url="https://learn.microsoft.com/ko-kr/windows/wsl/networking#accessing-a-wsl-2-distribution-from-your-local-area-network-lan"
    title="LAN(Local Area Network)에서 WSL 2 배포에 액세스"
    description="WSL을 사용하여 네트워크 애플리케이션에 액세스"
%}

아래의 명령어를 cmd 혹은 powershell에서 <u>관리자 모드</u>로 실행시킨다.
```shell
> netsh interface portproxy add v4tov4 listenport=<yourPortToForward> listenaddress=0.0.0.0 connectport=<yourPortToConnectToInWSL> connectaddress=(wsl hostname -I)

# 예시
> netsh interface portproxy add v4tov4 listenport=4000 listenaddress=0.0.0.0 connectport=8080 connectaddress=172.xxx.x.x
```
`portproxy`가 추가되었는지는 아래의 명령어를 통해 확인이 가능하다.
```shell
> netsh interface portproxy show all
```
![wsl-hostname]({{site.url}}/images/2023-08-22-install-nGrinder-with-ubuntu/complete-download-files.png){: width="70%" height="70%"}

! 테스트에 사용할 nGrinder의 port 번호를 `1010`으로 설정할 것이 아니라면, `8080`만 추가하는게 옳다. 중복으로 지정해두면 오히려 충돌이 일어날 수 있다. -- 내 프로젝트에서는 `1010`을 프로젝트 로컬 포트로 지정해주었다.


#### 접근 url을 local hostname → window hostname으로 변경
nGrinder 테스트 코드에서 접근 주소 부분을
```groovy
@Test
public void test() {
    HTTPResponse response = request.GET("http://127.0.0.1:1010/products", params)
```
```groovy
@Test
public void test() {
    HTTPResponse response = request.GET("http://${window-hostname}/products", params)
```

#### 테스트 실행
- 테스트 성공: 무사히 초록불이 들어왔다
![test-complete]({{site.url}}/images/2023-08-22-install-nGrinder-with-ubuntu/test-complete.png){: width="100%" height="100%"}

- Ubuntu에서도 `window hostname`으로 접근 시, `GET` 요청이 문제없이 실행된 것을 볼 수 있었다.
![curl-hostname]({{site.url}}/images/2023-08-22-install-nGrinder-with-ubuntu/curl-hostname.png){: width="100%" height="100%"}
