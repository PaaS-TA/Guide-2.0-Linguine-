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




