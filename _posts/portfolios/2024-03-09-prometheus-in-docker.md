---
layout: single
title: "Docker compose를 사용하여 Prometheus 모니터링 환경 구축 + jmx exporter"
categories: [Monitoring]
tag: [monitoring, prometheus, grafana, docker, docker compose, springboot, jmx exporter]
published: false
---

<div class="notice">
    <h4> 💡 진행에 사용한 매니페스트 파일은 아래에서 다운받을 수 있습니다. </h4>
    <ul>
        <li>
            <a href="https://github.com/JET-82/monitoring/tree/main/prometheus-in-docker"> Prometheus, Grafana Monitoring in Docker </a>
        </li>
        <li>
            <a href="https://github.com/JET-82/monitoring/tree/main/jmx-exporter"> JMX monitoring in Docker </a>
        </li>
    </ul>
</div>

<br>

## 들어가기 전에
1. **Kubernetes의 metric을 수집하기 위한 Prometheus 환경이 아닙니다.** JMX exporter를 사용해 Springboot 프로젝트를 모니터링 합니다. 
2. Docker container에서 Prometheus와 Grafana를 실행합니다.
3. 마찬가지로 Springboot 프로젝트 또한 Docker container에서 실행됩니다.

<br>

## 실행 환경
### ☁️ 터미널 혹은 Bastion host
Docker를 실행할 터미널 혹은 Bastion host가 필요합니다. 저는 GCP VM을 생성하여 진행했습니다. <span style="color:gray">(GCP는 최초 가입 시, 일정 기간동안 무료 크레딧을 받아서 사용할 수 있습니다.)</span>

|VM 구분|실행 구분|VM 유형|OS|비고|
|:--|:--|:--|:--:|:--:|
|Docker1|Springboot proj|e2-medium (2 vCPU, 1 Core, 4GiB Mem)|CentOS7|고정 IP 주소 사용|
|Docker2|Prometheus, Grafana|e2-medium (2 vCPU, 1 Core, 4 Mem)|CentOS7|고정 IP 주소 사용|

여기서는 Docker2가 Bastion host와 같은 역할을 하게 됩니다.

### 🐳 Docker compose
[v2.24.4 버전](https://github.com/docker/compose/releases/tag/v2.24.4) 사용

### 🕵️ JMX Exporter
[jmx_prometheus_javaagent-0.20.0 버전](https://repo1.maven.org/maven2/io/prometheus/jmx/jmx_prometheus_javaagent/0.20.0/jmx_prometheus_javaagent-0.20.0.jar) (모니터링 테스트 당시 최신 버전)

<br>

## 필요 파일
### Springboot 관련
1. prometheus가 metric 정보를 가져올 **Springboot 프로젝트**
    - 없는 경우, [제가 테스트에 사용한 데모용 프로젝트](https://hub.docker.com/r/ftest5916/team5-deal2/tags)를 `docker pull`하여 사용할 수 있습니다.
    ```shell
    # docker pull
    docker pull ftest5916/team5-deal2:v1.5

    # docker run
    docker run -p 8080:8080 -p 9090:9090 --name springboot ftest5916/team5-deal2:v1.5

    # container 조회
    docker container ls
    ```

2. **Dockerfile:** Springboot 프로젝트를 `docker build`시 필요합니다. JMX exporter를 나타냅니다.

```dockerfile
FROM ${java-base-image}

WORKDIR /app
# 수정 필요
ADD /local/path/jmx_prometheus_javaagent-0.20.0.jar /app/jmx_exporter.jar
# 수정 필요
ADD /local/path/config.yaml /app/config.yaml

ARG JAR_FILE=${springboot_jar}
COPY ${JAR_FILE} .
# 수정 필요
ENTRYPOINT ["java","-javaagent:/app/jmx_exporter.jar=9090:/app/config.yaml","-jar","${springboot_jar}"]
```

3. Dockerfile에서 호출되는 **`config.yaml`**: ~~합니다.


### Promehteus, Grafana 관련

## JMX Exporter란?

## 