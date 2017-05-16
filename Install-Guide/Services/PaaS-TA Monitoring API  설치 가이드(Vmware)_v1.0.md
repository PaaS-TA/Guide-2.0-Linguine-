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

하단 링크로 접속하여 PaaS-TA 모니터링 패키지 파일을 다운로드하여, 압축해제한다.

>PaaS-TA 모니터링 : **<https://paas-ta.kr/data/packages/2.0/PaaSTA-Monitoring.zip>** <br>
>다운로드 받은 PaaSTA-Monitoring.zip 파일을 압축해제한다. <br>
>paasta-monitoring-api-server-2.0.tgz 파일을 확인한다. <br>

다음의 명령어를 이용하여 릴리즈 파일을 bosh에 업로드한다.

$ bosh upload release paasta-monitoring-api-server-2.0.tgz

<kbd>![2-1-1]</kbd>
<kbd>![2-1-2]</kbd>

<div id='7'></div>

### 2.2.  manifest 파일 설정

1. "Monitoring API" 배포되는 환경에 맞게 manifest 파일의 설정 정보를 수정한다.

$ vi monitoring-api-release.yml

```
---
name: paasta-monitoring-api-server								#deployment name

compilation:
  cloud_properties:
    cpu: 2
    disk: 8192
    ram: 2048
  network: paasta-monitoring-api-server-network					#network name
  reuse_compilation_vms: true
  workers: 6
  
director_uuid: <%= `bosh status --uuid` %>						#bosh uuid

releases:
- {name: paasta-monitoring-api-server, version: 2.0}

jobs:
- name: api_server_z1													#service name
  templates:
  - {name: api_server, release: paasta-monitoring-api-server}			#release info
  instances: 1
  resource_pool: api_server												#resource name
  networks:
  - name: paasta-monitoring-api-server-network							#network name
    static_ips:
    - 10.30.152.181														#local static ip
  - default:
    - dns
    - gateway
    name: external
    static_ips:
    - "xxx.xxx.xxx.xxx"													#public ip
  properties: {}
  update:
    max_in_flight: 1
    serial: false
        
networks:
- name: paasta-monitoring-api-server-network							#internal network name
  subnets:
  - cloud_properties:
      name: Internal
    dns:
    - 10.30.20.27														#dns
    - 8.8.8.8
    gateway: 10.30.20.23												#gateway
    range: 10.30.0.0/16													#static ip range
    reserved:
    - 10.30.0.1 - 10.30.152.179											#reserved ip range
    static:
    - 10.30.152.180 - 10.30.152.185										#available ip range
  type: manual

- name: external														#external network name
  subnets:
  - cloud_properties:
      name: External
    dns:
    - 10.30.20.27														#dns
    - 8.8.8.8
    gateway: "xxx.xxx.xxx.xxx"											#gateway
    range: "xxx.xxx.xxx.xxx"/24											#static ip range
    static:
    - "xxx.xxx.xxx.xxx"													#available ip range
  type: manual

resource_pools:
- cloud_properties:
    cpu: 1
    disk: 1024
    ram: 1024
  env:
    bosh:
      password: $6$4gDD3aV0rdqlrKC$2axHCxGKIObs6tAmMTqYCspcdvQXh3JJcvWOY2WGb4SrdXtnCyNaWlrf3WEqvYR2MYizEGp3kMmbpwBC6jsHt0
  name: api_server														#resource name
  network: paasta-monitoring-api-server-network							#network name
  stemcell:
    name: bosh-vsphere-esxi-ubuntu-trusty-go_agent						#stemcell name
    version: latest														#stemcell version

update:
  canaries: 1
  canary_watch_time: 1000-180000
  max_in_flight: 1
  serial: true
  update_watch_time: 1000-180000
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
