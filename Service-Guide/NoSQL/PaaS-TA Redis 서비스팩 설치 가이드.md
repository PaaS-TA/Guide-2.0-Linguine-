## Table of Contents
1. [문서 개요](#1)
     * [1.1. 목적](#2)
     * [1.2. 범위](#3)
     * [1.3. 시스템 구성도](#4)
     * [1.4. 참고자료](#5)
2. [Redis 서비스팩 설치](#6)
     * [2.1. 설치전 준비사항](#7)
     * [2.2. Redis 서비스 릴리즈 업로드](#8)
     * [2.3. Redis 서비스 Deployment 파일 수정 및 배포](#9)
     * [2.4. Redis 서비스 브로커 등록](#10)
3. [Redis 연동 Sample App 설명](#11)
     * [3.1. Sample App 구조](#12)
     * [3.2. PaaS-TA에서 서비스 신청](#13)
     * [3.3. Sample App에 서비스 바인드 신청 및 App 확인](#14)
4. [Redis Client 툴 접속](#15)
     * [4.1. Redis Desktop Manager 설치 및 연결](#16)


<div id='1'></div>
# 1. 문서 개요

<div id='2'></div>
### 1.1. 목적
      
본 문서(Redis 서비스팩 설치 가이드)는 전자정부표준프레임워크 기반의 PaaS-TA에서 제공되는 서비스팩인 Redis 서비스팩을 Bosh를 이용하여 설치하는 방법과 PaaS-TA의 SaaS 형태로 제공하는 Application에서 Redis 서비스를 사용하는 방법을 기술하였다.

<div id='3'></div>
### 1.2. 범위 

설치 범위는 Redis서비스팩을 검증하기 위한 기본 설치를 기준으로 작성하였다. 

<div id='4'></div>
### 1.3. 시스템 구성도
본 문서의 설치된 시스템 구성도입니다. Redis dedicated-node(2대), Redis 서비스 브로커로 최소사항을 구성하였다.
![시스템 구성도][redis_01]

<table>
  <tr>
    <td>구분</td>
    <td>스펙</td>
  </tr>
  <tr>
    <td>paasta-redis-broker</td>
    <td>1vCPU / 1GB RAM / 8GB Disk+4GB(영구적 Disk)</td>
  </tr>
  <tr>
    <td>dedicated-node</td>
    <td>1vCPU / 1GB RAM / 8GB Disk+4GB(영구적 Disk)</td>
  </tr>
</table>

<div id='5'></div>
### 1.4. 참고자료
[**http://bosh.io/docs**](http://bosh.io/docs)
[**http://docs.cloudfoundry.org/**](http://docs.cloudfoundry.org/)


<div id='6'></div>
#   2. Redis 서비스팩 설치

<div id='7'></div>
### 2.1. 설치전 준비사항
본 설치 가이드는 Linux 환경에서 설치하는 것을 기준으로 하였다.
서비스팩 설치를 위해서는 먼저 BOSH CLI가 설치 되어 있어야 하고 BOSH 에 로그인 및 타켓 설정이 되어 있어야 한다.
BOSH CLI가 설치 되어 있지 않을 경우 먼저 BOSH 설치 가이드 문서를 참고 하여 BOSH CLI를 설치 해야 한다.

-    PaaS-TA에서 제공하는 압축된 릴리즈 파일들을 다운받는다. (PaaSTA-Deployment.zip, PaaSTA-Sample-Apps.zip, PaaSTA-Services.zip)


<div id='8'></div>
###   2.2. Redis 서비스 릴리즈 업로드

-    PaaSTA-Services.zip 파일 압축을 풀고 폴더안에 있는 Redis 서비스 릴리즈 paasta-redis-2.0.tgz 파일을 확인한다.
```
$ ls --all
```
```
.  cf236      paasta-cubrid-2.0.tgz    paasta-mysql-2.0.tgz    paasta-portal-object-storage-2.0.tgz paasta-redis-2.0.tgz
.. cf-release paasta-glusterfs-2.0.tgz paasta-pinpoint-2.0.tgz paasta-rabbitmq-2.0.tgz              paasta-web-ide-2.0.tgz
```

<br>
-    업로드 되어 있는 릴리즈 목록을 확인한다.
```
$ bosh releases
```
```
RSA 1024 bit CA certificates are loaded due to old openssl compatibility
Acting as user 'admin' on 'bosh'

+--------------------------+----------+-------------+
| Name                     | Versions | Commit Hash |
+--------------------------+----------+-------------+
| cflinuxfs2-rootfs        | 1.40.0*  | 19fe09f4+   |
| empty-release            | 1+dev.1* | 00000000    |
| etcd                     | 86*      | 2dfbef00+   |
| paasta-container         | 2.0*     | b857e171    |
| paasta-controller        | 2.0*     | 0f315314    |
| paasta-garden-runc       | 2.0*     | ea5f5d4d+   |
| paasta-influxdb-grafana  | 2.0*     | 00000000    |
| paasta-logsearch         | 2.0*     | 00000000    |
| paasta-metrics-collector | 2.0*     | 00000000    |
+--------------------------+----------+-------------+
(*) Currently deployed
(+) Uncommitted changes

Releases total: 9
```
Redis 서비스 릴리즈가 업로드 되어 있지 않은 것을 확인

<br>
-    Redis 서비스 릴리즈 파일을 업로드한다.

```
$ bosh upload release paasta-redis-2.0.tgz
```
```
RSA 1024 bit CA certificates are loaded due to old openssl compatibility
Acting as user 'admin' on 'bosh'

+--------------------------+----------+-------------+
| Name                     | Versions | Commit Hash |
+--------------------------+----------+-------------+
| cflinuxfs2-rootfs        | 1.40.0*  | 19fe09f4+   |
| empty-release            | 1+dev.1* | 00000000    |
| etcd                     | 86*      | 2dfbef00+   |
| paasta-container         | 2.0*     | b857e171    |
| paasta-controller        | 2.0*     | 0f315314    |
| paasta-garden-runc       | 2.0*     | ea5f5d4d+   |
| paasta-influxdb-grafana  | 2.0*     | 00000000    |
| paasta-logsearch         | 2.0*     | 00000000    |
| paasta-metrics-collector | 2.0*     | 00000000    |
+--------------------------+----------+-------------+
(*) Currently deployed
(+) Uncommitted changes

Releases total: 9
inception@inception-new:~/bosh-space/kimdojun/redis$ bosh upload release paasta-redis-2.0.tgz 
RSA 1024 bit CA certificates are loaded due to old openssl compatibility
Acting as user 'admin' on 'bosh'

Verifying manifest...
Extract manifest                                             OK
Manifest exists                                              OK
Release name/version                                         OK

File exists and readable                                     OK
Read package 'cf-cli' (1 of 8)                               OK
Package 'cf-cli' checksum                                    OK
Read package 'cf-redis-broker' (2 of 8)                      OK
Package 'cf-redis-broker' checksum                           OK
Read package 'cf-redis-nginx' (3 of 8)                       OK
Package 'cf-redis-nginx' checksum                            OK
Read package 'cf-redis-smoke-tests' (4 of 8)                 OK
Package 'cf-redis-smoke-tests' checksum                      OK
Read package 'go' (5 of 8)                                   OK
Package 'go' checksum                                        OK
Read package 'redis-common' (6 of 8)                         OK
Package 'redis-common' checksum                              OK
Read package 'redis' (7 of 8)                                OK
Package 'redis' checksum                                     OK
Read package 'ruby' (8 of 8)                                 OK
Package 'ruby' checksum                                      OK
Package dependencies                                         OK
Checking jobs format                                         OK
Read job 'broker-deregistrar' (1 of 6), version fd74f060a430d793ed734328638db0a5fee34395 OK
Job 'broker-deregistrar' checksum                            OK
Extract job 'broker-deregistrar'                             OK
Read job 'broker-deregistrar' manifest                       OK
Check template 'errand.sh.erb' for 'broker-deregistrar'      OK
Job 'broker-deregistrar' needs 'cf-cli' package              OK
Monit file for 'broker-deregistrar'                          OK
Read job 'broker-registrar' (2 of 6), version 77bba8ba0e8fa8b06cc8be6e8ef373d285fa4daf OK
Job 'broker-registrar' checksum                              OK
Extract job 'broker-registrar'                               OK
Read job 'broker-registrar' manifest                         OK
Check template 'errand.sh.erb' for 'broker-registrar'        OK
Job 'broker-registrar' needs 'cf-cli' package                OK
Monit file for 'broker-registrar'                            OK
Read job 'cf-redis-broker' (3 of 6), version a8cc45566d5ebc66dd90bce0737a9833e7f3fcb8 OK
Job 'cf-redis-broker' checksum                               OK
Extract job 'cf-redis-broker'                                OK
Read job 'cf-redis-broker' manifest                          OK
Check template 'pre-start.erb' for 'cf-redis-broker'         OK
Check template 'cf-redis-broker_ctl.erb' for 'cf-redis-broker' OK
Check template 'health_check.sh.erb' for 'cf-redis-broker'   OK
Check template 'process-watcher_ctl.erb' for 'cf-redis-broker' OK
Check template 'process-destroyer_ctl.erb' for 'cf-redis-broker' OK
Check template 'nginx_ctl.erb' for 'cf-redis-broker'         OK
Check template 'broker.yml.erb' for 'cf-redis-broker'        OK
Check template 'nginx.conf.erb' for 'cf-redis-broker'        OK
Check template 'redis.conf.erb' for 'cf-redis-broker'        OK
Check template 'drain.sh' for 'cf-redis-broker'              OK
Job 'cf-redis-broker' needs 'cf-redis-broker' package        OK
Job 'cf-redis-broker' needs 'redis-common' package           OK
Job 'cf-redis-broker' needs 'cf-redis-nginx' package         OK
Job 'cf-redis-broker' needs 'redis' package                  OK
Monit file for 'cf-redis-broker'                             OK
Read job 'dedicated-node' (4 of 6), version 660e3ceee60783b8dcbc01fef065371370fe2265 OK
Job 'dedicated-node' checksum                                OK
Extract job 'dedicated-node'                                 OK
Read job 'dedicated-node' manifest                           OK
Check template 'agent.yml.erb' for 'dedicated-node'          OK
Check template 'redis.conf.erb' for 'dedicated-node'         OK
Check template 'nginx.conf.erb' for 'dedicated-node'         OK
Check template 'redis_ctl.erb' for 'dedicated-node'          OK
Check template 'redis-agent_ctl.erb' for 'dedicated-node'    OK
Check template 'nginx_ctl.erb' for 'dedicated-node'          OK
Check template 'redis-agent.pem.erb' for 'dedicated-node'    OK
Check template 'redis-agent.key.erb' for 'dedicated-node'    OK
Check template 'drain-redis.sh' for 'dedicated-node'         OK
Job 'dedicated-node' needs 'redis' package                   OK
Job 'dedicated-node' needs 'redis-common' package            OK
Job 'dedicated-node' needs 'cf-redis-nginx' package          OK
Job 'dedicated-node' needs 'cf-redis-broker' package         OK
Monit file for 'dedicated-node'                              OK
Read job 'smoke-tests' (5 of 6), version e9a839a060cbac2985bc79cfbcabfc2c59ad0c8c OK
Job 'smoke-tests' checksum                                   OK
Extract job 'smoke-tests'                                    OK
Read job 'smoke-tests' manifest                              OK
Check template 'config.json.erb' for 'smoke-tests'           OK
Check template 'errand.sh.erb' for 'smoke-tests'             OK
Job 'smoke-tests' needs 'go' package                         OK
Job 'smoke-tests' needs 'cf-redis-smoke-tests' package       OK
Job 'smoke-tests' needs 'cf-cli' package                     OK
Monit file for 'smoke-tests'                                 OK
Read job 'syslog-configurator' (6 of 6), version e74e363f8b9ac570f30b32d024e41ef2c89db03f OK
Job 'syslog-configurator' checksum                           OK
Extract job 'syslog-configurator'                            OK
Read job 'syslog-configurator' manifest                      OK
Check template 'syslog-configurator_ctl.erb' for 'syslog-configurator' OK
Check template 'syslog_forwarder.conf.erb' for 'syslog-configurator' OK
Job 'syslog-configurator' needs 'redis-common' package       OK
Monit file for 'syslog-configurator'                         OK

Release info
------------
Name:    paasta-redis
Version: 2.0

Packages
 - cf-cli (33a64fb1b0ca68b3403fe5b0254e86ec7d672dba)
 - cf-redis-broker (f530f8b2135eac4a888c2da20177082eb081ee65)
 - cf-redis-nginx (dd15c82027671c74b108c52bcecb64fcaf9c0d38)
 - cf-redis-smoke-tests (b347f491c873fdd9e878c90defd276a82f980023)
 - go (32629593cd827ebaf88981b56d205bea6c8b7c18)
 - redis-common (3747d5011f5405b1f8033653dae31b28e6839451)
 - redis (c6226fd977b4bcb4693823d32ddeb4c9c2c0c76f)
 - ruby (9b59d2f2700da81a98c38c73cd27b6ccf26f188c)

Jobs
 - broker-deregistrar (fd74f060a430d793ed734328638db0a5fee34395)
 - broker-registrar (77bba8ba0e8fa8b06cc8be6e8ef373d285fa4daf)
 - cf-redis-broker (a8cc45566d5ebc66dd90bce0737a9833e7f3fcb8)
 - dedicated-node (660e3ceee60783b8dcbc01fef065371370fe2265)
 - smoke-tests (e9a839a060cbac2985bc79cfbcabfc2c59ad0c8c)
 - syslog-configurator (e74e363f8b9ac570f30b32d024e41ef2c89db03f)

License
 - license (443041add743bce9c52077d8f3d2e130c08340c5)

Checking if can repack release for faster upload...
cf-cli (33a64fb1b0ca68b3403fe5b0254e86ec7d672dba) UPLOAD
cf-redis-broker (f530f8b2135eac4a888c2da20177082eb081ee65) UPLOAD
cf-redis-nginx (dd15c82027671c74b108c52bcecb64fcaf9c0d38) UPLOAD
cf-redis-smoke-tests (b347f491c873fdd9e878c90defd276a82f980023) UPLOAD
go (32629593cd827ebaf88981b56d205bea6c8b7c18) UPLOAD
redis-common (3747d5011f5405b1f8033653dae31b28e6839451) UPLOAD
redis (c6226fd977b4bcb4693823d32ddeb4c9c2c0c76f) UPLOAD
ruby (9b59d2f2700da81a98c38c73cd27b6ccf26f188c) UPLOAD
broker-deregistrar (fd74f060a430d793ed734328638db0a5fee34395) UPLOAD
broker-registrar (77bba8ba0e8fa8b06cc8be6e8ef373d285fa4daf) UPLOAD
cf-redis-broker (a8cc45566d5ebc66dd90bce0737a9833e7f3fcb8) UPLOAD
dedicated-node (660e3ceee60783b8dcbc01fef065371370fe2265) UPLOAD
smoke-tests (e9a839a060cbac2985bc79cfbcabfc2c59ad0c8c) UPLOAD
syslog-configurator (e74e363f8b9ac570f30b32d024e41ef2c89db03f) UPLOAD
Uploading the whole release

Uploading release
paasta-redis-:  96% |oooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooo     | 104.3MB  22.8MB/s ETA:  00:00:00
Director task 
 Started extracting release > Extracting release. Done (00:00:01)

 Started verifying manifest > Verifying manifest. Done (00:00:00)

 Started resolving package dependencies > Resolving package dependencies. Done (00:00:00)

 Started creating new packages
 Started creating new packages > cf-cli/33a64fb1b0ca68b3403fe5b0254e86ec7d672dba. Done (00:00:00)
 Started creating new packages > cf-redis-broker/f530f8b2135eac4a888c2da20177082eb081ee65. Done (00:00:00)
 Started creating new packages > cf-redis-nginx/dd15c82027671c74b108c52bcecb64fcaf9c0d38. Done (00:00:00)
 Started creating new packages > cf-redis-smoke-tests/b347f491c873fdd9e878c90defd276a82f980023. Done (00:00:00)
 Started creating new packages > go/32629593cd827ebaf88981b56d205bea6c8b7c18. Done (00:00:02)
 Started creating new packages > redis-common/3747d5011f5405b1f8033653dae31b28e6839451. Done (00:00:00)
 Started creating new packages > redis/c6226fd977b4bcb4693823d32ddeb4c9c2c0c76f. Done (00:00:00)
 Started creating new packages > ruby/9b59d2f2700da81a98c38c73cd27b6ccf26f188c. Done (00:00:00)
    Done creating new packages (00:00:02)

 Started creating new jobs
 Started creating new jobs > broker-deregistrar/fd74f060a430d793ed734328638db0a5fee34395. Done (00:00:00)
 Started creating new jobs > broker-registrar/77bba8ba0e8fa8b06cc8be6e8ef373d285fa4daf. Done (00:00:00)
 Started creating new jobs > cf-redis-broker/a8cc45566d5ebc66dd90bce0737a9833e7f3fcb8. Done (00:00:00)
 Started creating new jobs > dedicated-node/660e3ceee60783b8dcbc01fef065371370fe2265. Done (00:00:00)
 Started creating new jobs > smoke-tests/e9a839a060cbac2985bc79cfbcabfc2c59ad0c8c. Done (00:00:00)
 Started creating new jobs > syslog-configurator/e74e363f8b9ac570f30b32d024e41ef2c89db03f. Done (00:00:00)
    Done creating new jobs (00:00:00)

 Started release has been created > paasta-redis/2.0. Done (00:00:00)

Task 2337 done

Started        2017-01-13 06:03:46 UTC
Finished  2017-01-13 06:03:49 UTC
Duration  :00:03
paasta-redis-:  96% |oooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooo     | 104.6MB  10.7MB/s Time: 00:00:09

Release uploaded
```

<br>
-    업로드 된 Redis 릴리즈를 확인한다.
```
$ bosh releases
```
```
RSA 1024 bit CA certificates are loaded due to old openssl compatibility
Acting as user 'admin' on 'bosh'

+--------------------------+----------+-------------+
| Name                     | Versions | Commit Hash |
+--------------------------+----------+-------------+
| cflinuxfs2-rootfs        | 1.40.0*  | 19fe09f4+   |
| empty-release            | 1+dev.1* | 00000000    |
| etcd                     | 86*      | 2dfbef00+   |
| paasta-container         | 2.0*     | b857e171    |
| paasta-controller        | 2.0*     | 0f315314    |
| paasta-garden-runc       | 2.0*     | ea5f5d4d+   |
| paasta-influxdb-grafana  | 2.0*     | 00000000    |
| paasta-logsearch         | 2.0*     | 00000000    |
| paasta-metrics-collector | 2.0*     | 00000000    |
| paasta-redis             | 2.0      | 2d766084+   |
+--------------------------+----------+-------------+
(*) Currently deployed
(+) Uncommitted changes

Releases total: 10
```

<br>
<div id='9'></div>
###   2.3. Redis 서비스 Deployment 파일 수정 및 배포
BOSH Deployment manifest 는 components 요소 및 배포의 속성을 정의한 YAML  파일이다.
Deployment manifest 에는 sotfware를 설치 하기 위해서 어떤 Stemcell (OS, BOSH agent) 을 사용할것인지와 Release (Software packages, Config templates, Scripts)의 이름과 버전, VMs 용량, Jobs params 등이 정의 되어 있다.

-    PaaSTA-Deployment.zip 파일 압축을 풀고 폴더안에 있는 IaaS별 Redis Deployment 파일을 복사한다. 
예) vsphere 일 경우 paasta_redis_vsphere_2.0.yml를 복사

-    Director UUID를 확인한다.
BOSH CLI가 배포에 대한 모든 작업을 허용하기위한 현재 대상 BOSH Director의 UUID와 일치해야한다. ‘bosh status’ CLI 을 통해서 현재 BOSH Director 에 target 되어 있는 UUID를 확인할수 있다.


```
$ bosh status
```
```
Config
            /home/inception/.bosh_config

Director
RSA 1024 bit CA certificates are loaded due to old openssl compatibility
 Name       bosh
 URL        https://10.30.40.105:25555
 Version    .1.0 (00000000)
 User       admin
 UUID       d363905f-eaa0-4539-a461-8c1318498a32
 CPI        vsphere_cpi
 dns        disabled
 compiled_package_cache disabled
 snapshots  disabled

Deployment
 Manifest   /home/inception/crossent-deploy/paasta-logsearch.yml
```

<br>
-    Deploy시 사용할 Stemcell을 확인한다.
```
$ bosh stemcells
```
```
RSA 1024 bit CA certificates are loaded due to old openssl compatibility
Acting as user 'admin' on 'bosh'

+------------------------------------------+---------------+---------+-----------------------------------------+
| Name                                     | OS            | Version | CID                                     |
+------------------------------------------+---------------+---------+-----------------------------------------+
| bosh-vsphere-esxi-ubuntu-trusty-go_agent | ubuntu-trusty | 3263.8* | sc-af443b65-9335-43b1-9b64-6d1791a10428 |
| bosh-vsphere-esxi-ubuntu-trusty-go_agent | ubuntu-trusty | 3309*   | sc-e00c788b-ac6b-4089-bc43-f56a3ffdb55a |
+------------------------------------------+---------------+---------+-----------------------------------------+

(*) Currently in-use

Stemcells total: 2
```
Stemcell 목록이 존재 하지 않을 경우 BOSH 설치 가이드 문서를 참고 하여 Stemcell을 업로드 해야 한다.

<br>
-    Deployment 파일을 서버 환경에 맞게 수정한다. (vsphere 용으로 설명, 다른 IaaS는 해당 Deployment 파일의 주석내용을 참고)
```yaml
# paasta-redis-vsphere 설정 파일 내용

name: paasta-redis-service                             # 서비스 배포이름(필수)
director_uuid: d363905f-eaa0-4539-a461-8c1318498a32    #bosh status 에서 확인한 Director UUID을 입력(필수)

releases:
- name: paasta-redis                                   #서비스 릴리즈 이름(필수)
  version: 2.0                                         #서비스 릴리즈 버전(필수): latest 시 업로드된 서비스 릴리즈 최신버전

update:
  canaries: 1                                          # canary 인스턴스 수(필수)
  canary_watch_time: 30000-180000                      # canary 인스턴스가 수행하기 위한 대기 시간(필수)
  max_in_flight: 6                                     # non-canary 인스턴스가 병렬로 update 하는 최대 개수(필수)
  update_watch_time: 30000-180000                      # non-canary 인스턴스가 수행하기 위한 대기 시간(필수)

compilation:                                           #컴파일시 필요한 가상머신의 속성(필수)
  cloud_properties:            # 컴파일 VM을 만드는 데 필요한 IaaS의 특정 속성 (instance_type, availability_zone), 직접 cpu,disk,ram 사이즈를 넣어도 됨
    cpu: 2
    disk: 4096
    ram: 4096
  network: default                       # Networks block에서 선언한 network 이름(필수)
  reuse_compilation_vms: true            # 컴파일지 VM 재사용 여부(옵션)
  workers: 6                             # 컴파일 하는 가상머신의 최대수(필수)

jobs:
- instances: 1
  name: paasta-redis-broker                # 작업 이름(필수)
  networks:                                # 네트워크 구성정보
  - name: default                          # Networks block에서 선언한 network 이름(필수)
    static_ips:
    - 10.30.60.71                          # 사용할 IP addresses 정의(필수)
  persistent_disk: 4096                    # 영구적 디스크 사이즈 정의(옵션)
  properties:
    nats:                                  # paas-ta nats 정보
      machines:
      - 10.30.110.31
      password: nats
      port: 4222
      user: nats
  resource_pool: services-small
  templates:
  - name: cf-redis-broker
    release: paasta-redis
  - name: syslog-configurator
    release: paasta-redis

- instances: 3
  name: dedicated-node                        # 전용 노드
  networks:                               
  - name: default
    static_ips:                              # 전용 노드 IP 목록(필수)
    - 10.30.60.72
    - 10.30.60.73
    - 10.30.60.74
  persistent_disk: 4096
  resource_pool: services-small
  templates:
  - name: dedicated-node
    release: paasta-redis
  - name: syslog-configurator
    release: paasta-redis

- instances: 1
  lifecycle: errand                                  # bosh deploy시 vm에 생성되어 설치 되지 않고 bosh errand 로실행할때 설정, 주로 테스트 용도에 쓰임
  name: broker-registrar
  networks:
  - name: default
  properties:
    broker:
      host: 10.30.60.71
      name: paasta-redis-service
      password: admin
      username: admin
  resource_pool: services-small
  templates:
  - name: broker-registrar
    release: paasta-redis

- instances: 1
  lifecycle: errand
  name: broker-deregistrar
  networks:
  - name: default
  properties:
    broker:
      host: 10.30.60.71
      name: paasta-redis-service
      password: admin
      username: admin
  resource_pool: services-small
  templates:
  - name: broker-deregistrar
    release: paasta-redis
- instances: 1
  lifecycle: errand
  name: smoke-tests
  networks:
  - name: default
  resource_pool: services-small
  templates:
  - name: smoke-tests
    release: paasta-redis

meta:
  apps_domain: 115.68.46.30.xip.io
  broker:                                                      # broker 정보 : 디폴트 포트 12350
    host: paasta-redis-broker.115.68.46.30.xip.io
    name: redis
    password: admin
    username: admin
  cf:                                                          # paas-ta 정보
    admin_password: admin
    admin_username: admin
    api_url: https://api.115.68.46.30.xip.io
    apps_domain: 115.68.46.30.xip.io
    skip_ssl_validation: false
    system_domain: 115.68.46.30.xip.io
  deployment_name: paasta-redis-service
  director_uuid: d363905f-eaa0-4539-a461-8c1318498a32          # uuid 정보 bosh status
  external_domain: 115.68.46.30.xip.io
  nats:                                                        # nats 정보
    machines:
    - 10.30.110.31
    password: nats
    port: 4222
    user: nats
  redis:
    bg_save_command: yetanotherrandomstring
    broker:                                                       # broker 정보
      dedicated_vm_plan_id: 48b35349-d3de-4e19-bc4a-66996ae07766
      name: redis
      service_id: 7aba7e52-f61b-4263-9de1-14e9d11fb67d
      shared_vm_plan_id: 78bf886c-bc50-4f31-a03c-cb786a158286
      subdomain: paasta-redis-broker
    config_command: configalias
    dedicated_plan:
      instance_count: 3                                        # 전용노드 수
    save_command: anotherrandomstring
    shared_plan:
      instance_limit: 10                                       # 공유 노드 수
      max_memory: 262144000
  release_name: paasta-redis
  route_name: paasta-redis-broker
  service_name: redis
  syslog_aggregator: null

networks:                             # 네트워크 블록에 나열된 각 서브 블록이 참조 할 수있는 작업이 네트워크 구성을 지정, 네트워크 구성은 네트워크 담당자에게 문의 하여 작성 요망
  subnets:
  - cloud_properties:
      name: Internal                                            # vsphere 에서 사용하는 network 이름(필수)
    dns:                                                        # dns 정보 
    - 8.8.8.8
    gateway: 10.30.20.23
    name: default_unused
    range: 10.30.0.0/16
    reserved:                                                   # 설치시 제외할 IP 설정
    - 10.30.0.1 - 10.30.10.254
    - 10.30.40.1 - 10.30.60.70
    - 10.30.60.81 - 10.30.254.254
    static:                                                     #사용 가능한 IP 설정
    - 10.30.60.71 - 10.30.60.80

properties:
  broker:                                                       # broker 정보
    host: paasta-redis-broker.115.68.46.30.xip.io
    name: redis
    password: admin
    username: admin
  cf:                                                           # paas-ta 정보 
    admin_password: admin
    admin_username: admin
    api_url: https://api.115.68.46.30.xip.io
    apps_domain: 115.68.46.30.xip.io
    skip_ssl_validation: false
    system_domain: 115.68.46.30.xip.io
  redis:
    agent:
      backend_port: 54321
    bg_save_command: yetanotherrandomstring
    broker:
      auth:
        password: admin
        username: admin
      backend_host: 10.30.60.71
      backend_port: 12345
      dedicated_nodes:
      - 10.30.60.72
      - 10.30.60.73
      - 10.30.60.74
      dedicated_vm_plan_id: 48b35349-d3de-4e19-bc4a-66996ae07766
      enable_service_access: true
      host: 10.30.60.71
      name: redis
      network: default
      route_name: paasta-redis-broker
      service_id: 7aba7e52-f61b-4263-9de1-14e9d11fb67d
      service_instance_limit: 20                                    # 최대 서비스 인스턴스 개수
      service_name: redis
      shared_vm_plan_id: 78bf886c-bc50-4f31-a03c-cb786a158286
      subdomain: redis-broker
    config_command: configalias
    host: 10.30.60.71
    maxmemory: 262144000
    save_command: anotherrandomstring
  syslog_aggregator: null

resource_pools:                                      # 배포시 사용하는 resource pools를 명시하며 여러 개의 resource pools 을 사용할 경우 name은 고유해야함(필수)
- cloud_properties:
    cpu: 1
    disk: 8192
    ram: 1024
  name: services-small                               # 고유한 resource pool 이름
  network: default 
  stemcell:
    name: bosh-vsphere-esxi-ubuntu-trusty-go_agent   # stemcell 이름(필수)
    version: "3263.8"                                 # stemcell 버전(필수)
```

<br>
-    Deploy 할 deployment manifest 파일을 BOSH 에 지정한다.
```
$ bosh deployment paasta_redis_vsphere_2.0.yml
```
```
RSA 1024 bit CA certificates are loaded due to old openssl compatibility
Acting as user 'admin' on deployment 'paasta-redis-service' on 'bosh'
Getting deployment properties from director...

Detecting deployment changes
----------------------------

...<manifest 파일 내용 출력// 생략 >...

Please review all changes carefully 

Deploying
---------
Are you sure you want to deploy? (type 'yes' to continue): yes

Director task 
Deprecation: Ignoring cloud config. Manifest contains 'networks' section.

 Started preparing deployment > Preparing deployment. Done (00:00:02)

 Started preparing package compilation > Finding packages to compile. Done (00:00:00)

 Started compiling packages
 Started compiling packages > cf-cli/33a64fb1b0ca68b3403fe5b0254e86ec7d672dba
 Started compiling packages > cf-redis-smoke-tests/b347f491c873fdd9e878c90defd276a82f980023
 Started compiling packages > redis/c6226fd977b4bcb4693823d32ddeb4c9c2c0c76f
 Started compiling packages > cf-redis-nginx/dd15c82027671c74b108c52bcecb64fcaf9c0d38
 Started compiling packages > redis-common/3747d5011f5405b1f8033653dae31b28e6839451
 Started compiling packages > go/32629593cd827ebaf88981b56d205bea6c8b7c18
    Done compiling packages > redis-common/3747d5011f5405b1f8033653dae31b28e6839451(00:01:17)
    Done compiling packages > cf-redis-smoke-tests/b347f491c873fdd9e878c90defd276a82f980023(00:01:20)
    Done compiling packages > cf-cli/33a64fb1b0ca68b3403fe5b0254e86ec7d672dba(00:01:22)
    Done compiling packages > go/32629593cd827ebaf88981b56d205bea6c8b7c18(00:01:57)
 Started compiling packages > cf-redis-broker/f530f8b2135eac4a888c2da20177082eb081ee65
    Done compiling packages > redis/c6226fd977b4bcb4693823d32ddeb4c9c2c0c76f(00:02:20)
    Done compiling packages > cf-redis-nginx/dd15c82027671c74b108c52bcecb64fcaf9c0d38(00:02:21)
    Done compiling packages > cf-redis-broker/f530f8b2135eac4a888c2da20177082eb081ee65(00:00:26)
    Done compiling packages (00:02:23)

 Started creating missing vms
 Started creating missing vms > paasta-redis-broker/113f1267-9b53-4e1c-94db-2e29abfbd687 (0)
 Started creating missing vms > dedicated-node/a90f1ed2-d7cd-437d-9671-b49901b1d7b3 (0)
 Started creating missing vms > dedicated-node/55611f55-db59-4c58-b74e-bf14deaea025 (1)
 Started creating missing vms > dedicated-node/c74d92c7-2d0f-4d51-9868-d7aad55167ee (2)
    Done creating missing vms > dedicated-node/a90f1ed2-d7cd-437d-9671-b49901b1d7b3 (0)(00:01:05)
    Done creating missing vms > dedicated-node/c74d92c7-2d0f-4d51-9868-d7aad55167ee (2)(00:01:06)
    Done creating missing vms > dedicated-node/55611f55-db59-4c58-b74e-bf14deaea025 (1)(00:01:07)
    Done creating missing vms > paasta-redis-broker/113f1267-9b53-4e1c-94db-2e29abfbd687 (0)(00:01:09)
    Done creating missing vms (00:01:09)

 Started updating instance paasta-redis-broker> paasta-redis-broker/113f1267-9b53-4e1c-94db-2e29abfbd687 (0) (canary). Done (00:01:3)
 Started updating instance dedicated-node
 Started updating instance dedicated-node> dedicated-node/a90f1ed2-d7cd-437d-9671-b49901b1d7b3 (0) (canary). Done (00:01:36)
 Started updating instance dedicated-node> dedicated-node/55611f55-db59-4c58-b74e-bf14deaea025 (1)
 Started updating instance dedicated-node> dedicated-node/c74d92c7-2d0f-4d51-9868-d7aad55167ee (2). Done (00:01:31)
    Done updating instance dedicated-node> dedicated-node/55611f55-db59-4c58-b74e-bf14deaea025 (1)(00:01:32)
    Done updating instance dedicated-node(00:03:08)

Task 2551 done

Started        2017-01-13 09:15:17 UTC
Finished  2017-01-13 09:24:08 UTC
Duration  :08:51

Deployed 'paasta-redis-service' to 'bosh'
```

<br>
-    배포된 Redis 서비스팩을 확인한다.
```
$ bosh vms
```
```
RSA 1024 bit CA certificates are loaded due to old openssl compatibility
Acting as user 'admin' on deployment 'paasta-redis-service' on 'bosh'

Director task 

Task 2415 done

+--------------------------------------------------------------+---------+-----+----------------+-------------+
| VM                                                           | State   | AZ  | VM Type        | IPs         |
+--------------------------------------------------------------+---------+-----+----------------+-------------+
| dedicated-node/0 (a1017de7-dbd9-4eeb-9790-996b69a9f06c)      | running | n/a | services-small | 10.30.60.72 |
| dedicated-node/1 (4020a083-6bfa-431e-a047-2f567775cfbb)      | running | n/a | services-small | 10.30.60.73 |
| dedicated-node/2 (904bd212-43dc-45ef-876e-37a9cad54d36)      | running | n/a | services-small | 10.30.60.74 |
| paasta-redis-broker/0 (b1ed5994-741d-4e7c-9bf9-2406621b10ec) | running | n/a | services-small | 10.30.60.71 |
+--------------------------------------------------------------+---------+-----+----------------+-------------+

VMs total: 4
```
