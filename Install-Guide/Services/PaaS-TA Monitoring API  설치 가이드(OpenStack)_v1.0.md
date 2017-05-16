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
      
본 문서는 Openstack 기반에 설치하기 위한 내용으로 한정되어 있다.

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
compilation:
  cloud_properties:
    name: random
    instance_type: monitoring								#openstack flavor
    availability_zone: nova									#available zone
  network: cf1
  reuse_compilation_vms: true
  workers: 1
  
director_uuid: <%= `bosh status --uuid` %>					#bosh status --uuid 
jobs:
- cloud_properties:
    name: default
  instances: 1												#instance count
  memory_mb: 2048
  name: api_server_z1										#bosh vm name
  networks:
  - name: monitoring-api-server-net							#network name
    static_ips: 10.244.30.20								#static IP
  properties: {}
  resource_pool: monitoring-api-server						#resource name (resource_pools)
  templates:
  - name: api_server
    release: paasta-monitoring-api-server
  update:
    max_in_flight: 1
    serial: false

name: paasta-monitoring-api-server							#deployment name

networks:
- name: monitoring-api-server-net							#network name
  subnets:
  - cloud_properties:
      name: random
      net_id: b7c8c08f-2d3b-4a86-bd10-641cb6faa317			#network id
      security_groups: [bosh]								#security group
    dns:
    - 10.10.0.2											#dns ip
    gateway: 10.10.5.1										#gateway 
    range: 10.10.5.0/24										#static ip range
    reserved:		
    - 10.10.5.2 - 10.10.5.110								#reserved ip range
    static:
    - 10.10.5.111 - 10.10.5.120								#available ip range
  type: manual

properties:
  openstack: &openstack																							
      auth_url: http://"xxx.xxx.xxx.xxx":5000/v3			#openstack auth url
      username: admin										#openstack admin id
      api_key: admin										#openstack admin password
      project: admin										#related project name
      domain: default										#default domain
      region: RegionOne																										
      default_key_name: monitoring							#keypair name
      default_security_groups: [bosh]						#security group
  syslog_daemon_config:
    address: null
    port: null
  api_server:
    listen_addr: 0.0.0.0:9999								#api server listen address
    log_level: debug										#log level
    dopplerUrl: wss://doppler.bosh-lite.com:4443			#doppler websocket url
    uaa:
      url: http://uaa.bosh-lite.com							#uaa url
      adminId: admin										#admin account id
      adminPass: admin										#admin account password

releases:
- name: paasta-monitoring-api-server						#release name
  version: latest											#release version

resource_pools:
- cloud_properties:
    name: random
    instance_type: monitoring								#openstack flavor
    availability_zone: nova									#available zone
  name: monitoring-api-server								#resource name
  network: monitoring-api-server-net						#network name
  stemcell:
    name: bosh-openstack-kvm-ubuntu-trusty-go_agent			#stemcell name
    version: latest											#stemcell version
    
update:
  canaries: 1
  canary_watch_time: 30000-100000
  max_errors: 1
  max_in_flight: 1
  update_watch_time: 30000-100000
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
