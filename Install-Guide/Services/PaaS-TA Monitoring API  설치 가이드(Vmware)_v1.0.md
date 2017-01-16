## Table of Contents
1. [문서 개요](#1)
     * [1.1. 목적](#2)
     * [1.2. 범위](#3)
     * [1.3. 전제조건](#4)
2. [Monitoring API Release 배포](#5)
     * [2.1.  upload release](#6)
     * [2.2.  manifest 파일 설정](#7)
     * [2.3.  deploy](#8)
     * [2.4.  확인](#9)

<div id='1'></div>
# 1. 문서 개요

<div id='2'></div>
### 1.1. 목적
      
본 문서는 PaaS-TA 사용자 포탈 서비스와 연동하여, Application이 배포되어 실행되는 Container 환경의 상태정보(cpu, memory, disk)를 제공함으로써, auto-scaling의 지표로 활용될 수 있도록 하는데 그 목적이 있다.

<div id='3'></div>
### 1.2. 범위
      
본 문서는 VMware 기반에 설치하기 위한 내용으로 한정되어 있다.

<div id='4'></div>
### 1.3. 전제조건
      
Monitoring API 서버를 설치하기 위해서는 사전에 PaaS-TA 서비스가 배포되어 서비스되고 있어야 한다.

<div id='5'></div>
# 2.  Monitoring API Release 배포

본 장에서는 Monitoring API Release를 배포하는 방법에 대해서 기술하였다.

<div id='6'></div>
### 2.1.  upload "Monitoring API" release

하단 링크로 접속하여 Monitoring API 릴리즈 파일인 paasta-monitoring-api-server-2.0.tgz를 다운로드 한다. 

>PaaS-TA Monitoring API : **<http://extdisk.hancom.com:8080/>**

다음의 명령어를 이용하여 릴리즈 파일을 bosh에 업로드한다.

$ bosh upload release paasta-monitoring-api-server-2.0.tgz

<kbd>![2-1-1]</kbd>
<kbd>![2-1-2]</kbd>

<div id='7'></div>
### 2.2.  manifest 파일 설정

1. "Monitoring API" 배포되는 환경에 맞게 manifest 파일의 설정 정보를 수정한다.

$ vi monitoring-api-release.yml

```

```

2. manifest 파일 지정

$ bosh deployment monitoring-api-release.yml

<kbd>![2-2-1]</kbd>

<div id='8'></div>
### 2.3.  배포

$ bosh -n deploy 

<kbd>![2-3-1]</kbd>
<kbd>![2-3-2]</kbd>

<div id='9'></div>
### 2.4.  확인

$ bosh deployments 

<kbd>![2-4-1]</kbd>


[2-1-1]:images/monitoring-api/2-1-1.png
[2-1-2]:images/monitoring-api/2-1-2.png
[2-2-1]:images/monitoring-api/2-2-1.png
[2-3-1]:images/monitoring-api/2-3-1.png
[2-3-2]:images/monitoring-api/2-3-2.png
[2-4-1]:images/monitoring-api/2-4-1.png
