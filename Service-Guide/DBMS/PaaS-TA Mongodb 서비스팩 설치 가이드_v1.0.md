## Table of Contents
1. [문서 개요](#1)
     * [1.1. 목적](#2)
     * [1.2. 범위](#3)
     * [1.3. 시스템 구성도](#4)
     * [1.4. 참고자료](#5)
2. [Mongodb 서비스팩 설치](#6)
     * [2.1. 설치전 준비사항](#7)
     * [2.2. Mongodb 서비스 릴리즈 업로드](#8)
     * [2.3. Mongodb 서비스 Deployment 파일 수정 및 배포](#9)
     * [2.4. Mongodb 서비스 브로커 등록](#10)
3. [Mongodb 연동 Sample App 설명](#11)
     * [3.1. Sample App 구조](#12)
     * [3.2. PaaS-TA에서 서비스 신청](#13)
     * [3.3. Sample App에 서비스 바인드 신청 및 App 확인](#14)
4. [Mongodb Client 툴 접속](#15)
     * [4.1. MongoChef 설치 & 연결](#16)

<br>
<div id ='1'></div>
#  1.  문서 개요

<br>
<div id='2'></div>
### 1.1. 목적

본 문서(Mongodb 서비스팩 설치 가이드)는 전자정부표준프레임워크 기반의 PaaS-TA에서 제공되는 서비스팩인 Mongodb 서비스팩을 Bosh를 이용하여 설치 하는 방법과 PaaS-TA의 SaaS 형태로 제공하는 Application 에서 Mongodb 서비스를 사용하는 방법을 기술하였다.

<br>
<div id='3'></div>
### 1.2. 범위 

설치 범위는 Mongodb 서비스팩을 검증하기 위한 기본 설치를 기준으로 작성하였다. 

<br>
<div id='4'></div>
### 1.3. 시스템 구성도

본 문서의 설치된 시스템 구성도입니다. Mongodb Server, Mongodb 서비스 브로커로 최소사항을 구성하였다.

![시스템구성도][pinpoint_image_02]

<table>
  <tr>
    <td>구분</td>
    <td>스펙</td>
  </tr>
  <tr>
    <td>PaaS-TA-mongodb-broker</td>
    <td>1vCPU / 1GB RAM / 8GB Disk</td>
  </tr>
  <tr>
    <td>Mongos</td>
    <td>1vCPU / 1GB RAM / 8GB Disk</td>
  </tr>
  <tr>
    <td>Mongo Config</td>
    <td>1vCPU / 1GB RAM / 8GB Disk+4GB(영구적 Disk)</td>
  </tr>
  <tr>
    <td>Mongod</td>
    <td>1vCPU / 1GB RAM /8GB Disk+4GB(영구적 Disk)</td>
  </tr>
</table>

<br>
<div id='5'></div>
### 1.4. 참고자료

[*http://bosh.io/docs*](http://bosh.io/docs)

[*http://docs.cloudfoundry.org/*](http://docs.cloudfoundry.org/)

<br>
<div id='6'></div>
#  2.  Pinpoint 서비스팩 설치

<br>
<div id='7'></div>
### 2.1.  설치전 준비사항
본 서비스팩 설치를 위해서는 먼저 BOSH CLI 가 설치 되어 있어야 하고 BOSH 에 로그인 및 타켓 설정이 되어 있어야 한다.
BOSH CLI 가 설치 되어 있지 않을 경우 먼저 BOSH 설치 가이드 문서를 참고 하여 BOSH CLI를 설치 해야 한다.

- PaaS-TA 에서 제공하는 압축된 릴리즈 파일들을 다운받는다. (PaaS-TA-Services.zip, PaaS-TA-Deployment.zip, PaaS-TA-Sample-Apps.zip)

<br>
<div id='8'></div>
### 2.2. Mongodb 서비스 릴리즈 업로드

- PaaS-TA-Services.zip 파일 압축을 풀고 폴더안에 있는 Mongodb 서비스 릴리즈 pasta-mongodb-shard-2.0.tgz 파일을 확인한다.

```
$ ls –all
```
```
-rw-rw-r-- 1 ubuntu ubuntu 121273779 Jan 16 04:05 paasta-mongodb-shard-2.0.tgz
```

<br>
-  업로드 되어 있는 릴리즈 목록을 확인한다.
```
$ bosh releases
```
```
+--------------------+----------------+-------------+
| Name               | Versions       | Commit Hash |
+--------------------+----------------+-------------+
| cf-monitoring      | 0+dev.1*       | 00000000    |
| cflinuxfs2-rootfs  | 1.40.0*        | 19fe09f4+   |
| etcd               | 86*            | 2dfbef00+   |
| logsearch          | 203.0.0+dev.1* | 00000000    |
| metrics-collector  | 0+dev.1*       | 00000000    |
| paasta-container   | 0+dev.1*       | b857e171    |
| paasta-controller  | 0+dev.1*       | 0f315314    |
| paasta-garden-runc | 2.0*           | ea5f5d4d+   |
+--------------------+----------------+-------------+
(*) Currently deployed
(+) Uncommitted changes
```
Mongodb 서비스 릴리즈가 업로드 되어 있지 않은 것을 확인

<br>
- Mongodb 서비스 릴리즈를 업로드한다.
```
$ bosh upload release paasta-mongodb-shard-2.0.tgz
```
```
Uploading release
paasta-mongod:  96% |oooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooo     | 111.0MB  22.9MB/s ETA:  00:00:00
Director task 692
  Started extracting release > Extracting release. Done (00:00:01)

  Started verifying manifest > Verifying manifest. Done (00:00:00)

  Started resolving package dependencies > Resolving package dependencies. Done (00:00:00)

  Started creating new packages
  Started creating new packages > mongodb_broker/d547d39098e73acb70d58ab2be2c18c2410dfa5b. Done (00:00:01)
  Started creating new packages > java7/cb28502f6e89870255182ea76e9029c7e9ec1862. Done (00:00:01)
  Started creating new packages > cli/24305e50a638ece2cace4ef4803746c0c9fe4bb0. Done (00:00:00)
  Started creating new packages > mongodb/b355ac045b257e6a0cec85874c6fb6e7abe92b6d. Done (00:00:00)
     Done creating new packages (00:00:02)

  Started creating new jobs
  Started creating new jobs > mongodb_slave/cd18c5187f44f8e3d1d2c7937047cc748a851a43. Done (00:00:00)
  Started creating new jobs > mongodb_broker/10da2f3c0e374b01f818b28ff5ecb8044fd0cd1a. Done (00:00:00)
  Started creating new jobs > mongodb_config/dcb9c707d4e9757a150f540ee5af39efb8580f04. Done (00:00:01)
  Started creating new jobs > mongodb_master/adfc199c9d2f3aceaf31fe56e553e15faf605ee7. Done (00:00:00)
  Started creating new jobs > mongodb_broker_deregistrar/d797f068e89265313436b7c6439d93288d0fafbe. Done (00:00:00)
  Started creating new jobs > mongodb_shard/a549bee23d326211549a2dce9def42d85b655e4d. Done (00:00:00)
  Started creating new jobs > mongodb_broker_registrar/a4892a7dfec7acdc7ba0cd2618a79ee3b2f80d9b. Done (00:00:00)
     Done creating new jobs (00:00:01)

  Started release has been created > paasta-mongodb-shard/2.0. Done (00:00:00)

Task 692 done

Started   2017-01-16 04:16:20 UTC
Finished  2017-01-16 04:16:24 UTC
Duration  00:00:04
paasta-mongod:  96% |oooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooo     | 111.3MB  11.0MB/s Time: 00:00:10

Release uploaded
```

<br>
-   업로드 되어 있는 릴리즈 목록을 확인한다.

```
$ bosh releases
```
```
Acting as user 'admin' on 'my-bosh'

+----------------------+----------------+-------------+
| Name                 | Versions       | Commit Hash |
+----------------------+----------------+-------------+
| cf-monitoring        | 0+dev.1*       | 00000000    |
| cflinuxfs2-rootfs    | 1.40.0*        | 19fe09f4+   |
| etcd                 | 86*            | 2dfbef00+   |
| logsearch            | 203.0.0+dev.1* | 00000000    |
| metrics-collector    | 0+dev.1*       | 00000000    |
| paasta-container     | 0+dev.1*       | b857e171    |
| paasta-controller    | 0+dev.1*       | 0f315314    |
| paasta-garden-runc   | 2.0*           | ea5f5d4d+   |
| paasta-mongodb-shard | 2.0*           | 85e3f01e+   |
+----------------------+----------------+-------------+
(*) Currently deployed
(+) Uncommitted changes

Releases total: 9
```
Mongodb 서비스 릴리즈가 업로드 되어 있는 것을 확인

<br>
<div id='9'></div>
###  2.3. Mongodb 서비스 Deployment 파일 수정 및 배포

BOSH Deployment manifest 는 components 요소 및 배포의 속성을 정의한 YAML  파일이다.
Deployment manifest 에는 sotfware를 설치 하기 위해서 어떤 Stemcell (OS, BOSH agent) 을 사용할것이며 Release (Software packages, Config templates, Scripts) 이름과 버전, VMs 용량, Jobs params 등을 정의가 되어 있다.

- PaaS-TA-Deployment.zip 파일 압축을 풀고 폴더안에 있는 vSphere 용 Mongodb Deployment 화일인 paasta-mongodb-shard-vsphere-2.0.yml 를 복사한다.
다운로드 받은 Deployment Yml 파일을 확인한다. (paasta-mongodb-shard-vsphere-2.0.yml)

```
$ ls –all
```
![mongodb_image_02]

<br>
- Director UUID를 확인한다.
BOSH CLI가 배포에 대한 모든 작업을 허용하기 위한 현재 대상 BOSH Director의 UUID와 일치해야한다. ‘bosh status’ CLI 을 통해서 현재 BOSH Director 에 target 되어 있는 UUID를 확인할수 있다.

```
$ bosh status
```

![mongodb_image_03]

<br>
- Deploy시 사용할 Stemcell을 확인한다.

```
$ bosh stemcells
```
![mongodb_image_04]
Stemcell 목록이 존재 하지 않을 경우 BOSH 설치 가이드 문서를 참고 하여 Stemcell을 업로드 해야 한다.

<br>
-    Deployment 파일을 서버 환경에 맞게 수정한다. (vsphere 용으로 설명, 다른 IaaS는 해당 Deployment 파일의 주석내용을 참고)

```yaml
$vi openpaas-paasta-pinpoint-cluster-vsphere-1.0.yml

# openpaas-paasta-pinpoint-vSphere-1.0설정 파일 내용
<% num_collectors = 2 %>
<% num_slaves = 2 %>
<% num_webui = 2 %>
<% h_master_ip = '10.30.70.75' %>
<% h_secondary_ip = '10.30.70.76' %>
#<% h_slave_ips = ["10.30.70.73", "10.30.70.74", "10.30.70.75"] %>
<% h_slave_ips = ["10.30.70.73", "10.30.70.74"] %>
#<% collector_ips = ["10.30.70.40","10.30.70.41","10.30.70.43"] %>
<% collector_ips = ["10.30.70.40","10.30.70.41"] %>
<% haproxy_webui_ip = '10.30.70.78' %>
<% haproxy_webui_public_ip = '115.68.46.182' %>
<% haproxy_webui_http_port = '80' %>
#<% webui_ips = ["10.30.70.79", "10.30.70.80", "10.30.70.81"] %>
<% webui_ips = ["10.30.70.79", "10.30.70.80"] %>
<% broker_ip = '10.30.70.82' %>
<% tcp_listen_port = '29994' %>
<% udp_stat_listen_port = '29995' %>
<% udp_span_listen_port = '29996' %>
---
name: openpaas-paasta-pinpoint
director_uuid: d363905f-eaa0-4539-a461-8c1318498a32
release:
  name: paasta-pinpoint-cluster
  version: latest
compilation:
  workers: 6
  network: default
  cloud_properties:
    ram: 4096
    disk: 8096
    cpu: 4
update:
  canaries: 1
  canary_watch_time: 120000
  update_watch_time: 120000
  max_in_flight: 8
 
networks:
- name: default
  subnets:
  - cloud_properties:
      name: Internal
    dns:
    - 10.30.20.24
    - 8.8.8.8
    gateway: 10.30.20.23
    name: default_unused
    range: 10.30.0.0/16
    reserved:
    - 10.30.20.0 - 10.30.20.22
    - 10.30.20.24 - 10.30.20.255
    - 10.30.40.0 - 10.30.40.255
    - 10.30.60.0 - 10.30.60.112
    static:
    - 10.30.70.0 - 10.30.70.255 
- name: public_network
  type: manual
  subnets:
  - cloud_properties:
      name: External
    dns:
    - 8.8.8.8
    range: 115.68.46.176/28
    gateway: 115.68.46.177
    static:
    - <%= haproxy_webui_public_ip %>
 
resource_pools:
  - name: pinpoint_small 
    network: default
    stemcell:
      name: bosh-vsphere-esxi-ubuntu-trusty-go_agent
      version: 3263.8 
    cloud_properties:
      cpu: 1
      disk: 4096
      ram: 1024
  - name: pinpoint_medium
    network: default
    stemcell:
      name: bosh-vsphere-esxi-ubuntu-trusty-go_agent
      version: 3263.8
    cloud_properties:
      cpu: 2
      disk: 8192
      ram: 2048
jobs:
- name: h_slave
  template: h_slave
  instances: <%= num_slaves %>
  resource_pool: pinpoint_small
  networks:
  - name: default
    static_ips:
    <% h_slave_ips.each do |ip| %>
    - <%= ip %>
    <% end %>
- name: h_secondary
  template: h_secondary
  instances: 1
  resource_pool: pinpoint_small
  networks:
  - name: default
    static_ips:
    - <%= h_secondary_ip %>
- name: h_master
  template: h_master
  instances: 1
  resource_pool: pinpoint_medium
  networks:
  - name: default
    static_ips:
    - <%= h_master_ip %>
- name : h_master_register
  template : h_master_register
  instances: 1
  lifecycle: errand   
  resource_pool: pinpoint_small
  networks:
  - name: default 
- name: collector
  template: collector
  instances: <%= num_collectors %>
  resource_pool: pinpoint_medium
  networks:
  - name: default
    static_ips:
    <% collector_ips.each do |ip| %>
    - <%= ip %>
    <% end %>
- name: haproxy_webui
  template: haproxy_webui
  instances: 1
  resource_pool: pinpoint_small
  networks:
  - name: default
    static_ips:
    - <%= haproxy_webui_ip %>
  - name: public_network
    default: [dns, gateway]
    static_ips: <%= haproxy_webui_public_ip %>
- name: webui
  template: webui
  instances: <%= num_webui %>
  resource_pool: pinpoint_small
  networks:
  - name: default
    static_ips:
    <% webui_ips.each do |ip| %>
    - <%= ip %>
    <% end %>
- name: pinpoint_broker
  template: broker
  instances: 1
  resource_pool: pinpoint_small
  networks:
  - name: default
    static_ips:
    - <%= broker_ip %>
 
properties:
  master:
    host_name: h-master
    host_ip: <%= h_master_ip %>
    port: 9000
    http_port: 50070
    replication: <%= num_slaves %>
    tcp_listen_port: <%= tcp_listen_port %>
    udp_stat_listen_port: <%= udp_stat_listen_port %>
    udp_span_listen_port: <%= udp_span_listen_port %>
  broker:
    collector_ips: <%= collector_ips %>
    collector_tcp_port: <%= tcp_listen_port %>
    collector_stat_port: <%= udp_stat_listen_port %>
    collector_span_port: <%= udp_span_listen_port %>
    dashboard_uri: http://<%= haproxy_webui_public_ip %>:<%= haproxy_webui_http_port %>/#/main
  secondary:
    host_name: h-secondary
    host_ip: <%= h_secondary_ip %>
    http_port: 50090
  yarn:
    host_name: h-master
    host_ip: <%= h_master_ip %>
    resource_tracker_port: 8025
    scheduler_port: 8030
    resourcemanager_port: 8040
  slave:
    host_names:
      <% num_slaves.times do |i| %>
      <%= "- h-slave#{i}" %>
      <% end %>
    host_ips: 
      <% h_slave_ips.each do |ip| %>
      - <%= ip %>
      <% end %>
  collector:
    host_names:
      <% num_collectors.times do |i| %>
      <%= "- h-collector#{i}" %>
      <% end %>
    host_ips: 
      <% collector_ips.each do |ip| %>
      - <%= ip %>
      <% end %>
  haproxy:
    host_name: haproxy-webui
    host_ip: <%= haproxy_webui_ip %>
    http_port: <%= haproxy_webui_http_port %>
  webui:
    host_names:
      <% num_webui.times do |i| %>
      <%= "- h-webui#{i}" %>
      <% end %>
    host_ips: 
      <% webui_ips.each do |ip| %>
      - <%= ip %>
      <% end %>
```

<br>
-   Deploy 할 deployment manifest 파일을 BOSH 에 지정한다.

  ```sh
  $bosh deployment {Deployment manifest 파일 PATH}
  $bosh deployment openpaas-paasta-pinpoint-cluster-vsphere-1.0.yml
  
  RSA 1024 bit CA certificates are loaded due to old openssl compatibility
  ```

<br>
-   Pinpoint 서비스팩을 배포한다.

  ```sh
  $ bosh deploy
  
  RSA 1024 bit CA certificates are loaded due to old openssl compatibility
  Acting as user 'admin' on deployment 'openpaas-paasta-pinpoint' on 'bosh'
  Getting deployment properties from director...
  Unable to get properties list from director, trying without it...

  Detecting deployment changes
  ----------------------------
  resource_pools:
  - name: pinpoint_small
    network: default
    stemcell:
      name: bosh-vsphere-esxi-ubuntu-trusty-go_agent
      version: '3263.8'
    cloud_properties:
      cpu: 1
      disk: 4096
      ram: 1024
  - name: pinpoint_medium
    network: default
    stemcell:
      name: bosh-vsphere-esxi-ubuntu-trusty-go_agent
      version: '3263.8'
    cloud_properties:
      cpu: 2
      disk: 8192
      ram: 2048

  compilation:
    workers: 6
    network: default
    cloud_properties:
      ram: 4096
      disk: 8096
      cpu: 4

  networks:
  - name: default
    subnets:
    - cloud_properties:
        name: Internal
      dns:
      - 10.30.20.24
      - 8.8.8.8
      gateway: 10.30.20.23
      name: default_unused
      range: 10.30.0.0/16
      reserved:
      - 10.30.20.0 - 10.30.20.22
      - 10.30.20.24 - 10.30.20.255
      - 10.30.40.0 - 10.30.40.255
      - 10.30.60.0 - 10.30.60.112
      static:
      - 10.30.70.0 - 10.30.70.255
  - name: public_network
    type: manual
    subnets:
    - cloud_properties:
        name: External
      dns:
      - 8.8.8.8
      range: 115.68.46.176/28
      gateway: 115.68.46.177
      static:
      - 115.68.46.182

  update:
    canaries: 1
    canary_watch_time: 120000
    update_watch_time: 120000
    max_in_flight: 8

  jobs:
  - name: h_slave
    template: h_slave
    instances: 2
    resource_pool: pinpoint_small
    networks:
    - name: default
      static_ips:
      - 10.30.70.73
      - 10.30.70.74
  - name: h_secondary
    template: h_secondary
    instances: 1
    resource_pool: pinpoint_small
    networks:
    - name: default
      static_ips:
      - 10.30.70.76
  - name: h_master
    template: h_master
    instances: 1
    resource_pool: pinpoint_medium
    networks:
    - name: default
      static_ips:
      - 10.30.70.75
  - name: h_master_register
    template: h_master_register
    instances: 1
    lifecycle: errand
    resource_pool: pinpoint_small
    networks:
    - name: default
  - name: collector
    template: collector
    instances: 2
    resource_pool: pinpoint_medium
    networks:
    - name: default
      static_ips:
      - 10.30.70.40
      - 10.30.70.41
  - name: haproxy_webui
    template: haproxy_webui
    instances: 1
    resource_pool: pinpoint_small
    networks:
    - name: default
      static_ips:
      - 10.30.70.78
    - name: public_network
      default:
      - dns
      - gateway
      static_ips: 115.68.46.182
  - name: webui
    template: webui
    instances: 2
    resource_pool: pinpoint_small
    networks:
    - name: default
      static_ips:
      - 10.30.70.79
      - 10.30.70.80
  - name: pinpoint_broker
    template: broker
    instances: 1
    resource_pool: pinpoint_small
    networks:
    - name: default
      static_ips:
      - 10.30.70.82

  name: openpaas-paasta-pinpoint

  director_uuid: d363905f-eaa0-4539-a461-8c1318498a32

  release:
    name: paasta-pinpoint-cluster
    version: latest

  properties:
    master:
      host_name: "<redacted>"
      host_ip: "<redacted>"
      port: "<redacted>"
      http_port: "<redacted>"
      replication: "<redacted>"
      tcp_listen_port: "<redacted>"
      udp_stat_listen_port: "<redacted>"
      udp_span_listen_port: "<redacted>"
    broker:
      collector_ips:
      - "<redacted>"
      - "<redacted>"
      collector_tcp_port: "<redacted>"
      collector_stat_port: "<redacted>"
      collector_span_port: "<redacted>"
      dashboard_uri: "<redacted>"
    secondary:
      host_name: "<redacted>"
      host_ip: "<redacted>"
      http_port: "<redacted>"
    yarn:
      host_name: "<redacted>"
      host_ip: "<redacted>"
      resource_tracker_port: "<redacted>"
      scheduler_port: "<redacted>"
      resourcemanager_port: "<redacted>"
    slave:
      host_names:
      - "<redacted>"
      - "<redacted>"
      host_ips:
      - "<redacted>"
      - "<redacted>"
    collector:
      host_names:
      - "<redacted>"
      - "<redacted>"
      host_ips:
      - "<redacted>"
      - "<redacted>"
    haproxy:
      host_name: "<redacted>"
      host_ip: "<redacted>"
      http_port: "<redacted>"
    webui:
      host_names:
      - "<redacted>"
      - "<redacted>"
      host_ips:
      - "<redacted>"
      - "<redacted>"
  Please review all changes carefully

  Deploying
  ---------
  Are you sure you want to deploy? (type 'yes' to continue): yes

  Director task 446
  Deprecation: Ignoring cloud config. Manifest contains 'networks' section.

    Started preparing deployment > Preparing deployment. Done (00:00:00)

    Started preparing package compilation > Finding packages to compile. Done (00:00:00)

    Started compiling packages
    Started compiling packages > haproxy/88e2cea38299d775fda4bac58eba167eefae6dc5
    Started compiling packages > java/788dd8b3824f7ed89a51f0d870d8a5b6376820a5
    Started compiling packages > ssh-keys/a92619994e7c69a7c52d30f4bc87c949ca2e5b77
    Started compiling packages > bosh-helpers/10d1d5cf2e9dfd6c8279aa99386c560ed32eb6fe. Done (00:01:24)
       Done compiling packages > ssh-keys/a92619994e7c69a7c52d30f4bc87c949ca2e5b77 (00:01:29)
       Done compiling packages > haproxy/88e2cea38299d775fda4bac58eba167eefae6dc5 (00:02:16)
       Done compiling packages > java/788dd8b3824f7ed89a51f0d870d8a5b6376820a5 (00:02:17)
    Started compiling packages > broker/9c8209bff0c69156bcb15e1ed7f1c7f3b73df2cd
    Started compiling packages > tomcat/462a5a0b802cb559e23fe8e59b804c594c641428
    Started compiling packages > hadoop/1d3ea90f6e3173e51ce42eacccd380c8c821f3a4
    Started compiling packages > hbase/5d5427c7c567de557a7f899f2c83f347723007c3
       Done compiling packages > tomcat/462a5a0b802cb559e23fe8e59b804c594c641428 (00:02:00)
    Started compiling packages > webui/655536239b03532d40b99159865520d8ee9e870d
    Started compiling packages > collector/e0b59e5385885af84c3cb4363821d5d6d1be0eae
       Done compiling packages > broker/9c8209bff0c69156bcb15e1ed7f1c7f3b73df2cd (00:02:07)
       Done compiling packages > hadoop/1d3ea90f6e3173e51ce42eacccd380c8c821f3a4 (00:02:16)
       Done compiling packages > hbase/5d5427c7c567de557a7f899f2c83f347723007c3 (00:02:36)
       Done compiling packages > webui/655536239b03532d40b99159865520d8ee9e870d (00:01:33)
       Done compiling packages > collector/e0b59e5385885af84c3cb4363821d5d6d1be0eae (00:01:41)
       Done compiling packages (00:05:58)

    Started creating missing vms
    Started creating missing vms > h_slave/c6ed5066-be08-4dfa-b9cf-40c8c9c89778 (1)
    Started creating missing vms > h_slave/ae8f48e2-5273-4b23-b63f-b2a3ab77eb4c (0)
    Started creating missing vms > h_secondary/6b29e5c9-0aac-48fb-96d1-c029415451f6 (0)
    Started creating missing vms > h_master/50b79e72-c97f-4402-98ed-c08d97723291 (0)
    Started creating missing vms > collector/2af508e8-1751-41be-905a-43f5884c4402 (0)
    Started creating missing vms > collector/e741680d-b612-4ccf-8495-7855006dfc80 (1)
    Started creating missing vms > haproxy_webui/2a977c16-331b-430c-9690-314e31eea9c8 (0)
    Started creating missing vms > webui/b23ab30f-c0f9-473a-9609-30fc5f2408b2 (0)
    Started creating missing vms > webui/0b3a252a-9657-4efa-98ff-1eeaac368bc9 (1)
    Started creating missing vms > pinpoint_broker/82756b8b-f1a1-4bbe-9408-6875bb2a9976 (0)
       Done creating missing vms > haproxy_webui/2a977c16-331b-430c-9690-314e31eea9c8 (0) (00:01:11)
       Done creating missing vms > h_master/50b79e72-c97f-4402-98ed-c08d97723291 (0) (00:01:22)
       Done creating missing vms > webui/b23ab30f-c0f9-473a-9609-30fc5f2408b2 (0) (00:01:25)
       Done creating missing vms > collector/2af508e8-1751-41be-905a-43f5884c4402 (0) (00:01:25)
       Done creating missing vms > webui/0b3a252a-9657-4efa-98ff-1eeaac368bc9 (1) (00:01:26)
       Done creating missing vms > pinpoint_broker/82756b8b-f1a1-4bbe-9408-6875bb2a9976 (0) (00:01:27)
       Done creating missing vms > h_slave/ae8f48e2-5273-4b23-b63f-b2a3ab77eb4c (0) (00:01:28)
       Done creating missing vms > h_secondary/6b29e5c9-0aac-48fb-96d1-c029415451f6 (0) (00:01:30)
       Done creating missing vms > h_slave/c6ed5066-be08-4dfa-b9cf-40c8c9c89778 (1) (00:01:30)
       Done creating missing vms > collector/e741680d-b612-4ccf-8495-7855006dfc80 (1) (00:01:40)
       Done creating missing vms (00:01:40)

    Started updating instance h_slave
    Started updating instance h_slave > h_slave/ae8f48e2-5273-4b23-b63f-b2a3ab77eb4c (0) (canary). Done (00:02:51)
    Started updating instance h_slave > h_slave/c6ed5066-be08-4dfa-b9cf-40c8c9c89778 (1). Done (00:02:52)
       Done updating instance h_slave (00:05:43)
    Started updating instance h_secondary > h_secondary/6b29e5c9-0aac-48fb-96d1-c029415451f6 (0) (canary). Done (00:02:43)
    Started updating instance h_master > h_master/50b79e72-c97f-4402-98ed-c08d97723291 (0) (canary). Done (00:02:48)
    Started updating instance collector
    Started updating instance collector > collector/2af508e8-1751-41be-905a-43f5884c4402 (0) (canary). Done (00:02:34)
    Started updating instance collector > collector/e741680d-b612-4ccf-8495-7855006dfc80 (1)

  . Done (00:02:32)
       Done updating instance collector (00:05:06)
    Started updating instance haproxy_webui > haproxy_webui/2a977c16-331b-430c-9690-314e31eea9c8 (0) (canary). Done (00:02:29)
    Started updating instance webui
    Started updating instance webui > webui/b23ab30f-c0f9-473a-9609-30fc5f2408b2 (0) (canary). Done (00:02:37)
    Started updating instance webui > webui/0b3a252a-9657-4efa-98ff-1eeaac368bc9 (1). Done (00:02:37)
       Done updating instance webui (00:05:14)
    Started updating instance pinpoint_broker > pinpoint_broker/82756b8b-f1a1-4bbe-9408-6875bb2a9976 (0) (canary). Done (00:02:32)

  Task 446 done

  Started   2016-12-29 07:54:37 UTC
  Finished  2016-12-29 08:28:51 UTC
  Duration  00:34:14

  Deployed 'openpaas-paasta-pinpoint' to 'bosh'
```

-   배포된 Pinpoint 서비스팩을 확인한다.

  ```
  Deployment 'openpaas-paasta-pinpoint'

  Director task 503

  Task 503 done

  +----------------------------------------------------------+---------+-----+-----------------+---------------+
  | VM                                                       | State   | AZ  | VM Type         | IPs           |
  +----------------------------------------------------------+---------+-----+-----------------+---------------+
  | collector/0 (2af508e8-1751-41be-905a-43f5884c4402)       | running | n/a | pinpoint_medium | 10.30.70.40   |
  | collector/1 (e741680d-b612-4ccf-8495-7855006dfc80)       | running | n/a | pinpoint_medium | 10.30.70.41   |
  | h_master/0 (50b79e72-c97f-4402-98ed-c08d97723291)        | running | n/a | pinpoint_medium | 10.30.70.75   |
  | h_secondary/0 (6b29e5c9-0aac-48fb-96d1-c029415451f6)     | running | n/a | pinpoint_small  | 10.30.70.76   |
  | h_slave/0 (ae8f48e2-5273-4b23-b63f-b2a3ab77eb4c)         | running | n/a | pinpoint_small  | 10.30.70.73   |
  | h_slave/1 (c6ed5066-be08-4dfa-b9cf-40c8c9c89778)         | running | n/a | pinpoint_small  | 10.30.70.74   |
  | haproxy_webui/0 (2a977c16-331b-430c-9690-314e31eea9c8)   | running | n/a | pinpoint_small  | 10.30.70.78   |
  |                                                          |         |     |                 | 115.68.46.182 |
  | pinpoint_broker/0 (82756b8b-f1a1-4bbe-9408-6875bb2a9976) | running | n/a | pinpoint_small  | 10.30.70.82   |
  | webui/0 (b23ab30f-c0f9-473a-9609-30fc5f2408b2)           | running | n/a | pinpoint_small  | 10.30.70.79   |
  | webui/1 (0b3a252a-9657-4efa-98ff-1eeaac368bc9)           | running | n/a | pinpoint_small  | 10.30.70.80   |
  +----------------------------------------------------------+---------+-----+-----------------+---------------+

  ```

<div id ='HBase-기본-데이터-실행'></div>
### 2.4. HBase 기본 데이터 실행

> Pinpoint 서비스팩 배포가 완료 되었으면 HBase 14개의 기본 Table이
> 생성되어야 Application에서 서비스 팩을 정상 사용할 수 있다.

  ```
  $ bosh run errand h_master_register
  Acting as user 'admin' on deployment 'openpaas-paasta-pinpoint' on 'my-bosh'

  Director task 1875
    Started preparing deployment > Preparing deployment. Done (00:00:00)

    Started preparing package compilation > Finding packages to compile. Done (00:00:00)

    Started creating missing vms > h_master_register/0 (3cccc024-c528-44ea-88fc-d6fea301ded5). Done (00:00:59)

    Started updating job h_master_register > h_master_register/0 (3cccc024-c528-44ea-88fc-d6fea301ded5) (canary)^C
  Do you want to cancel task 1875? [yN] (^C again to detach): n

  . Done (00:02:13)

    Started running errand > h_master_register/0. Done (00:00:34)

    Started fetching logs for h_master_register/3cccc024-c528-44ea-88fc-d6fea301ded5 (0) > Finding and packing log files. Done (00:00:01)

    Started deleting errand instances h_master_register > h_master_register/0 (3cccc024-c528-44ea-88fc-d6fea301ded5). Done (00:00:09)

  Task 1875 done

  Started   2016-12-14 06:41:06 UTC
  Finished  2016-12-14 06:45:02 UTC
  Duration  00:03:56

  [stdout]
  0 row(s) in 1.6690 seconds

  0 row(s) in 2.2490 seconds

  0 row(s) in 1.2360 seconds

  0 row(s) in 1.2220 seconds

  0 row(s) in 1.2240 seconds

  0 row(s) in 2.2290 seconds

  0 row(s) in 1.2290 seconds

  0 row(s) in 2.2430 seconds

  0 row(s) in 4.2790 seconds

  0 row(s) in 2.2410 seconds

  0 row(s) in 2.2270 seconds

  0 row(s) in 2.2390 seconds

  0 row(s) in 1.2260 seconds

  0 row(s) in 1.2300 seconds

  TABLE
  AgentEvent
  AgentInfo
  AgentLifeCycle
  AgentStat
  ApiMetaData
  ApplicationIndex
  ApplicationMapStatisticsCallee_Ver2
  ApplicationMapStatisticsCaller_Ver2
  ApplicationMapStatisticsSelf_Ver2
  ApplicationTraceIndex
  HostApplicationMap_Ver2
  SqlMetaData_Ver2
  StringMetaData
  Traces
  14 row(s) in 0.0360 seconds

  [stderr]
  Warning: Permanently added 'h-master,10.244.2.21' (ECDSA) to the list of known hosts.
  Unauthorized use is strictly prohibited. All access and activity
  is subject to logging and monitoring.
  2016-12-14 06:44:22,899 WARN ㅊㄹ [main] util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable

  Errand 'h_master_register' completed successfully (exit code 0)
  ```
<div id='pinpoint-서비스-브로커-등록'></div>
### 2.5. Pinpoint 서비스 브로커 등록

Pinpoint 서비스팩 배포가 완료 되었으면 Application에서 서비스 팩을
사용하기 위해서 먼저 Pinpoint 서비스 브로커를 등록해 주어야 한다.

서비스 브로커 등록시 개방형 클라우드 플랫폼에서 서비스브로커를 등록 할
수 있는 사용자로 로그인이 되어 있어야 한다.

-   서비스 브로커 목록을 확인한다.

  ```
  $cf service-brokers
  Getting service brokers as admin...

  name   url
  No service brokers found
  ```

-   Pinpoint 서비스 브로커를 등록한다.

  ```sh
  $cf create-service-broker {서비스팩 이름}{서비스팩 사용자ID}{서비스팩 사용자비밀번호} http://{서비스팩 URL(IP)}

  ```

  -   서비스팩 이름 : 서비스 팩 관리를 위해 개방형 클라우드 플랫폼에서 보여지는 명칭이다. 서비스 Marketplace에서는 각각의 API 서비스 명이 보여지니 여기서 명칭은 서비스팩 리스트의 명칭이다.
  
  -   서비스팩 사용자ID / 비밀번호 : 서비스팩에 접근할 수 있는 사용자 ID입니다. 서비스팩도 하나의 API 서버이기 때문에 아무나 접근을 허용할 수 없어 접근이 가능한 ID/비밀번호를 입력한다.
  
  -   서비스팩 URL : 서비스팩이 제공하는 API를 사용할 수 있는 URL을 입력한다.
  
  ```sh
  $ cf create-service-broker pinpoint-service-broker admin cloudfoundry http:// 10.30.70.82:8080
  Creating service broker pinpoint-service-broker as admin...
  OK
  ```

-   등록된 Pinpoint 서비스 브로커를 확인한다.

  ```
  $ cf service-brokers
  
  Getting service brokers as admin...
  name url
  pinpoint-service-broker http:// 10.30.70.82:8080
  ```

-   접근 가능한 서비스 목록을 확인한다.

  ```sh
  $ cf service-access
  
  Getting service access as admin...
  broker: Pinpoint-service-broker
  service plan access orgs
  Pinpoint Pinpoint\_standard none
  서비스 브로커 생성시 디폴트로 접근을 허용하지 않는다.
  ```

-   특정 조직에 해당 서비스 접근 허용을 할당하고 접근 서비스 목록을 다시
    확인한다. (전체 조직)

  ```sh
  $ cf enable-service-access Pinpoint
  Enabling access to all plans of service Pinpoint for all orgs as admin...
  OK
  ```

서비스 접근 허용을 확인한다.

  ```sh
  $ cf service-access

  Getting service access as admin...
  broker: Pinpoint-service-broker
  service plan access orgs
  Pinpoint Pinpoint\_standard all
  ```

<div id='sample-web-app-연동-pinpoint-연동'></div>
#   3. Sample Web App 연동 Pinpoint 연동

본 Sample Web App은 개발형 클라우드 플랫폼에 배포되며 Pinpoint의
서비스를 Provision과 Bind를 한 상태에서 사용이 가능하다.

<div id ='sample-web-app-구조'></div>
### 3.1. Sample App 구조

Sample Web App은 개방형 클라우드 플랫폼에 App으로 배포가 된다. 배포된
App에 Pinpoint 서비스 Bind 를 통하여 초기 데이터를 생성하게 된다. 바인드
완료 후 연결 url을 통하여 브라우로 해당 App에 대한 Pinpoint 서비스
모니터링을 할 수 있다.

-   Spring-music App을 이용하여 Pinpoint 모니터링을 테스트 하였다.
-   앱을 다운로드후 –b 옵션을 주어 buildpack을 지정하여 push 해 놓는다.

  ```sh
  $ cf push -b java_buildpack_pinpoint --no-start
  Using manifest file /home/ubuntu/workspace/bd_test/spring-music/manifest.yml

  Creating app spring-music-pinpoint in org org / space space as admin...
  OK

  Creating route spring-music-pinpoint.monitoring.open-paas.com...
  OK

  Binding spring-music-pinpoint.monitoring.open-paas.com to spring-music-pinpoint...
  OK

  Uploading spring-music-pinpoint...
  Uploading app files from: /tmp/unzipped-app175965484
  Uploading 21.2M, 126 files
  Done uploading               
  OK

  $ cf apps
  Getting apps in org org / space space as admin...
  OK

  name                    requested state   instances   memory   disk   urls
  php-demo                started           1/1         256M     1G     php-demo.monitoring.open-paas.com
  spring-music            stopped           0/1         512M     1G     spring-music.monitoring.open-paas.com
  spring-music-pinpoint   stopped           0/1         512M     1G     spring-music-pinpoint.monitoring.open-paas.com
  ```
<div id='개방형-클라우드-플랫폼에서-서비스-신청'></div>
### 3.2. 개방형 클라우드 플랫폼에서 서비스 신청

Sample Web App에서 Pinpoint 서비스를 사용하기 위해서는 서비스
신청(Provision)을 해야 한다.

\*참고: 서비스 신청시 개방형 클라우드 플랫폼에서 서비스를신청 할 수 있는
사용자로 로그인이 되어 있어야 한다.

-   먼저 개방형 클라우드 플랫폼 Marketplace에서 서비스가 있는지 확인을
    한다.

  ```sh
  $ cf marketplace
  Getting services from marketplace in org org / space space as admin...
  OK

  service    plans               description
  Pinpoint   Pinpoint_standard   A simple pinpoint implementation 
  ```

-   Marketplace에서 원하는 서비스가 있으면 서비스 신청(Provision)을 하여
    서비스 인스턴스를 생성한다.

  
  \$cf create-service {서비스명} {서비스플랜} {내서비스명}
  
  -   서비스명 : p-Pinpoint로 Marketplace에서 보여지는 서비스 명칭이다.
  
  -   서비스플랜 : 서비스에 대한 정책으로 plans에 있는 정보 중 하나를 선택한다. Pinpoint 서비스는 10 connection, 100 connection 를 지원한다.
  
  -   내서비스명 : 내 서비스에서 보여지는 명칭이다. 이 명칭을 기준으로 환경설정정보를 가져온다.
  
  ```sh  
  $ cf create-service Pinpoint Pinpoint_standard PS1
  Creating service instance PS1 in org org / space space as admin...
  OK 
  ```
-   생성된 Pinpoint 서비스 인스턴스를 확인한다.

  ```sh
  $ cf services
  Getting services in org org / space space as admin...
  OK

  name             service         plan                bound apps               last operation
  app_log_drain    user-provided
  PS1              Pinpoint        Pinpoint_standard                            create succeeded
  syslog_service   user-provided                       php-demo, spring-music 
  ```

<div id='#sample-web-app에-서비스-바인드-신청-및-app-확인'></div>
### 3.3. Sample App에 서비스 바인드 신청 및 App 확인
-------------------------------------------------

서비스 신청이 완료되었으면 Sample Web App 에서는 생성된 서비스
인스턴스를 Bind 하여 App에서 Pinpoint 서비스를 이용한다.

\*참고: 서비스 Bind 신청시 개방형 클라우드 플랫폼에서 서비스 Bind신청 할
수 있는 사용자로 로그인이 되어 있어야 한다.

-   Sample Web App에서 생성한 서비스 인스턴스 바인드 신청을 한다.

  서비스 인스턴스 확인
  ```
  $ cf s
  Getting services in org org / space space as admin...
  OK

  name             service         plan                bound apps               last operation
  app_log_drain    user-provided
  PS1              Pinpoint        Pinpoint_standard                            create succeeded
  syslog_service   user-provided                       spring-music, php-demo
  ubuntu@crossent-monitoring:~/workspace/bd_test$ cf bind-service spring-music-pinpoint PS1$cf bind-service spring-music-pinpoint PS1

  $ cf bind-service spring-music-pinpoint PS1 -c '{"application_name":"spring-music"}'
  Binding service PS1 to app spring-music-pinpoint in org org / space space as admin...
  OK
  TIP: Use 'cf restage spring-music-pinpoint' to ensure your env variable changes take effect
  ```
  
  cf cli 리눅스 버전 : 
  ```sh
    cf bind-service <application이름> PS1 -c ‘{"application_name\":"<application이름>"}
  ```  
  
  cf cli window 버전 : 
  cf bind-service <application이름> PS1 -c "{\"application_name\":\"<application이름>\"}"
  
  을 사용한다.

-   바인드가 적용되기 위해서 App을 restage한다.

  ```sh
  $ cf restage spring-music-pinpoint
  Restaging app spring-music-pinpoint in org org / space space as admin...
  Downloading binary_buildpack...
  Downloading go_buildpack...
  Downloading staticfile_buildpack...
  Downloading java_buildpack...
  Downloading ruby_buildpack...
  Downloading nodejs_buildpack...
  Downloading python_buildpack...
  Downloading php_buildpack...
  Downloaded python_buildpack
  Downloaded binary_buildpack
  Downloaded go_buildpack
  Downloaded java_buildpack
  Downloaded ruby_buildpack
  Downloaded nodejs_buildpack
  Downloaded staticfile_buildpack
  Downloaded php_buildpack
  Creating container
  Successfully created container
  Downloading app package...
  Downloaded app package (24.5M)
  Downloading build artifacts cache...
  Downloaded build artifacts cache (54.1M)
  Staging...
  -----> Java Buildpack Version: v3.7.1 | https://github.com/cloudfoundry/java-buildpack.git#78c3d0a
  -----> Downloading Open Jdk JRE 1.8.0_91-unlimited-crypto from https://java-buildpack.cloudfoundry.org/openjdk/trusty/x86_64/openjdk-1.8.0_91-unlimited-crypto.tar.gz (found in cache)
         Expanding Open Jdk JRE to .java-buildpack/open_jdk_jre (1.6s)
  -----> Downloading Open JDK Like Memory Calculator 2.0.2_RELEASE from https://java-buildpack.cloudfoundry.org/memory-calculator/trusty/x86_64/memory-calculator-2.0.2_RELEASE.tar.gz (found in cache)
         Memory Settings: -XX:MaxMetaspaceSize=64M -Xss995K -Xmx382293K -Xms382293K -XX:MetaspaceSize=64M
  -----> Downloading Spring Auto Reconfiguration 1.10.0_RELEASE from https://java-buildpack.cloudfoundry.org/auto-reconfiguration/auto-reconfiguration-1.10.0_RELEASE.jar (found in cache)
  -----> Downloading Tomcat Instance 8.0.39 from https://java-buildpack.cloudfoundry.org/tomcat/tomcat-8.0.39.tar.gz (found in cache)
         Expanding Tomcat Instance to .java-buildpack/tomcat (0.1s)
  -----> Downloading Tomcat Lifecycle Support 2.5.0_RELEASE from https://java-buildpack.cloudfoundry.org/tomcat-lifecycle-support/tomcat-lifecycle-support-2.5.0_RELEASE.jar (found in cache)
  -----> Downloading Tomcat Logging Support 2.5.0_RELEASE from https://java-buildpack.cloudfoundry.org/tomcat-logging-support/tomcat-logging-support-2.5.0_RELEASE.jar (found in cache)
  -----> Downloading Tomcat Access Logging Support 2.5.0_RELEASE from https://java-buildpack.cloudfoundry.org/tomcat-access-logging-support/tomcat-access-logging-support-2.5.0_RELEASE.jar (found in cache)
  Exit status 0
  Staging complete
  Uploading droplet, build artifacts cache...
  Uploading droplet...
  Uploading build artifacts cache...
  Uploaded build artifacts cache (54.1M)
  Uploaded droplet (77.3M)
  Uploading complete

  0 of 1 instances running, 1 starting
  0 of 1 instances running, 1 starting
  0 of 1 instances running, 1 starting
  1 of 1 instances running

  App started


  OK
  ```
-   App이 정상적으로 Pinpoint 서비스를 사용하는지 확인한다.

  ![pinpoint_image_03]
  
  -   환경변수 확인
  
  ```sh
  $ cf env spring-music-pinpoint
  Getting env variables for app spring-music-pinpoint in org org / space space as admin...
  OK

  System-Provided:
  {
   "VCAP_SERVICES": {
    "Pinpoint": [
     {
      "credentials": {
       "application_name": "spring-music",
       "collector_host": "10.244.2.30",
       "collector_span_port": 29996,
       "collector_stat_port": 29995,
       "collector_tcp_port": 29994
      },
      "label": "Pinpoint",
      "name": "PS1",
      "plan": "Pinpoint_standard",
      "provider": null,
      "syslog_drain_url": null,
      "tags": [
       "pinpoint"
      ],
      "volume_mounts": []
     }
    ]
   }
  }

  {
   "VCAP_APPLICATION": {
    "application_id": "b010e6e9-5431-4198-81f8-7d6ba9c14f40",
    "application_name": "spring-music-pinpoint",
    "application_uris": [
     "spring-music-pinpoint.monitoring.open-paas.com"
    ],
    "application_version": "9a600116-97bd-45da-a33e-3b0d5592b1d0",
    "limits": {
     "disk": 1024,
     "fds": 16384,
     "mem": 512
    },
    "name": "spring-music-pinpoint",
    "space_id": "bc70b951-d870-49ca-b57d-5c7137060e5e",
    "space_name": "space",
    "uris": [
     "spring-music-pinpoint.monitoring.open-paas.com"
    ],
    "users": null,
    "version": "9a600116-97bd-45da-a33e-3b0d5592b1d0"
   }
  }

  No user-defined env variables have been set

  No running env variables have been set

  No staging env variables have been set

  $ curl http://115.68.151.187/#/main/spring-music-pinpoint@TOMCAT 로 화면 정상 구동 확인
  ```

[pinpoint_image_01]:/images/paasta-service/pinpoint/pinpoint-image1.png
[pinpoint_image_02]:/images/paasta-service/pinpoint/pinpoint-image1.png
[pinpoint_image_03]:/images/paasta-service/pinpoint/pinpoint-image1.png
