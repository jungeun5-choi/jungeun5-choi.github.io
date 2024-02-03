---
layout: single
title: "nGrinder; 테스트에 사용할 groovy 프로젝트 생성"
categories: [nGrinder]
tag: [ngrinder, stress test, groovy]
---

## 🐛 groovy 설치
groovy로 작성한 스크립트를 사용하고 관리해야하니까, groovy 프로젝트를 만들어야하지 않을까? 라는 생각이 들어서 groovy 설치부터 해보기로 했다.

**설치 버전:** v4.0.14
{% include link.html
    url="https://groovy.apache.org/download.html"
    title="The Apache Groovy programming language - Download"
    description="Ways to get Apache Groovy: ..."
%}
nGrinder와 호환되는 groovy 버전을 찾아봤는데, 별 말은 없었기 때문에 가장 최신 버전으로 설치해주었다.


파일은 .zip 파일로 다운되고, 내가 직접 원하는 위치에 다운을 받아야한다. 나는 Java SDK가 `C:/Program Files/` 경로에 있었기 때문에 groovy도 동일하게 이 곳에 압축을 해제시켜두었다.

**환경변수 설정**은 이 쪽 글을 참고하였다. 2007년 글이지만, 환경변수 설정은 지금에도 큰 차이점이 없기 때문에 설정에는 무리가 없었다.
{% include link.html
    url="https://javaworld.co.kr/4"
    title="Groovy 처음 시작하기 : 설치"
    description="자바월드"
%}

환경변수 설정까지 완료하면, cmd 창에서 `groovy -version` 명령어로 설치를 확인할 수 있다. (Java JDK까지 설치되어있어야 조회가 가능 주의할 것!)

```powershell
> groovy -version

# response
Groovy Version: 4.0.14 JVM: 17.0.7 Vendor: Eclipse Adoptium OS: Windows 10
```

## 📁 IntelliJ에서 groovy 프로젝트 생성

<div class="notice">
    <h4> 💡 참고: </h4>
    <ul>
        <li>
            <a href="https://peterica.tistory.com/175"> [Intellij] IntelliJ에서 Groovy 환경구성하기 </a>
        </li>
        <li>
            <a href="https://xzio.tistory.com/1507"> IntelliJ에서 Groovy 프로젝트 생성하고 테스트 코드 실행하기 </a>
        </li>
    </ul>
</div>

<div class="notice">
<h4> 🔧 환경 설정 </h4>
    <ul>
        <li> <b>Java:</b> JDK 11 </li>
        <li> <b>Groovy:</b> v4.0.14 </li>
        <li> <b>Build:</b> Gradle </li>
    </ul>
</div>
JDK는 nGrinder에 맞추어서 11로 설정해두었다. (혹시 이것 때문에 괜히 오류가 나면 안되니깐….)

###### build.gradle
다른 의존성들은 오히려 오류를 불러올 수 있다는 글을 발견해서 전부 지우고 이 둘만 추가해주었다.
```java
dependencies {
    // groovy
    implementation 'org.apache.groovy:groovy:4.0.14'
    // ngrinder
    implementation 'org.ngrinder:ngrinder-groovy:3.5.8'
}
```

### .groovy 테스트 파일 생성 순서

1. main과 test 디렉토리 중 `test` 디렉토리에 파일을 생성한다.
2. New > Groovy Script 선택 (GroovyDSL Script 아님)
3. 생성 후 테스트 스크립트 작성

#### (테스트) 로컬 서버 접속 및 스크립트 확인
상품 정보를 조회하는 API를 테스트 삼아서 진행해보긴 했다 (돌아가는지만 확인)

![로컬서버접속및스크립트확인-1]({{site.url}}/images/2023-08-25-groovy-project/로컬서버접속및스크립트확인-1.png){: width="80%" height="80%"}

![로컬서버접속및스크립트확인-2]({{site.url}}/images/2023-08-25-groovy-project/로컬서버접속및스크립트확인-2.png){: width="80%" height="80%"}


## 😵 환경변수 설정
### IntelliJ에서: local hostname 환경변수 설정

내 ip 주소라는 민감한 정보를… github에 바로 올릴 순 없으니 환경변수로 설정하려고 한다. `NGRINDER_HOSTNAME`이라는 이름으로 환경변수를 설정해주었다. 환경 설정 시 등록한 IP 주소는 **window host ip(IPv4 주소)**이다.

- Run Configuration 혹은 Edit Configuration → Environment variables
![환경변수-intellij]({{site.url}}/images/2023-08-25-groovy-project/환경변수-intellij.png){: width="80%" height="80%"}

###### src/test/groovy/GetItem.groovy
```groovy
@RunWith(GrinderRunner)
class GetItem {
    /* 생략 */

    public static HOSTNAME = System.getenv("NGRINDER_HOSTNAME");

    /* @BeforeProcess */
    /* @BeforeThread */
    /* @Before */

    @Test
    public void test() {
    HTTPResponse response = request.GET("http://${NGRINDER_HOSTNAME}:1010/tour-ranger/items/1")
        // ...
    }
}
```
참고로, hostname 뒤의 포트 번호는 내가 포트포워딩 해준 포트이다.

<br>

### 환경변수 설정 후, 테스트 스크립트 실행 시 발생하는 오류
#### 1. RuntimeException: Please add -javaagent:{file dir} in ‘Run As JUnit’ vm argument

스크립트 작성 후, 실행하면 이런 오류가 뜨는데,
```
java.lang.RuntimeException: Please add 
-javaagent:C:\Users\MY%20PC\.gradle\caches\modules-2\files-2.1\net.sf.grinder\grinder-dcr-agent\3.9.1\37607dc5d7192b652e5dec8394a0334b4d3a63d2\grinder-dcr-agent-3.9.1.jar
in 'Run As JUnit' vm argument.
```
Edit Configuration > VM options에 `-javaagent:${파일경로}` 부분을 입력해주면 된다.

#### Error opening zip file or JAR manifest missing : ${file dir}
간혹 위의 오류를 해결해도 아래의 오류가 뜨는 경우가 있다.
```
Error occurred during initialization of VM
agent library failed to init: instrument
Error opening zip file or JAR manifest missing : C:\Users\MY%20PC\.gradle\caches\modules-2\files-2.1\net.sf.grinder\grinder-dcr-agent\3.9.1\37607dc5d7192b652e5dec8394a0334b4d3a63d2\grinder-dcr-agent-3.9.1.jar

Process finished with exit code 1
```

세 가지의 가능성이 있는데, 나의 경우는 **2번**이 문제였다.

1. 해당 파일에 접근 권한 문제
2. **파일 경로(이름) 문제 (특수문자나 공백 등이 포함된 경우) ⬅️**
3. 파일이 존재하지 않는 경우

WSL에서 스크립트를 실행시켜보거나, 환경변수를 조작해보는 등, 온갖 해결방법은 다 해보았는데… `grinder-dcr-agent-3.9.1.jar` 파일을 E 드라이브에 복사하는 것으로 해결되었다.

![grinder-dcr-agent-391-jar]({{site.url}}/images/2023-08-25-groovy-project/grinder-dcr-agent-391-jar.png){: width="80%" height="80%"}

이러면 경로가 `E:\grinder-dcr-agent-3.9.1.jar`로 단순해지고, 권한문제에도 걸리지 않는다.


### Ubuntu에서:local hostname 환경변수 설정
나의 경우, nGrinder는 ubuntu 환경에서 실행되기 때문에 ubuntu에 환경변수를 설정해줘야한다. (groovy 스크립트도 ubuntu 상에서 실행됨)

#### 1. 단발적인 환경변수 설정

ngrinder 실행 직전에 한 번씩 해주면 된다. 단, 우분투 창이 꺼지면 초기화되기 때문에 매번 해줘야한다.

```shell
$ export NGRINDER_HOSTNAME=${window-hostname-ip}
```

#### 2. 영구적인 환경변수 설정
Ubuntu의 기본값인 Bash 셸의 경우 프로필 파일은 일반적으로 `~/.bashrc`이다. `~/.bashrc` 파일에 내보내기 명령을 추가한 다음, `source` 명령어를 사용하면 다시 로드되어 변경 사항이 현재 세션에 적용된다...고 한다.

```shell
$ echo 'export NGRINDER_HOSTNAME=${window-hostname-ip}' >> ~/.bashrc
$ source ~/.bashrc
```

#### 3. 환경변수가 설정되었는지 확인하는 방법
```shell
$ echo $NGRINDER_HOSTNAME

# response
${window-hostname-ip}
```

## 🖇️ GitHub-nGrinder 연결
### .gitconfig.yml
nGrinder GUI는 github repository에 접근할 수 있는데, `.gitconfig.yml` 파일이 그 이어주는 역할을 한다. (참고: [공식문서](https://github.com/naver/ngrinder/wiki/GitHub-Script-Storage))

##### 작성 방법
```
name: Tour-Ranger-Test-Runner                   # nGrinder에서 표시할 이름
owner: Tour-Ranger                              # repository 소유자 github 이름
repo: Tour-Ranger-Test-Runner                   # github repository 이름
access-token: ${github-personal-access-token}   # 아래 액세스 토큰 생성 참고
branch: feature/tour-ranger                     # 특정 branch에만 올라간 경우
#script-root: tour-ranger-test-runner           # 폴더가 나뉘어있을 경우 경로를 알려준다
```

단, Repository 공개 설정이 Public일 때만 스크립트를 찾을 수 있다.
![gitconfig-yml]({{site.url}}/images/2023-08-25-groovy-project/gitconfig-yml.png){: width="80%" height="80%"}