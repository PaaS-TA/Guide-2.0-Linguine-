## Table of Contents
1. [개요](#1)
	* [문서 개요](#2)
	 * [목적](#3)
	 * [범위](#4)
	 * [참고자료](#5)

2. [Abacus 배포](#6)
    * [미터링 범위](#7)
    * [배포 전제 조건](#8)
    * [Node.js 설치](#9)
     * [Node.js 설치 순서](#10)
    * [pouchdb, couchdb 설치](#11)
     * [couchdb 설치](#12)
     * [pouchdb 설치(옵션)](#13)
     * [설치 확인](#14)
     * [CouchDB 계정 생성](#32)
    * [CF에 abacus UAA 계정 등록](#15)
     * [UAA 클라이언트 설치](#16)
     * [CF 앱 사용량 수집을 위한 UAA 계정 드록](#17)
     * [Secured Abacus를 위한 UAA 계정 등록](#18)
    * [cf-abacus 배포](#19)
     * [Git을 통해 cf-abacus를 다운받는다.](#20)
     * [gradle build를 위한 dependency 추가](#21)
     * [Abacus와 연동할 DB 및 Secure 정보 설정](#22)
     * [Abacus 빌드](#23)
     * [Abacus 배포](#24)
     * [Abaus-cf-bridge 배포](#25)
* [PAASTA-USAGE-REPORTION 배포](#26)
    * [배포 전제 조건](#27)
    * [CF에 UAA 계정 등록](#28)
    * [paasta-usage-repoting 배포](#29)
    * [다운로드](#30)
    * [paasta-usage-reportion 배포](#31)
    * [배포 형상](#32)
    * [api 호출 예제](#33)

# <div id='1'/>1.  개요
## <div id='2'/>1.1.  문서 개요
### <div id='3'/>1.1.1.  목적

본 문서(node.js API 서비스 미터링 애플리케이션 개발 가이드)는 파스-타
플랫폼 프로젝트의 미터링 플러그인과 Node.js API 애플리케이션과 연동하여
API 서비스를 미터링하는 방법에 대해 기술 하였다.


### <div id='4'/>1.1.2.  범위

본 문서의 범위는 파스-타 플랫폼 프로젝트의 Node.js API 서비스
애플리케이션 개발과 CF-Abacus 연동에 대한 내용으로 한정되어 있다.


### <div id='5'/>1.1.3.  참고 자료 

-   [https://github.com/cloudfoundry-incubator/cf-abacus](https://github.com/cloudfoundry-incubator/cf-abacus)


# <div id='6'/>2.  Abacus 배포 

## <div id='7'/>2.1.  미터링 범위
- 파스-타 플랫폼에서 배포 또는 삭제 한 앱의 컨테이너 사용량을 집계
- 조직/영역/앱에 대해 당월 사용량 및 지정한 기간 동안의 월간 사용량에 대해 조회 가능

## <div id='8'/>2.2.  배포 전제 조건

-   이 가이드는 ubuntu14.04 및 로컬 설치를 기준으로 작성되어 있다. 
-   cf-CLI 가 로컬에 설치 되어 있어야 한다.
-   zip 패키지가 로컬에 설치 되어 있어야 한다.
-   CF가 설치 되어 있어야 한다.

    ※ 운영 환경의 CF에 abacus를 배포할 경우, abacus를 서비스 하기 위한 security-group을 설정해야 한다.

    -   **Abacus를 위한 security 설정 정보**
    <table style ="width : 700;">
      <tr>
    	<th>Component</th>
        <th>Protocol</th>
        <th>Port</th>
      </tr>
      <tr>
      	<td>abacus-pouchserver</td>
        <td rowspan="12">TCP</td>
        <td>5984</td>
      </tr>
      <tr>
      	<td>abacus-usage-collector</td>
        <td>9080</td>
      </tr>
      <tr>
      	<td>abacus-usage-reporting</td>
        <td>9088</td>
      </tr>
       <tr>
      	<td>abacus-usage-meter</td>
        <td>9100</td>
      </tr>
       <tr>
      	<td>abacus-usage-accumulator</td>
        <td>9200</td>
      </tr>
       <tr>
      	<td>abacus-usage-aggregator</td>
        <td>9300</td>
      </tr>
       <tr>
      	<td>abacus-cf-bridge</td>
        <td>9500</td>
      </tr>
       <tr>
      	<td>abacus-cf-renewer</td>
        <td>9501</td>
      </tr>
      <tr>
      	<td>abacus-provisioning-plugin</td>
        <td>9880</td>
      </tr>
       <tr>
      	<td>abacus-account-plugin</td>
        <td>9881</td>
      </tr>
      <tr>
      	<td>abacus-authserver-plugin</td>
        <td>9882</td>
      </tr>
       <tr>
      	<td>abacus-eureka-plugin</td>
        <td>9990</td>
      </tr>

    </table>


## <div id='9'/>2.3.  Node.js 설치

※ 설치할 abacus가 요구하는 Node.js 및 Npm 버전을 설치한다. Cf-abacus
사이트에서 해당 버전을 확인한다.
[https://github.com/cloudfoundry-incubator/cf-abacus](https://github.com/cloudfoundry-incubator/cf-abacus)

※ Node.js와 npm은 같이 설치된다

### <div id='10'/>2.3.1. Node.js 설치 순서

  	$ sudo apt-get install curl

  	## Node.js version 6.x를 설치할 경우
  	## Source Repository 등록
  	$ curl -sL https://deb.nodesource.com/setup_6.x | sudo -E bash -

  	## Node.js & Npm 설치
  	$ sudo apt-get install -y nodejs

## <div id='11'/>2.4.  pouchdb, couchdb 설치

※ abacus 데모를 실행하기 위해서는 pouchdb를 설치 해야 한다.

※ abacus 데이터를 영구 보존하기 위해 서는 couchdb를 설치해야 한다.

### <div id='12'/>2.4.1.couchdb 설치

  	## CouchDB 설치
  	$ sudo apt-get install couchdb

  	## CouchDB 설정
  	※ abacus와 couchdb가 모두 같은 로컬 환경인 경우 아래 설정은 생략한다.
  	$ vi /etc/couchdb/default.ini

  	; etc/couchdb/default.ini.tpl. Generated from default.ini.tpl.in by configure.

  	; Upgrading CouchDB will overwrite this file.
  	[vendor]
  	name = Ubuntu
  	version = 14.04

  	<<중략>>

  	[httpd]
  	port = 5984
  	bind_address = 127.0.0.1 ## Remote 서비스를 할 경우, 0.0.0.0 또는 cf 도메인 ip로 수정한다.
	authentication_handlers = {couch_httpd_oauth, oauth_authentication_handler}, {couch_httpd_auth, cookie_authentication_handler}, {couch_httpd_auth, default_authentication_handler}
  	default_handler = {couch_httpd_db, handle_request}
  	secure_rewrites = true
  	vhost_global_handlers = _utils, _uuids, _session, _oauth, _users
  	allow_jsonp = false

  	<<후략>>

  	$ sudo service couchdb restart

※ couchDB 설정에 관해서 다음 웹 사이트를 참조한다.

[http://docs.couchdb.org/en/master/config/index.html](http://docs.couchdb.org/en/master/config/index.html)


### <div id='13'/>2.4.2. pouchdb 설치(옵션)

  	## pouchdb 설치
  	$ sudo npm install -g pouchdb-server

  	## pouchdb에 서비스 포트 할당
  	$ pouchdb-server --port 5984

### <div id='14'/>2.4.3.  설치 확인 

  	$ curl localhost:5984 
  	> {"couchdb":"Welcome","version":"1.5.1",...}

### <div id='32'/>2.4.4.  CouchDB 계정 생성
[CouchDB 계정 생성](https://github.com/PaaS-TA/Guide-2.0-Linguine-/blob/master/Install-Guide/metering/Couchdb_Create_Admin.md)

## <div id='15'/>2.5.  CF에 abacus UAA 계정 등록

CF 설치한 abacus에서 CF의 앱 사용량 정보를 수집하기 위해서 CF 접근을
위한 계정 및 토큰을발급 받아야 한다.

또한 보안 모드로 abacus를 배포한 경우, abacus 컴포넌트간의 데이터 전송을
위한 계정을 등록해야 한다.

### <div id='16'/>2.5.1. UAA 클라이언트 설치

  	$ gem install cf-uaac

### <div id='17'/>2.5.2. CF 앱 사용량 수집을 위한 UAA 계정 등록

  	## CF target 설정
  	$ uaac target uaa.<CF 도메인> --skip-ssl-validation

  	예) $ uaac target uaa.bosh-lite.com --skip-ssl-validation

  	## uaa client 토큰 취득
  	$ uaac token client get <uaac user id> -s <uaac user password>

  	예) $ uaac token client get admin -s admin-secret

  	## cloud_controller.admi 계정 생성
  	$ uaac client add <CF_CLIENT_ID> --name <CF_CLIENT_ID> --authorized_grant_types client_credentials --authorities cloud_controller.admin --secret <CF_CLIENT_SECRET>

 	예) $ uaac client add abacus-cf-bridge --name abacus-cf-bridge --authorized_grant_types client_credentials --authorities cloud_controller.admin --secret secret

## <div id='18'/>2.5.3.  Secured Abacus를 위한 UAA 계정 등록 

-   Secured Abacus를 위한 권한 목록

    <table>
      <tr>
    	<th>내용</th>
        <th>권한 SCOPE</th>
        <th>예시</th>
      </tr>
      <tr>
      	<td rowspan="2">미터링 자원 사용량 정보에 대한 abacus 접근 권한</td>
        <td>abacus.usage.{Resource_id}.write</td>
        <td>abacus.usage.linux-container.write</td>
      </tr>
      <tr>
        <td>abacus.usage.{Resource_id}.read</td>
        <td>abacus.usage.linux-container.read</td>
      </tr>
      <tr>
      	<td  rowspan="2">사용량 정보에 대한 abacus접근 권한</td>
        <td>abacus.usage.write</td>
        <td>-</td>
      </tr>
      <tr>
        <td>abacus.usage.read</td>
        <td>-</td>
      </tr>
     <tr>
      	<td>Abacus 모니터링 권한</td>
        <td>abacus.system.read</td>
        <td>-</td>
      </tr>
    </table>

  		##미터링 자원 사용량 정보에 대한 abacus 접근 권한을 UAA 에 등록

  		$ uaac client add <CLIENT_ID> --name <CLIENT_ID> --authorized_grant_types client_credentials --authorities abacus.usage.<Resource_id>.write,abacus.usage.<Resource_id>.read --scope abacus.usage.<Resource_d>*.write,abacus.usage.<Resource_id>.read --secret *<CLIENT_SECRET>

  		예)
  		$ uaac client add abacus-linux-container --name abacus-linux-container --authorized_grant_types client_credentials --authorities abacus.usage.linux-container.write,abacus.usage.linux-container.read --scope abacus.usage.linux-container.write,abacus.usage.linux-container.read --secret secret

  		##사용량 정보에 대한 abacus 접근 권한을 UAA에 등록
  		$ uaac client add <CLIENT_ID> --name <CLIENT_ID> --authorized_grant_types client_credentials --authorities abacus.usage.write,abacus.usage.read --scope abacus.usage.write,abacus.usage.read --secret <CLIENT_SECRET>

  		예)
  		$ uaac client add abacus --name abacus --authorized_grant_types client_credentials --authorities abacus.usage.write,abacus.usage.read --scope abacus.usage.write,abacus.usage.read --secret secret

  		##Abacus 모니터링 권한을 UAA에 등록
  		$ uaac client add <CLIENT_ID> --name <CLIENT_ID> --authorized_grant_types client_credentials –authorities abacus.system.read --scope abacus.system.read --secret <CLIENT_SECRET>

  		예)
  		$ uaac client add abacus-system --name abacus-system --authorized_grant_types client_credentials --authorities abacus.system.read --scope abacus.system.read --secret secret

※ 하나의 \<CLIENT_ID\>에 모든 권한을 부여할 수 있다.

※ Secured Abacus에 대해서는 다음 웹 사이트를 참조한다.
[https://github.com/cloudfoundry-incubator/cf-abacus/blob/master/doc/security.md](https://github.com/cloudfoundry-incubator/cf-abacus/blob/master/doc/security.md)


## <div id='19'/>2.6.  Abacus 배포를 위한 조직 및 영역 설정

  	<< Bosh Lite의 경우>>

  	##라우트 확인

  	$ route

  	$ route
  	Kernel IP routing table
  	Destination   Gateway      Genmask         Flags Metric Ref Use Iface
  	default       10.8.0.5     128.0.0.0       UG    0      0   0   tun0
  	default       192.168.4.1  0.0.0.0         UG    0      0   0   eth1
  	10.8.0.1      10.8.0.5     255.255.255.255 UGH   0      0   0   tun0
  	1	0.8.0.5   *            255.255.255.255 UH    0      0   0   tun0
  	10.244.0.0    192.168.50.4 255.255.0.0     UG    0      0   0   vboxnet0
  	61.74.60.194  192.168.4.1  255.255.255.255 UGH   0      0   0   eth1
  	128.0.0.0     10.8.0.5     128.0.0.0       UG    0      0   0   tun0
  	192.168.4.0   *            255.255.255.0   U     1      0   0   eth1
  	192.168.50.0  *            255.255.255.0   U     0      0   0   vboxnet0

  	## route 설정에 10.244.0.0가 없는 경우
  	$ <bosh-lite 설치 경로>/bin/add-route

  	## CF target 설정
  	$ cf api https://api.<CF 도메인> --skip-ssl-validation
  	예) $ cf api https://api.bosh-lite.com --skip-ssl-validation


  	## CF 로그인
  	$ cf login

  	##조직 생성 및 설정
  	$ cf create-org <조직>
  	$ cf target -o <조직>

  	##영역 생성 및 설정
  	$ cf create-space <영역>
  	$ cf target -o <조직> -s <영역>

## <div id='20'/>2.7.  cf-abacus 배포

-   **Abacus****기능 개요**

| 형상  |설명|
|--------------------------------|--------------------------------------------------------------------------|
|  abacus-pouchserver       |Abacus가 사용하는 in-browser database. 앱을 재시작하면 데이터가 사라지므로 운영 환경에서는 별도의 CouchDB 또는 MongoDB를 구성해야 한다.   |
|  abacus-usage-collector      | CF 앱 사용량 수집기  |
|  abacus-usage-reporting       |Abacus가 수집/집계한 미터링 정보에 사용자의 요청에 맞게 보고한다.   |
|   abacus-usage-meter      |  수집한 CF 앱 사용량의 집계를 위해 미터링 정보를 가공한다.  | 
|  abacus-usage-accumulator       | 수집한 CF 앱 사용량 정보를 계정/조직/영역/앱 별로 누적한다.   | 
|  abacus-usage-aggregator       | 누적한 CF 앱 사용량 정보를 계정/조직/영역/앱 별로 집계한다.  |
|  abacus-cf-bridge       |CF와 연동하여 앱의 Container 사용량 정보를 수집한다.   |  
| abacus-provisioning-plugin        | Abacus의 각 기능에서 수집/누적/집계에 필요한 메타 정보를 제공한다.  |
|  abacus-account-plugin       | 앱의 계정 정보 조회 및 유효성 체크를 수행한다.  |
|  abacus-authserver-plugin       | Secured Abacus 운영을 위한 인증 서비스를 제공한다.<br>운영 환경에서는 CF UAA로 대체 한다.|
|   abacus-eureka-plugin      | Netflix의 Eureka 시스템과 연동하여 Abacus 앱의 분산 처리 서비스를 제공한다.  |


### <div id='21'/>2.7.1. Git을 통해 cf-abacus를 다운 받는다.

  	$ cd <abacus를 설치할 경로>
  	$ git clone https://github.com/cloudfoundry-incubator/cf-abacus

-   Abacus를 빌드하기 위해서는 Node.js 및 Npm을 사전에 설치해야 한다.


### <div id='22'/>2.7.2. Abacus와 연동할 DB 및 Secure 정보 설정

-	DB연동 및 Secure 정보를 설정하기 위해서는 다음 경로에 있는 manifest.yml 파일을 수정한다.

| 앱  |경로|
|-----------------------|------------------------------------------------------|
|  abacus-pouchserver       | <abacus 경로>/cf-abacus/lib/utils/pouchserver  |
|  abacus-usage-collector       | <abacus 경로>/cf-abacus/lib/metering/collector  |
|  abacus-usage-reporting      | <abacus 경로>/cf-abacus/lib/aggregation/reporting  |
|  abacus-usage-meter       | <abacus 경로>/cf-abacus/lib/metering/meter  | 
|  abacus-usage-accumulator       | <abacus 경로>/cf-abacus/lib/aggregation/accumulator  | 
|  abacus-usage-aggregator       | <abacus 경로>/cf-abacus/lib/aggregation/aggregator  |
|  abacus-provisioning-plugin       | <abacus 경로>/cf-abacus/lib/plugins/provisioning  |  
|  abacus-account-plugin       | <abacus 경로>/cf-abacus/lib/plugins/account  |
|  abacus-authserver-plugin       |<abacus 경로>/cf-abacus/lib/plugins/authserver   |
|  abacus-eureka-plugin       | <abacus 경로>/cf-abacus/lib/plugins/eureka  |

-   manifest.yml 설정의 예

```yaml
  
applications:
- name: abacus-usage-collector ## 앱 명
  host: abacus-usage-collector   ## 호스트 명
  path: .cfpack/app.zip          ## Cf에 push할 앱을 빌드한 파일 경로 및 파일
  instances: 1 ## 앱에 할당할 인스턴스 수
  memory: 512M     	           ## 앱에 할당할 메모리 사이즈
  disk_quota: 512M               ## 앱에 할당할 디스크 사이즈
  env:            		       ## 앱 실행 환경 설정
    CONF: default
    DEBUG: e-abacus-* 			   ## 앱의 출력 로그 레벨
    METER: abacus-usage-meter
    PROVISIONING: abacus-provisioning-plugin
    #DB: abacus-pouchserver 	    ## 로컬 pouchDB를 사용할 경우 설정한 예시
    DB: http://<user_id>:<password>@<coudb_ip>:<port> ## 외부의 CouchDB를 사용할 경우 설정한 예시
    #DB: mongodb://9a6e635e-41aa-4522-97b3-ade805ce5b89:4fa40ffd-7a78-445d-937c-82999844fb8e@192.168.40.153:27017/728b0614-5357-480e-b238-b618fcc8b957 ## MongoDB 서비스와 바인드하여 MongoDB를 사용할 경우 설정한 예시
    DBCLIENT: abacus-couchclient ## CouchDB, PouchDB를 사용할 경우 설정
    #DBCLIENT: abacus-mongoclient ## MongoDB를 사용할 경우 설정
    ACCOUNT: abacus-account-plugin
    EUREKA: abacus-eureka-plugin
    NODE_MODULES_CACHE: false
    SECURED: true ## Abacus를 Secured 모드로 운영할 경우: true
    AUTH_SERVER: https://api.bosh-lite.com:443 ## Auth 서비스 Endpoint
    CLIENT_ID: abacus ## 사용량 정보 접근 권한 ID
    CLIENT_SECRET: secret ## 사용량 정보 접근 권한 Secret
    JWTKEY: |+ ### cf 배포 manifest의 properties.uaa.jwt.verification_key를 설정
      -----BEGIN PUBLIC KEY-----
      MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQDHFr+KICms+tuT1OXJwhCUmR2d
      …
      -----END PUBLIC KEY-----
    JWTALGO: RS256 ## JWTKEY 알고리즘
    THROTTLE: 2
```
  >
	
	### abacus-authserver를 Auth 서비스로 사용할 경우 설정 예시

  	AUTH_SERVER: abacus-authserver-plugin ## Auth 서비스 Endpoint
  	CLIENT_ID: abacus ## 사용량 정보 접근 권한 ID
  	CLIENT_SECRET: secret ## 사용량 정보 접근 권한 Secret
  	JWTKEY: encode
  	JWTALGO: HS256 ## JWTKEY 알고리즘


### <div id='23'/>2.7.3. Abacus 빌드

  	$ cd <abacus 경로>
  	$ npm run build


### <div id='24'/>2.7.4. Abacus 배포

  	$ cd <abacus 경로>

  	## abacus 설치를 위한 CF 타겟 지정 5. 에서 생성한 조직 과 영역을 지정 한다 .	
  	이미 지정한 경우 생략한다.
  	$ bin/cfsetup

  	$ bin/cfsetup
  	Enter your API URL [https://api.bosh-lite.com]: https://api.bosh-lite.com ## CF API EndPoint
  	Enter your user name [admin]: admin ## <계정 ID>
  	Enter your organization [abacus]: abacus ## <조직>
  	Enter your space [dev]: dev ## <영역>
  	API endpoint: https://api.bosh-lite.com
  	Password> ## <계정 비밀번호>
  	Authenticating...
  	OK

  	Targeted org abacus

  	Targeted space dev

  	API endpoint: https://api.bosh-lite.com (API version: 2.62.0)
  	User: admin
  	Org: abacus
  	Space: dev
  	Creating security group abacus as admin
  	OK

  	Assigning security group abacus to space dev in org abacus as admin...

  	OK

  	TIP: Changes will not apply to existing running applications until they are restarted.

  	##타겟으로 지정한 조직 / 영역에 대해 abacus앱을 배포한다.
  	※ 주의: CF의 Nodejs Build Pack 버전이 낮을 경우 cfpush는 실패한다. CF의 Nodejs build pack 버전이 낮을 경우, 반드시 Nodejs build pack을 업그레이드 한다.
  	$ npm run cfpush
  	## abacus 설치 형상 확인

  	$ cf a

  	$ cf a
  	Getting apps in org abacus / space dev as admin...
  	OK

  	name                        requested state instances memory   disk   urls
  	abacus-pouchserver          started         1/1       1G       512M   abacus-pouchserver.bosh-lite.com
  	abacus-authserver-plugin    started         1/1       512M     512M   abacus-authserver-plugin.bosh-lite.com
  	abacus-account-plugin       started         1/1       512M     512M   abacus-account-plugin.bosh-lite.com
  	abacus-usage-collector      started         1/1       512M     512M   abacus-usage-collector.bosh-lite.com
  	abacus-usage-aggregator     started         1/1       512M     512M   abacus-usage-aggregator.bosh-lite.com
  	abacus-provisioning-plugin  started         1/1       512M     512M   abacus-provisioning-plugin.bosh-lite.com
  	abacus-eureka-plugin        started         1/1       512M     512M   abacus-eureka-plugin.bosh-lite.com
  	abacus-usage-meter          started         1/1       512M     512M   abacus-usage-meter.bosh-lite.com
  	abacus-usage-accumulator    started         1/1       512M     512M   abacus-usage-accumulator.bosh-lite.com
  	abacus-usage-reporting      started         1/1       512M     512M   abacus-usage-reporting.bosh-lite.com

※ Build Pack 업데이트 참고 사이트

[https://docs.cloudfoundry.org/adminguide/buildpacks.html](https://docs.cloudfoundry.org/adminguide/buildpacks.html)

### <div id='25'/>2.7.5. abacus-cf-bridge 배포

[2.6.4](#23) 에서 배포한 abacus는 외부 사용량에 대한 수집 기능은 없다. 따라서
실제 CF에서 발생한 앱 사용량을 수집하기 위해서는 별도의 앱을 배포해야
한다. Abacus-cf-bridge는 CF에서 사용 중인 앱의 Container 사용량을
수집하기 위한 앱이다.


  	$ cd <abacus 경로>/lib/cf/bridge

  	## CF Bridge 빌드
  	$ npm install && npm run babel && npm run lint && npm test && npm run cfpack

  	## CF 연동을 위한 Secure 정보 설정
  	$ vi manifest.yml
 	※ *manifest.yml**내용 및 수정 사항에 대해서는 별도 기술*

  	## CF Bridge 배포
  	$ cf push –no-start

  	## abacus-cf-bridge 서비스 시작
  	$ cf start abacus-cf-bridge
-   Abacus-cf-bridge의 manifest.yml

```yaml

applications:
- name: abacus-cf-bridge
  host: abacus-cf-bridge
  path: .cfpack/app.zip
  instances: 1                                   ### 앱 인스턴스 수
  memory: 512M                                   ### 앱 할당 메모리
  disk_quota: 512M                               ### 앱 할당 디스크
  env:                                           ### 앱 실행 환경 정보
    CONF: default
    DEBUG: e-abacus-*
    COLLECTOR: abacus-usage-collector
    EUREKA: abacus-eureka-plugin
    API: https://api.<CF 도메인>                  ### CF api 도메인
    #DB: abacus-pouchserver                      ### Abacus의 Pouch서버를 DB로 사용할 경우
    DB: http://<user_id>:<password>@<coudb_ip>:<port>      ### 외부의 CouchDB를 DB로 사용할 경우 설정 예시
    #DB: mongodb://9a6e635e-41aa-4522-97b3-ade805ce5b89:4fa40ffd-7a78-445d-937c-82999844fb8e@192.168.40.153:27017/728b0614-5357-480e-b238-b618fcc8b957  ## Abacus를 CF MongoDB 서비스팩과 바인드하여 DB로 사용할 경우 설정 예시
    DBCLIENT: abacus-couchclient     ## CouchDB 또는 PouchDB를 사용할 경우
    #DBCLIENT: abacus-mongoclient    ## MongoDB를 사용할 경우
    CF_CLIENT_ID: abacus-cf-bridge   ## CF cloud_controller.admin Client ID
    CF_CLIENT_SECRET: secret         ## CF cloud_controller.admin Client Secret
    NODE_MODULES_CACHE: false
    SECURED: true                  ## Abacus를 Secured 모드로 운영할 경우: true
    AUTH_SERVER: https://api.bosh-lite.com   ## Auth 서비스 Endpoint
    CLIENT_ID: abacus-linux-container        ## 미터링 자원 접근 권한 ID
    CLIENT_SECRET: secret                    ## 미터링 자원 접근 권한 secret
    JWTKEY: |+     ### cf 배포 manifest의 properties.uaa.jwt.verification_key를 설정
      -----BEGIN PUBLIC KEY-----
      MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQDHFr+KICms+tuT1OXJwhCUmR2d
      …
      -----END PUBLIC KEY-----
    JWTALGO: RS256                           ## JWTKEY 알고리즘
    THROTTLE: 2

```

※	참고: cf-abacus 는 cf-abacus 가 설치 완료 된 이후 시점부터, cf 상의 app이 새로 push 되거나 cf stop 및 cf start 된 cf event 를 기반으로 데이터를 수집, 집계한다.


# <div id='26'/>3.  PAASTA-USAGE-REPORTING 배포

PAASTA-USAGE-REPORTING은 abacus 시스템과 연동하여 PAASTA에 앱의 사용량을
보고하는 서비스이다.

## <div id='27'/>3.1.  배포 전제 조건 

[[Abacus 배포 전제 조건](#7)]  참조.

## <div id='28'/>3.2.  CF에 UAA 계정 등록 

[[Abacus UAA 계정 등록](#14)]  참조.

## <div id='29'/>3.3.  paasta-usage-repoting 배포 

### <div id='30'/>3.3.1.다운로드 

[다운로드](https://paas-ta.kr/data/packages/2.0/PaaSTA-Metering.zip)

  	##다운로드 대상 파일
  	PaaS-TA-Usage-Reporting.tar

  	##대상 파일을<설치 경로>에 다운로드
  	$ cd <설치 경로>

  	##파일압축 해제
  	$ tar xvf PaaS-TA-Usage-Reporting.tar


### <div id='31'/>3.3.2. paasta-usage-reporting 배포

  	$ cd <설치 경로>/PaaS-TA-Usage-Reporting/usageReporting

  	## Abacus 연동을 위한 DB 및 Secure 정보 설정
  	$ vi manifest.yml
  	※ manifest.yml 내용 및 수정 사항에 대해서는 별도 기술

  	$ cf push

※ paasta-usage-reporting과 연동하기 위한 인터페이스는 다음 파일을
참조한다.

[PaaS-TA_Usage_Reporting_API_가이드](../../Use-Guide/PaaS-TA_Usage_Reporting_API_%EA%B0%80%EC%9D%B4%EB%93%9C.md)

※ paasta-usage-reporting manifest.yml

```yml
applications:
- name: paasta-usage-reporting
  memory: 1024M
  disk_quota: 512M
  instances: 1
  command: node app.js # 애플리케이션 실행 명령어
  path: ./ # 배포될 애플리케이션의 위치
  env:
    DEBUG: a*
    API: https://api.<CF 도메인 >
    CF_CLIENT_ID: abacus-cf-bridge
    CF_CLIENT_SECRET: secret
    ABACUS_REPORT_SERVER: http://abacus-usage-reporting.<CF 도메인>
    NODE_TLS_REJECT_UNAUTHORIZED: 0
    NODE_MODULES_CACHE: false
    SECURED: false # abacus-usage-reporting의 SECURED 설정이 true인 경우, true 설정
    AUTH_SERVER: https://api.<CF 도메인>
    CLIENT_ID: abacus
    CLIENT_SECRET: secret
    JWTKEY: |+
       -----BEGIN PUBLIC KEY-----
      -----END PUBLIC KEY-----
    JWTALGO: RS256
```


### <div id='32'/>3.3.3. 배포 형상

  	$ cf a

  	Getting apps in org test / space test as admin...
  	OK
  	name requested state instances memory disk urls
  	paasta-usage-reporting started 1/1 512M 512M paasta-usage-reporting.bosh-lite.com

### <div id='33'/>3.3.4.  api 호출 예제

  	$ curl -k -X GET https://paasta-usage-reporting.bosh-lite.com/v1/org/:org_id/space/:space_id

