## Table of Contents
1. [문서 개요](#1)
     * [1.1. 목적](#2)
     * [1.2. 범위](#3)
     * [1.3. 전제조건](#4)
2. [Bosh Monitoring Agent 설치](#5)
     * [2.1.  Bosh Monitoring Agent 파일 업로드](#6)
     * [2.2.  Bosh VM 로그인](#7)
     * [2.3.  Bosh Monitoring Agent 실행](#8)
     * [2.4.  확인](#9)

<div id='1'></div>
# 1. 문서 개요

<div id='2'></div>
### 1.1. 목적
      
본 문서는 IaaS(Infrastructure as a Service) 중 하나인 AWS 환경에서 Bosh VM 환경의 시스템 Metrics(CPU, Memory, Disk) 정보를 수집하기 위한 agent를  설치하는 방법을 제공하는데 그 목적이 있다.

<div id='3'></div>
### 1.2. 범위
      
본 문서는 AWS 기반에 설치하기 위한 내용으로 한정되어 있다.

<div id='4'></div>
### 1.3. 전제조건
      
Bosh Monitoring Agent를 설치하기 위해서는 사전에 Bosh 서비스가 배포되어 서비스되고 있어야 한다.

<div id='5'></div>
# 2.  Bosh Monitoring Agent 설치

본 장에서는 Monitoring API Release를 배포하는 방법에 대해서 기술하였다.

<div id='6'></div>
### 2.1.  Bosh Monitoring Agent 파일 업로드 

하단 링크로 접속하여 bosh monitoring agent 파일을 다운로드 한다. 

>Bosh Monitoring Agent : **<http://extdisk.hancom.com:8080/share.cgi?ssid=01wAQNx>**

다음의 명령어를 이용하여 압축 파일을 해제한다.

$ tar xvf paasta-monitoring-agent-2.0.tar

다음의 명령어를 이용하여, Monitoring Agent 파일을 업로드한다.

$ scp -i "keypair file" paasta-monitoring-agent-2.0/monitoring-agent vcap@"Bosh VM IP":"저장 위치"

```
[example]
$ scp -i .ssh/monitoring.pem paasta-monitoring-agent-2.0/monitoring-agent vcap@10.10.0.6:~/home/vcap/
```

<kbd>![2-1-1]</kbd>

<div id='7'></div>
### 2.2.  Bosh VM 로그인

다음의 명령어를 이용하여 Bosh VM 환경에 로그인한다.

$ ssh -i "keypair file" vcap@"Bosh VM IP"

```
[example]
$ ssh -i id_rsa_bosh vcap@10.10.0.6
```

<kbd>![2-2-1]</kbd>

<div id='8'></div>
### 2.3.  Bosh Monitoring Agent 실행

[2-1] 단계에서 업로드한 위치로 이동한다.
$ cd ~/home/vcap

Monitoring Agent 파일을 실행한다.
$./monitoring_agent -origin=”Bosh VM 명칭” -influxUrl=”influxDB_IP:Port” -influxDatabase=”Target Database Name” -measurement=”Target Measurement Name”

```
[example]
$./monitoring_agent -origin="Bosh Director" -influxUrl="10.244.3.151:8059" -influxDatabase="bosh_metric_db" -measurement="bosh_metrics"
```

<kbd>![2-3-1]</kbd>


<div id='9'></div>
### 2.4.  확인

다음의 명령어를 실행하여 Monitoring Agent가 실행되었는지 확인한다.
$ ps -ef |grep monitoring_agent

<kbd>![2-4-1]</kbd>

[2-1-1]:images/monitoring-agent/2-1-1.png
[2-2-1]:images/monitoring-agent/2-2-1.png
[2-3-1]:images/monitoring-agent/2-3-1.png
[2-4-1]:images/monitoring-agent/2-4-1.png
