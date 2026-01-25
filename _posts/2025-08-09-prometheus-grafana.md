---
title: Prometheus + Grafana 적용기
author: dnya0
date: 2025-08-09 06:19:00 +0900
categories: [Study, Monitoring]
tag: [Study, Grafana, Prometheus, Monitoring, AWS]
---

## 배경

서버에 올리고 CPU가 급격히 증가하여 멈추는 사태가 발생하였다. t2 micro에서 t2 small로 인스턴스를 변경하면서 해당 이슈를 해결하였다. 이 이슈로 모니터링 툴의 필요성을 느꼈고, 앞으로 서버에 문제가 생길 경우 Scale Up이 필요한지 판단하기 위해 모니터링 툴을 도입하기로 결심하였다. AWS CloudWatch와 Prometheus 중 Prometheus를 선택하였는데 AWS CloudWatch는 비용이 부과되기 때문에..

<br>

## Prometheus 설치

```shell
$ sudo yum update
$ https://github.com/prometheus/prometheus/releases/download/v3.5.0/prometheus-3.5.0.linux-386.tar.gz
$ tar xvf prometheus-3.5.0.linux-386.tar.gz
```

설치 확인

```shell
$ cd prometheus-3.5.0.linux-386/
$ ./prometheus --version
```

명령어 등록

```shell
$ echo 'export PATH=$PATH:/home/ec2-user/prometheus/prometheus-3.5.0.linux-386' | sudo tee -a /etc/profile
$ source /etc/profile
$ prometheus --version #동작 확인

```

systemctl에 등록하기 전 그룹 생성

```shell
$ sudo groupadd prometheus
$ sudo usermod -aG prometheus ec2-user
```

systemctl에 등록하기 위해 파일 생성

```shell
sudo vi /etc/systemd/system/prometheus.service
```

파일에 해당 내용 입력

```bash
[Unit]
Description=Prometheus
After=network-online.target

[Service]
User=ec2-user
Group=prometheus
Type=Simple
ExecStart=/home/ec2-user/prometheus/prometheus-3.5.0.linux-386/prometheus --config.file=/home/ec2-user/prometheus/prometheus-3.5.0.linux-386/prometheus.yml
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
```

재기동 및 실행, 상태확인

```shell
$ sudo systemctl daemon-reload
$ sudo systemctl start prometheus
$ sudo systemctl enable prometheus
$ sudo systemctl status prometheus
```

<br>

## prometheus-node-exporter 설치

```shell
$ wget https://github.com/prometheus/node_exporter/releases/download/v1.9.1/node_exporter-1.9.1.linux-amd64.tar.gz
$ tar xvf node_exporter-1.9.1.linux-amd64.tar.gz
```

명령어 등록

```shell
$ echo 'export PATH=$PATH:/home/ec2-user/prometheus-node-exporter/node_exporter-1.9.1.linux-amd64' | sudo tee -a /etc/profile
$ source /etc/profile
$ node_exporter --version #동작 확인
```

systemctl에 등록하기 위해 `prometheus-node-exporter.service` 생성

```
$ sudo vi /etc/systemd/system/prometheus-node-exporter.service
```

파일에 해당 내역 입력

```bash
[Unit]
Description=Node Exporter
Wants=network-online.target
After=network-online.target

[Service]
User=ec2-user
Group=prometheus
Type=simple
ExecStart=/home/ec2-user/prometheus-node-exporter/node_exporter-1.9.1.linux-amd64/node_exporter
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
```

재기동 및 실행

```shell
$ sudo systemctl daemon-reload
$ sudo systemctl start prometheus-node-exporter
$ sudo systemctl enable prometheus-node-exporter
```

<br>

## Grafana 설치

```shell
$ sudo tee /etc/yum.repos.d/grafana.repo <<EOF
[grafana]
name=Grafana
baseurl=https://packages.grafana.com/oss/rpm
enabled=1
gpgcheck=1
gpgkey=https://packages.grafana.com/gpg.key
EOF
$ sudo yum install grafana
```

서비스 시작

```shell
$ sudo systemctl start grafana-server
$ sudo systemctl enable grafana-server

$ grafana-server -v #설치 확인
```

<br>

## 포트 개방

ec2 보안 그룹 인바운드에서 `9090`, `3000`, `9100`번의 포트를 열어주면 된다.

<br>

## SpringBoot에 Prometheus 추가

```kotlin
implementation("org.springframework.boot:spring-boot-starter-actuator")
implementation("io.micrometer:micrometer-registry-prometheus")
```

```yaml
management:
  endpoints:
    web:
      exposure:
        include: health,info,prometheus
  endpoint:
    prometheus:
      enabled: true
```

그리고 나는 security filter에 걸릴까봐 actuator로 시작하는 엔드포인트는 permitAll 해줬다.

<br>

## Docker로 변경

그런데 열심히 설치해서 실행시켜놨더니 Docker 컨테이너로 설치하는 방법이 있었다.. 난 이미 Docker 기반으로 서버를 운영하고 있었기 때문에 Docker 방식으로 바꾸기로 결정하였다.

`prometheus.yml` 작성

```shell
$ vi prometheus.yml
```

```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: spring
    metrics_path: '/actuator/prometheus'
    static_configs:
      - targets: ['app-blue:8080', 'app-green:8080']
  - job_name: node
    static_configs:
      - targets: ['node-exporter:9100']
```

`docker-compose-monitoring.yml` 작성


```shell
$ vi docker-compose-monitoring.yml
```

```yaml
services:
  prometheus:
    image: prom/prometheus:latest
    container_name: prometheus
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
    ports:
      - "9090:9090"
    networks:
      - monitoring-network

  node_exporter:
    image: prom/node-exporter:latest
    networks:
      - monitoring-network

  grafana:
    image: grafana/grafana:latest
    container_name: grafana
    ports:
      - "3000:3000"
    networks:
      - monitoring-network

networks:
  monitoring-network:
    driver: bridge
```

```shell
$ docker compose -f docker-compose-monitoring.yml up -d
```

그 이후 `docker-compose.yml`에 아래 내용을 추가해주었다.

```yaml

services:
  app-blue:
    ...
    networks:
      - monitoring-network
  
  app-green:
    ...
    networks:
      - monitoring-network

  redis:
    ...
    networks:
      - monitoring-network

networks:
  monitoring-network:
    driver: bridge
```

`http://<ec2-public-ip>:9090/targets`에 접속하면 아래와 같은 화면이 떠야한다.

![](/assets/img/posts-image/2025-08-09-07.png)

나는 블루 그린 배포를 해놓았기 때문에 블루 컨테이너는 닫혀있는 상태다.

<br>

## Grafana 구성

### Data Source 설정

`http://<ec2-public-ip>:3000`으로 접속한다. Grafana의 기본 아이디와 비밀번호는 admin/admin이다. 로그인을 성공하면 홈 화면이 뜨는데,

![datasource 화면](/assets/img/posts-image/2025-08-09-01.png)

DATA SOURCES로 들어가주면 된다. Prometheus를 선택한 뒤

![](/assets/img/posts-image/2025-08-09-02.png)

Connection에 `http://<container-name>:9090`을 입력한다. `http://<ec2-public-ip>:9090`도 괜찮다.

아래에서 save & test 버튼을 눌러서 

![](/assets/img/posts-image/2025-08-09-03.png)

해당 박스가 나오면 성공한 것이다.


### Dashboard 설정

![](/assets/img/posts-image/2025-08-09-04.png)

이제 대시보드 화면으로 들어가 원하는 대시보드를 만들거나 Import를 해주면 된다. 나는 [JVM (Micrometer)](https://grafana.com/grafana/dashboards/4701-jvm-micrometer/)와 [Node Exporter Full](https://grafana.com/grafana/dashboards/1860-node-exporter-full/)을 사용하기로 하였다.

![](/assets/img/posts-image/2025-08-09-05.png)

id인 4701과 1860을 입력하고 Load 후 Import를 누르면 된다.

#### 적용 화면

![](/assets/img/posts-image/2025-08-09-06.png)

