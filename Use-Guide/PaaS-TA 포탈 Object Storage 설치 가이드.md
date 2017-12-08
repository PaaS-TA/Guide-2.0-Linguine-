## Table of Contents
1. [문서 개요](#1)
     * [1.1. 목적](#2)
     * [1.2. 범위](#3)
     * [1.3. 시스템 구성도](#4)
     * [1.4. 참고자료](#5)
2. [Object Storage 설치](#6)
     * [2.1. 설치전 준비사항](#7)
     * [2.2. Object Strorage 릴리즈 업로드](#8)
     * [2.3. Object Strorage Deployment 파일 수정 및 배포](#9)



# <div id='1'> 1. 문서 개요



### <div id='2'> 1.1. 목적
본 문서(포털 Object Storage 설치 가이드)는 PaaS-TA 포털에서 사용하는 Object Storage의 설치를 Bosh를 이용하여 설치 하는 방법을 기술하였다.



### <div id='3'> 1.2. 범위
설치 범위는 Object Storage 사용을 검증하기 위한 기본 설치를 기준으로 작성하였다.



### <div id='4'> 1.3. 시스템 구성도
본 문서에서 설명하는 Object Storage의 시스템 설치 구성도이다. OpenStack Swift의 간편한 설치를 지원하는 Swift All In One과 인증처리를 위한 Keystone으로 기본 최소사항을 구성하였다.
※	Openstack 환경에서는 Portal Object Storage를 설치할 필요없이, OpenStack이 제공하는 Openstack Swift 서비스를 이용할 수 있다.

![시스템 구성도][object_storage_image_01]

<table>
  <tr>
    <td>구분</td>
    <td>스펙</td>
  </tr>
  <tr>
    <td>swift-ketstone</td>
    <td>1vCPU / 2GB RAM / 4GB Disk</td>
  </tr>
</table>



### <div id='5'> 1.4. 참고자료
[**http://bosh.io/docs**](http://bosh.io/docs)
[**http://docs.cloudfoundry.org/**](http://docs.cloudfoundry.org/)
[**http://docs.openstack.org/developer/swift/**](http://docs.openstack.org/developer/swift/)



# <div id='6'> 2. Object Storage 설치



### <div id='7'> 2.1. 설치전 준비사항
본 설치 가이드는 Linux 환경에서 설치하는 것을 기준으로 하였다.
서비스팩 설치를 위해서는 먼저 BOSH CLI 가 설치 되어 있어야 하고 BOSH 에 로그인 및 target 설정이 되어 있어야 한다.
BOSH CLI 가 설치 되어 있지 않을 경우 먼저 BOSH 설치 가이드 문서를 참고 하여 BOSH CLI를 설치 해야 한다.

-	PaaS-TA에서 제공하는 포털 릴리즈 파일들을 다운받는다. (PaaSTA-Portal.zip)

- 다운로드 위치
> PaaSTA-Portal : **<https://paas-ta.kr/data/packages/2.0/PaaSTA-Portal.zip>**  
> PaaSTA-Deployment : **<https://paas-ta.kr/data/packages/2.0/PaaSTA-Deployment.zip>**



### <div id='8'> 2.2. Object Storage 릴리즈 업로드

-	PaaSTA-Portal.zip의 압축을 풀고 폴더 안에 있는 파스타 포털 Object Storage 릴리즈 paasta-portal-object-storage-2.0.tgz 파일을 확인한다.
```
$ ls --all
```
```
.   카탈로그이미지  api2          paasta-portal-object-storage-2.0.tgz  registraion  web-admin
..  api             auto-scaling  postgresql
```


-	업로드 되어 있는 릴리즈 목록을 확인한다.
```
$ bosh releases
```
```
RSA 1024 bit CA certificates are loaded due to old openssl compatibility
Acting as user 'admin' on 'my-bosh'

+-----------------+----------+-------------+
| Name            | Versions | Commit Hash |
+-----------------+----------+-------------+
| empty-release   | 2.0      | 870201f29+  |
+-----------------+----------+-------------+
(+) Uncommitted changes

Releases total: 1
```
Portal Object Storage 릴리즈가 업로드 되어 있지 않은 것을 확인


-	Object Storage 릴리즈 파일을 업로드한다
```
$ bosh upload release paasta-portal-object-storage-2.0.tgz
```
```
RSA 1024 bit CA certificates are loaded due to old openssl compatibility
Acting as user 'admin' on 'bosh'

Verifying manifest...
Extract manifest                                             OK
Manifest exists                                              OK
Release name/version                                         OK

File exists and readable                                     OK
Read package 'python' (1 of 3)                               OK
Package 'python' checksum                                    OK
Read package 'mysql-5.6' (2 of 3)                            OK
Package 'mysql-5.6' checksum                                 OK
Read package 'swift-all-in-one' (3 of 3)                     OK
Package 'swift-all-in-one' checksum                          OK
Package dependencies                                         OK
Checking jobs format                                         OK
Read job 'swift-keystone' (1 of 1), version aeb8ab57e095d146b8d18a2c19e0c448422879ac OK
Job 'swift-keystone' checksum                                OK
Extract job 'swift-keystone'                                 OK
Read job 'swift-keystone' manifest                           OK
Check template 'bin/install.sh.erb' for 'swift-keystone'     OK
Check template 'bin/swift_ctl.erb' for 'swift-keystone'      OK
Check template 'bin/mysql_ctl.erb' for 'swift-keystone'      OK
Check template 'helper/install_deb.sh.erb' for 'swift-keystone' OK
Check template 'helper/mysql_install.sh.erb' for 'swift-keystone' OK
Check template 'helper/swift_set_up.sh.erb' for 'swift-keystone' OK
Check template 'config/mysql/my-default.cnf' for 'swift-keystone' OK
Check template 'config/keystone_install/keystone_install.sh.erb' for 'swift-keystone' OK
Check template 'config/keystone_install/config_admin_port' for 'swift-keystone' OK
Check template 'config/keystone_install/config_public_port' for 'swift-keystone' OK
Check template 'config/keystone_install/config_service_token' for 'swift-keystone' OK
Check template 'config/keystone_install/keystone.conf-init' for 'swift-keystone' OK
Check template 'config/keystone_install/keystone_middleware_settings' for 'swift-keystone' OK
Check template 'config/keystone_install/keystone-init.d' for 'swift-keystone' OK
Check template 'config/keystone_install/service_endpoint' for 'swift-keystone' OK
Check template 'config/keystone_install/tenant_token' for 'swift-keystone' OK
Check template 'config/keystone_install/default_keystone_config.sh' for 'swift-keystone' OK
Check template 'config/keystone/etc/default_catalog.templates' for 'swift-keystone' OK
Check template 'config/keystone/etc/keystone.conf.sample' for 'swift-keystone' OK
Check template 'config/keystone/etc/keystone-paste.ini' for 'swift-keystone' OK
Check template 'config/keystone/etc/logging.conf.sample' for 'swift-keystone' OK
Check template 'config/keystone/etc/policy.json' for 'swift-keystone' OK
Check template 'config/keystone/etc/policy.v3cloudsample.json' for 'swift-keystone' OK
Check template 'config/keystone/etc/sso_callback_template.html' for 'swift-keystone' OK
Job 'swift-keystone' needs 'python' package                  OK
Job 'swift-keystone' needs 'swift-all-in-one' package        OK
Job 'swift-keystone' needs 'mysql-5.6' package               OK
Monit file for 'swift-keystone'                              OK

Release info
------------
Name:    paasta-portal-object-storage
Version: 2.0

Packages
 - python (4e255efa754d91b825476b57e111345f200944e1)
 - mysql-5.6 (eb25e13462585cbc738ef7c171d0ad56f3228d3b)
 - swift-all-in-one (47721449a8e511e6e8e73676bcb44ad1c35c1cdc)

Jobs
 - swift-keystone (aeb8ab57e095d146b8d18a2c19e0c448422879ac)

License
 - none

Checking if can repack release for faster upload...
python (4e255efa754d91b825476b57e111345f200944e1) UPLOAD
mysql-5.6 (eb25e13462585cbc738ef7c171d0ad56f3228d3b) UPLOAD
swift-all-in-one (47721449a8e511e6e8e73676bcb44ad1c35c1cdc) UPLOAD
swift-keystone (aeb8ab57e095d146b8d18a2c19e0c448422879ac) UPLOAD
Uploading the whole release

Uploading release
paasta-portal:  96% |ooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooo   | 370.7MB  25.4MB/s ETA:  00:00:00
Director task
 Started extracting release > Extracting release. Done (00:00:04)

 Started verifying manifest > Verifying manifest. Done (00:00:00)

 Started resolving package dependencies > Resolving package dependencies. Done (00:00:00)

 Started creating new packages
 Started creating new packages > python/4e255efa754d91b825476b57e111345f200944e1. Done (00:00:00)
 Started creating new packages > mysql-5.6/eb25e13462585cbc738ef7c171d0ad56f3228d3b. Done (00:00:06)
 Started creating new packages > swift-all-in-one/47721449a8e511e6e8e73676bcb44ad1c35c1cdc. Done (00:00:01)
    Done creating new packages (00:00:07)

 Started creating new jobs > swift-keystone/aeb8ab57e095d146b8d18a2c19e0c448422879ac. Done (00:00:00)

 Started release has been created > paasta-portal-object-storage/2.0. Done (00:00:00)

Task 2420 done

Started		2017-01-13 07:43:31 UTC
Finished	2017-01-13 07:43:42 UTC
Duration	:00:11
paasta-portal:  96% |ooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooo   | 371.6MB  13.7MB/s Time: 00:00:27

Release uploaded
```


-	업로드 된 Object Storage 릴리즈를 확인한다.
```
$ bosh releases
```
```
RSA 1024 bit CA certificates are loaded due to old openssl compatibility
Acting as user 'admin' on 'bosh'

+------------------------------+----------+-------------+
| Name                         | Versions | Commit Hash |
+------------------------------+----------+-------------+
| cflinuxfs2-rootfs            | 1.40.0   | 19fe09f4+   |
| empty-release                | 1+dev.1* | 00000000    |
| etcd                         | 86       | 2dfbef00+   |
| paasta-container             | 2.0      | b857e171    |
| paasta-controller            | 2.0*     | 0f315314    |
| paasta-garden-runc           | 2.0      | ea5f5d4d+   |
| paasta-influxdb-grafana      | 2.0*     | 00000000    |
| paasta-logsearch             | 2.0*     | 00000000    |
| paasta-metrics-collector     | 2.0*     | 00000000    |
| paasta-monitoring-api-server | 2.0      | 00000000    |
| paasta-portal-object-storage | 2.0      | 00000000    |
| paasta-redis                 | 2.0      | 2d766084+   |
| paasta-web-ide               | 2.0      | 00000000    |
+------------------------------+----------+-------------+
(*) Currently deployed
(+) Uncommitted changes

Releases total: 13

```



### <div id='9'> 2.3. Object Storage Deployment 파일 수정 및 배포
BOSH Deployment manifest 는 components 요소 및 배포의 속성을 정의한 YAML 파일이다.
Deployment manifest 에는 sotfware를 설치 하기 위해서 어떤 Stemcell (OS, BOSH agent)을 사용 할 것인지와 Release(Software packages, Config templates, Scripts)의 이름과 버전, VMs 용량, Jobs params 등이 정의 되어 있다.


-	PaaSTA-Deployment.zip 파일 압축을 풀고 폴더안에 있는 IaaS별 Portal Object Storage Deployment 파일을 복사한다.
예) vsphere 일 경우 paasta_portal_object_storage_vsphere_2.0.yml를 복사



-	Director UUID를 확인한다.
BOSH CLI가 배포에 대한 모든 작업을 허용하기 위한 현재 대상 BOSH Director의 UUID와 일치해야 한다. ‘bosh status’ CLI 을 통해서 현재 BOSH Director 에 target 되어 있는 UUID를 확인할 수 있다.

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


-	Deploy시 사용할 Stemcell을 확인한다.
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


-	Deployment 파일을 서버 환경에 맞게 수정한다. (vsphere 용으로 설명, 다른 IaaS는 해당 Deployment 파일의 주석 내용을 참고)

```
yaml
# paasta_portal_object_storage_vsphere_2.0.yml 설정 파일 내용
---
name: paasta-portal-object-storage                    # 서비스 배포 이름 (필수)
director_uuid: d363905f-eaa0-4539-a461-8c1318498a32   # bosh status로 확인한 Director UUID

releases:                                             
- name: paasta-portal-object-storage                  # 서비스 릴리즈 이름(필수)
  version: latest                                     # 서비스 릴리즈 버전(필수): latest 시 업로드된 서비스 릴리즈 최신버전

update:
  canaries: 0                                         # canary 인스턴스 수(필수)
  canary_watch_time: 30000-240000                     # canary 인스턴스가 수행하기 위한 대기 시간(필수)
  max_in_flight: 1                                    # non-canary 인스턴스가 병렬로 update 하는 최대 개수(필수)     
  serial: true
  update_watch_time: 30000-240000                     # non-canary 인스턴스가 수행하기 위한 대기 시간(필수)

compilation:                                          # 컴파일시 필요한 가상머신의 속성(필수)
  cloud_properties:                                   # 컴파일 VM을 만드는 데 필요한 IaaS의 특정 속성 (instance_type, availability_zone), 직접 cpu,disk,ram 사이즈를 넣어도 됨
    cpu: 4                                             
    disk: 20480
    ram: 4096
  network: default
  reuse_compilation_vms: false
  workers: 4                                          # 컴파일 하는 가상머신의 최대수(필수)

resource_pools:                                       # 배포시 사용하는 resource pools를 명시하며 여러 개의 resource pools 을 사용할 경우 name 은 unique 해야함(필수)
- cloud_properties:                                   # 컴파일 VM을 만드는 데 필요한 IaaS의 특정 속성을 설명 (instance_type, availability_zone), 직접 cpu, disk, 메모리 설정가능
    cpu: 1
    disk: 4096
    ram: 2048
  name: swift-keystone                                # 고유한 resource pool 이름
  network: default
  stemcell:
    name: bosh-vsphere-esxi-ubuntu-trusty-go_agent    # 사용할 stemcell 이름(필수)
    version: 3263.8                                   # stemcell 버전(필수)

jobs:
- instances: 1                                  # job 인스턴스 수(필수)
  name: swift-keystone                          # 작업 이름(필수)
  networks:                                     # 네트워크 구성정보
  - name: default                               # Networks block에서 선언한 network 이름(필수)
    static_ips:
    - 10.30.131.12                              # 사용할 IP addresses 정의(필수)
  persistent_disk: 2048                         # object storage 저장 공간 크기(필수)
  resource_pool: swift-keystone                 # resource_pools block에 정의한 resource pool 이름(필수)
  templates:
  - name: swift-keystone                        # job template 이름(필수)

networks:                                       # 네트워크 블록에 나열된 각 서브 블록이 참조 할 수있는 작업이 네트워크 구성을 지정, 네트워크 구성은 네트워크 담당자에게 문의 하여 작성 요망
- name: default                                 # vsphere 에서 사용하는 network 이름(필수)
  subnets:
  - cloud_properties:
      name: Internal
    dns:
    - 10.30.20.24
    - 8.8.8.8
    gateway: 10.30.20.23
    name: default_unused
    range: 10.30.0.0/16
    reserved:                                   # 설치시 제외할 IP 설정
    - 10.30.0.1 - 10.30.0.5
    static:
    - 10.30.131.12                              # 사용 가능한 IP 설정
  type: manual

properties:
  proxy_ip: 10.30.131.12                      # 프록시 서버  IP  (swift-keystone job의 static_ip, Object Storage 접속 IP)
  proxy_port: 10008                           # 프록시 서버 Port (Object Storage 접속 Port)
  keystone_username: paasta-portal            # 최초 생성되는 유저이름(Object Storage 접속 유저이름)
  keystone_password: paasta                   # 최초 생성되는 유저 비밀번호(Object Storage 접속 유저 비밀번호)
  keystone_tenantname: paasta-portal          # 최초 생성되는 테넌트 이름(Object Storage 접속 테넌트 이름)
  keystone_auth_port: 5000                    # Keystone 인증을 위해 사용하는 포트
  keystone_email: email@email.com             # 최소 생성되는 유저의 이메일
```


-    Deploy 할 deployment manifest 파일을 BOSH 에 지정한다.
```
$ bosh deployment paasta_portal_object_storage_vsphere_2.0.yml
```
```
RSA 1024 bit CA certificates are loaded due to old openssl compatibility
Deployment set to '/home/inception/bosh-space/kimdojun/swift/paasta_portal_object_storage_vsphere_2.0.yml'
```


-	Object Storage를 배포한다.
※	서버 사양에 따라 5분 ~ 20분 정도 소요된다.
```
$ bosh deploy
```
```
RSA 1024 bit CA certificates are loaded due to old openssl compatibility
Acting as user 'admin' on deployment 'paasta-portal-object-storage' on 'bosh'
Getting deployment properties from director...
Unable to get properties list from director, trying without it...

Detecting deployment changes
----------------------------

...<manifest 파일 내용 출력// 생략 >...

Please review all changes carefully

Deploying
---------
Are you sure you want to deploy? (type 'yes' to continue): yes

Director task
Deprecation: Ignoring cloud config. Manifest contains 'networks' section.

 Started preparing deployment > Preparing deployment. Done (00:00:00)

 Started preparing package compilation > Finding packages to compile. Done (00:00:00)

 Started compiling packages
 Started compiling packages > swift-all-in-one/47721449a8e511e6e8e73676bcb44ad1c35c1cdc
 Started compiling packages > mysql-5.6/eb25e13462585cbc738ef7c171d0ad56f3228d3b
 Started compiling packages > python/4e255efa754d91b825476b57e111345f200944e1
    Done compiling packages > swift-all-in-one/47721449a8e511e6e8e73676bcb44ad1c35c1cdc(00:01:45)
    Done compiling packages > mysql-5.6/eb25e13462585cbc738ef7c171d0ad56f3228d3b(00:03:25)
    Done compiling packages > python/4e255efa754d91b825476b57e111345f200944e1(00:04:44)
    Done compiling packages (00:04:44)

 Started creating missing vms > swift-keystone/da8b5443-92a3-44a8-a9a7-769407b501a2 (0). Done (00:01:09)

 Started updating instance swift-keystone> swift-keystone/da8b5443-92a3-44a8-a9a7-769407b501a2 (0). Done (00:04:05)

Task 2430 done

Started		2017-01-13 08:04:45 UTC
Finished	2017-01-13 08:14:43 UTC
Duration	:09:58

Deployed 'paasta-portal-object-storage' to 'bosh'
```


-	배포된 Object Storage를 확인한다.
```
$ bosh vms paasta-portal-object-storage
```
```
RSA 1024 bit CA certificates are loaded due to old openssl compatibility
Acting as user 'admin' on deployment 'paasta-portal-object-storage' on 'bosh'

Director task

Task 2486 done

+---------------------------------------------------------+---------+-----+----------------+--------------+
| VM                                                      | State   | AZ  | VM Type        | IPs          |
+---------------------------------------------------------+---------+-----+----------------+--------------+
| swift-keystone/0 (dea97c2b-10d2-4655-a867-fc1c781340c3) | running | n/a | swift-keystone | 10.30.131.12 |
+---------------------------------------------------------+---------+-----+----------------+--------------+

VMs total: 1
```

[object_storage_image_01]:/images/paasta-portal/portal-object-storage/object_storage_image_01.png
