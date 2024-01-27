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
JDK는 nGrinder에 맞추어서 11로 설정해두었다. (혹시 이것 때문에 뻑나면 안되니깐….)

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

### (테스트) 로컬 서버 접속 및 스크립트 확인용
상품 정보를 조회하는 API를 테스트 삼아서 진행해보긴 했다 (돌아가는지만 확인)
    
![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/8ad30eb7-8510-4c7b-963a-6ccb51e8f0bf/Untitled.png)
    
![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/0e92e777-c2b9-4870-bfa3-8f43b5e4b4af/Untitled.png)