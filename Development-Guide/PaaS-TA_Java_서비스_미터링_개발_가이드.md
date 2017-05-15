## Table of Contents

1. [개요](#1)
	* [문서 개요](#2)
	 * [목적](#3)
	 * [범위](#4)
	 * [참고자료](#5)

2. [JAVA API 서비스 미터링 개발가이드](#6)
    * [개요](#7)
    * [개발 환경 구성](#8)
    * [서비스 브로커 라이브러리](#9)
     * [서비스 브로커 라이브러리란 무엇인가?](#10)
     * [서비스 브로커 라이브러리를 다운로드 한 후, 프로젝트 import 한다](#11)
     * [서비스 브로커 라이브러리에서 미터링을 위해 추가 되거나 수정 되는 파일들](#12)
     * [ServiceInstanceBindingController](#13)
     * [ServiceInstanceBinding](#14)
     * [SampleMeteringReportService 추상화 클래스](#15)
     * [SampleMeteringOAuthService 추상화 클래스](#16)
    * [서비스 브로커 라이브러리](#17)
     * [mongo-db 서비스 브로커 API](#18)
     * [mongo-db 서비스 브로커 API 다운로드](#19)
     * [mongo-db 서비스 브로커 API에 추가 및 수정 되는 파일](#20)
     * [gradle build를 위한 dependency 추가](#21)
     * [application-mvc.properties 설정](#22)
     * [datasource.properties 설정](#23)
     * [MongoServiceInstanceBindingService 구현체](#24)
     * [SampleMeteringOAuthService 구현](#25)
     * [SampleMeteringReportService 구현](#26)
    * [미터링/등급/과금 정책](#27)
     * [미터링 정책](#28)
     * [등급 정책](#39)
     * [과금 정책](#30)
     * [정책 등록](#31)
    * [배포](#32)
     * [파스-타 플랫폼 로그인](#33)
     * [mongo-db 서비스 브로커 생성](#34)
     * [API 서비스 연동 샘플 애플리케이션 배포 및 서비스 연결](#35)
    * [서비스 바인딩 CF-Abacus 연동 테스트](#36)
    * [단위 테스트](#37)
    * [샘플 코드](#38)


# <div id='1'/>1.  개요
## <div id='2'/> 1.1 문서 개요
### <div id='3'/>1.1.1.  목적


본 문서(Java 서비스브로커 미터링 애플리케이션 개발 가이드)는 파스-타
플랫폼 프로젝트의 서비스 브로커에 미터링 서비스를 추가하여, CF(Cloud
Foundry) 서비스를 미터링 하는 방법에 대해 기술 한다.


### <div id='4'/>1.1.2.  범위

본 문서의 범위는 파스-타 플랫폼 프로젝트의 Cloud Foundry JAVA 서비스
브로커 애플리케이션 미터링 개발과 CF-Abacus 연동에 대한 내용으로
한정되어 있다. 서비스브로커 API 개발에 대해서는 별도 제공 하는
서비스브로커 API 개발 가이드를 참고 한다.

본 문서는 Ubuntu 14.04 ver의 개발 환경을 전제로 기술 한다.

본 문서는 mongo-db 서비스 팩이 설치 되어 있는 개발 환경을 전제로 기술
한다.

mongo-db 서비스 팩 설치는 별로 로 제공 되는 문서를 참고 하여 설치 한다.

**[https://github.com/OpenPaaSRnD/Documents/blob/master/Service-Guide/NOSQL/OpenPaaS_PaaSTA_ServicePack_MongoDB_BOSH-Lite_install_guide.md](https://github.com/OpenPaaSRnD/Documents/blob/master/Service-Guide/NOSQL/OpenPaaS_PaaSTA_ServicePack_MongoDB_BOSH-Lite_install_guide.md)**


본 문서는 cf-abacus 가 설치 되어 있는 개발 환경을 전제로 기술 한다.
(cf-abacus 설치는 별도 제공하는 Abacus 설치 가이드를 참고하여
CF-Abacus를 설치한다.)

## <div id='5'/>1.3.  참고 자료

-   **[https://docs.cloudfoundry.org/devguide/](https://docs.cloudfoundry.org/devguide/)**
-   **[http://cli.cloudfoundry.org/ko-KR/cf/](http://cli.cloudfoundry.org/ko-KR/cf/)**
-   **[https://github.com/cloudfoundry-community/spring-boot-cf-service-broker/](https://github.com/cloudfoundry-community/spring-boot-cf-service-broker)**
-   **[https://github.com/cloudfoundry-incubator/cf-abacus](https://github.com/cloudfoundry-incubator/cf-abacus)**


# <div id='6'/>2.  Java서비스 미터링 개발가이드


## <div id='7'/>2.1.  개요


CF Services 는 Service Broker API 라고 불리우는 cloud controller
클라이언트 API를 구현하여 개방형 클라우드 플랫폼에서 사용된다. Services
API는 독립적인 cloud controller API의 버전이다. 이는 플랫폼에서 외부
application을 이용 가능하게 한다. (database, message queue, rest
endpoint, etc.)

개방형 클라우드 플랫폼 Service API는 Cloud Controller 와 Service Broker
사이의 규약 (catalog, provision, de provision, update provision plan,
bind, unbind)이고 Service Broker 는 RESTful API 로 구현하고 Cloud
Controller 에 등록한다.

서비스에 미터링 구현하고자 할 때, 이 규약들 중 서비스 정책 및 취지에
맞는 프로세스를 선택하여, 그 프로세스에 미터링을 연동할 수 있다.

본 개발가이드에서는 mongo-db 서비스를 예시로, bind 와 unbind 시 미터링을
하는 방법에 대해 가이드 한다.

서비스를 사용하고자 하는 애플리케이션과 API 서비스를 바인딩 할 때, CF
CLI 바인딩 요청 request에 적용 된 애플리케이션 환경정보(org guid, space
guid, app guid, metering plan id) 를 이용해 바인딩 정보를 획득 하여,
서비스 요청을 처리함과 동시에 서비스의 사용 내역을 CF-ABACUS에 전송하는
미터링 서비스 기능을 mongo-db 서비스 브로커에 추가하여 개발 한다.


Service Broker API Architecture

![Java_Service_Metering_Image01]

![Java_Service_Metering_Image02]

<table>
  <tr>
    <th colspan ="2">기능</th>
     <th>설명</th>
  </tr>
  <tr>
     <td rowspan="4">Runtime</td>
     <td>미터링/등급/과금 정책</td>
     <td>서비스 제공자가 제공하는 서비스에 대한 각종 정책 정의 정보. JSON 형식으로 되었으며, 해당 정책을 CF-ABACUS에 등록하면 정책에 정의한 내용에 따라 서비스 사용량을 집계 한다.
정책은 서비스 제공자가 정의해야 하며, JSON 스키마는 다음을 참조한다. <br>
<a href = "https://github.com/cloudfoundry-incubator/cf-abacus/blob/master/doc/api.md" >https://github.com/cloudfoundry-incubator/cf-abacus/blob/master/doc/api.md
</a>
</td>
  </tr>
   <tr>
     <td width="160px">서비스 브로커 API</td>
     <td>Cloud Controller와 Service 사이에서 서비스의 create, delete, update, bind, unbind를 처리한다. 본문서에서는 mongo-db 서비스 브로커 API를 대상으로 미터링 서비스를 개발 하여 추가 한다.
</td>
  </tr> 
  <tr>
     <td>서비스 브로커 <br>미터링 서비스
</td>
     <td>서비스 브로커 API 가 abacus-usage-collector 에 서비스 사용량 정보를 전송하는 서비스 (본 문서에서 가이드 할 개발 영역 이다)</td>
  </tr> 
   <tr>
     <td>서비스</td>
     <td>서비스 제공자가 제공하는 서비스 기능</td>
  </tr> 
  <tr>
  	<td colspan ="2">CF-ABACUS</td>
    <td>CF-ABACUS 핵심 기능으로써 수집한 사용량 정보를 집계한다.<br>
CF-ABACUS은 CF 설치 후, CF에 마이크로 서비스 형태로 설치한다. 자세한 사항은 다음을 참조한다.<br>
<a href = "https://github.com/cloudfoundry-incubator/cf-abacus" >https://github.com/cloudfoundry-incubator/cf-abacus</a>
</td>
  </tr>
</table>                


※ 본 개발 가이드는 ***애플리케이션과 서비스가 바인드 되는 시점을 서비스의 이용 시작으로 판단할 수 있는 서비스에 대해 미터링 하는 기능 개발***에 대해서만 기술한다.

※ ***서비스의 특정 API 호출 , 서비스의 특정 자원 이용 등에 대한 미터링 기능 개발에 대해서는 기술하지 않는다.***
※ API 호출에 대한 미터링은 API 서비스 미터링 개발 가이드를 참조한다.

※ 다른 컴포넌트의 개발 또는 설치에 대해서 링크한 사이트를 참조한다.


## <div id='8'/>2.2.  개발환경 구성


서비스 미터링 개발을 위해 다음과 같은 환경을 개발환경을 전제 한다.

-   CF release: v226 이상 (bosh-lite 설치 환경에서 테스트)
-   gradle 2.14
-   java version "1.8.0_101"
-   springBootVersion: 1.3.0. BUILD-SNAPSHOT
-   mongo-db service broker 2.5 (미터링 서비스를 추가 하기 위한 대상
    서비스 브로커 프로젝트)
-   springBootCfServiceBrokerVersion "2.5.0" (서비스 브로커 라이브러리)
-   Spring Tool Suite 혹은 Eclipse


## <div id='9'/>2.3.  서비스 브로커 라이브러리



### <div id='10'/>2.3.1.  서비스 브로커 라이브러리는 무엇인가?


CF (개방형 플랫폼) 에서는 플랫폼 상에서 서비스 할 수 있는 다양한
서비스들이 존재한다.<br>
이 서비스들은 각각 그 서비스 고유의 서비스 브로커를 개발 함으로써, CF
(개방형 플랫폼) 에서 애플리케이션이 서비스를 사용 할 수 있도록 하고
있다.<br>
서비스는 다양하지만, 서비스를 사용하기 위한 개방형 플랫폼의 RESTAPI가
미리 정해져 있다.<br>
서비스 브로커 라이브러리는 각각 다른 서비스 브로커들이 서비스 브로커
라이브러리 Jar 파일을 build path 에 추가 하고, 추상화 클래스들을 구현
하는 것으로 이 개방형 플랫폼의 REST API에 기반 하여, 서비스가 제공 될 수
있도록 해주는 라이브러리 이다.

본 가이드에서는 mongo-db 서비스 브로커에 미터링 서비스를 구현하기
위해서는 이 서비스 브로커 라이브러리에 미터링을 하기 위한 추상화
클래스를 추가 한 후, mongo-db 서비스 브로커에서 이 라이브러리를
dependency로 사용하여 빌드 한다.


### <div id='11'/>2.3.2.  서비스 브로커 라이브러리를 다운로드 한 후, 프로젝트 import 한다.

#### 1.  오픈 소스로 제공되고 있는 서비스 브로커 소스를 git clone 으로 다운받는다.<br>

**[https://github.com/cloudfoundry-community/spring-boot-cf-service-broker/tree/master/src/main/java/org/cloudfoundry/community/servicebroker/controller](https://github.com/cloudfoundry-community/spring-boot-cf-service-broker/tree/master/src/main/java/org/cloudfoundry/community/servicebroker/controller)**
  
	$ git clone https://github.com/cloudfoundry-community/spring-boot-cf-service-broker.git
	Cloning into 'spring-boot-cf-service-broker'...
	remote: Counting objects: 2394, done.
	remote: Total 2394 (delta 0), reused 0 (delta 0), pack-reused 2394
	Receiving objects: 100% (2394/2394), 351.72 KiB | 279.00 KiB/s, done.
	Resolving deltas: 100% (939/939), done.
	Checking connectivity... done.


다운 받은 소스를 Java 개발 도구 Eclipse 및 Spring Tool Suite 로 import
한다.

gradle 플러그인을 Eclipse 에 추가한 후, gradle import 하면 개발이 보다
용이 해진다.


### <div id='12'/>2.3.3.  서비스 브로커 라이브러리에서 미터링을 위해 추가 되거나 수정 되는 파일들


| 　　  |Java class | 설명|
|---------|---|----|
|    수정     | ServiceIncetanceBindingController  | 클라우드 컨트롤러의 서비스 바인딩 요청을 처리하는 컨트롤러,<br> SampleMeteringOAuthService 에서 uaa token 을 취득하여, SampleMeteringReportService 의 파라메터로 호출 하는 프로세스를 추가 한다.   |
|    수정     | ServiceInstanceBinding  | service-binding-request 가 ServiceIncetanceBindingController 에서 처리 될 때 바인딩 연결에 대해 미터링이 적용된 사용량 보고서를 abacus-usage-collector 에 리포팅 한다.   |     
|    추가     | SampleMeteringReportService  | SampleMeteringReportService 추상화 된 인터페이스로서, 미터링/등급/과금 정책과 관련된 그 어떠한 정보도 가지고 있지 않다. 이는 이 인터페이스를 구현할 서비스 제공자가 서비스 구현체에 적용할 수 있도록 제공되고 있는 추상화 클래스 이다.<br>SampleMeteringReportService 추상화 된 인터페이스로서, 미터링/등급/과금 정책과 관련된 그 어떠한 정보도 가지고 있지 않다. 이는 이 인터페이스를 구현할 서비스 제공자가 서비스 구현체에 적용할 수 있도록 제공되고 있는 추상화 클래스 이다.|     
|    추가     | SampleMeteringOAuthService  | 개방형 플랫폼 상의 UAA 서버에서 abacus-usage-collector 에 대한 접근 권한 토큰을 취득하여, SampleMeteringReportService 에 토큰을 전달 하기 위한 추상화 클래스 이다.   |


서비스 브로커 라이브러리에서 미터링을 위해 추가 되거나 수정 되는 파일의
형상

![Java_Service_Metering_Image04]

### <div id='13'/>2.3.4.  ServiceInstanceBindingController

bindServiceInstance 프로세스 에 SampleMeteringOAuthService 에서 uaa
token 을 취득하여, SampleMeteringReportService 의 파라메터로 호출 하는
프로세스를 추가 한다.

	@RequestMapping (value = BASE_PATH + "/{bindingId}", method = RequestMethod.PUT)
	public ResponseEntity<ServiceInstanceBindingResponse> bindServiceInstance (
			@PathVariable("instanceId") String instanceId, 
			@PathVariable("bindingId") String bindingId,
			@Valid @RequestBody CreateServiceInstanceBindingRequest request) throws
			ServiceInstanceDoesNotExistException, ServiceInstanceBindingExistsException, 
			ServiceBrokerException {
		ServiceInstance instance = serviceInstanceService.getServiceInstance(instanceId);
		if (instance == null) {
			throw new ServiceInstanceDoesNotExistException(instanceId);
		}
		ServiceInstanceBinding binding = serviceInstanceBindingService.createServiceInstanceBinding(			request. withServiceInstanceId(instanceId).and (). withBindingId(bindingId));
	
		if (binding == null) {
			throw new ServiceBrokerException(bindingId);
		}
		
		String uaaToken = sampleMeteringOAuthService.getUAAToken();
		
		try {
			int httpStatus = sampleMeteringReportService.reportServiceInstanceBinding(binding, uaaToken); 
			if (httpStatus == 400) {
				throw new ServiceBrokerException(bindingId);
			} 	
		} catch (ServiceBrokerException e) {
			throw e;
		}
		
		logger. debug ("ServiceInstanceBinding Created: " + binding. getId());
	return new ResponseEntity<ServiceInstanceBindingResponse>(
			new ServiceInstanceBindingResponse(binding), 
			binding.getHttpStatus());
	}


### <div id='14'/>2.3.5.  ServiceInstanceBinding 

ServiceInstanceBinding 에 미터링 서비스를 구현하기 위해 바인딩 되는
애플리케이션의 환경 정보 필드를 추가 한다. 추가 된 필드 들은
ServiceInstanceBindingService 의 구현체 에서 서비스 바인딩 request
parameter 의 필드 값들을 매핑 처리 한 후, mongo-db repository 에 전달될
것이다. 라이브러리를 gradle build 한다.

	package org.openpaas.servicebroker.model;
	
	import java.util.HashMap;
	import java.util.Map;
	import org.springframework.http.HttpStatus;
	import com.fasterxml.jackson.annotation.JsonIgnore;

	public class ServiceInstanceBinding {
	
	private String id;
	private String serviceInstanceId;
	private Map<String,Object> credentials = new HashMap<String,Object>();
	private String syslogDrainUrl;
	// 미터링에 사용되는 필드
	private String appGuid;
	
	// 미터링을 위해 추가 된 필드
	private String appOrganizationId;
	private String appSpaceId;
	private String meteringPlanId;
	
	@JsonIgnore
	private HttpStatus httpStatus = HttpStatus.CREATED;
	public ServiceInstanceBinding (String id, 
			String serviceInstanceId, 
			Map<String,Object> credentials,
			String syslogDrainUrl, String appGuid,
			String appOrganizationId,
			String appSpaceId,
			String meteringPlanId			
			) {
		this.id = id;
		this.serviceInstanceId = serviceInstanceId;
		setCredentials(credentials);
		this.syslogDrainUrl = syslogDrainUrl;
		this.appGuid = appGuid;
		
		this.appOrganizationId = appOrganizationId;		
		this.appSpaceId = appSpaceId;		
		this.meteringPlanId = meteringPlanId;
	}


### <div id='15'/>2.3.6.  SampleMeteringOAuthService 추상화 클래스

UAA OAuthToken은 Abacus가 Secured로 운영될 경우, abucus-collector
RESTAPI에 접근 하기 위해 필요하다.<br>
SampleMeteringOAuthService를 상속하는 클래스는 UAA OAuthToken을 취득하여
리턴하는 처리를 구현해야 한다.

	package org.openpaas.servicebroker.service;
	import org.openpaas.servicebroker.exception.ServiceBrokerException;
	
	public interface SampleMeteringOAuthService {
	String getUAAToken() throws ServiceBrokerException;
	}


### <div id='16'/>2.3.7.  SampleMeteringReportService 추상화 클래스
SampleMeteringReportService를 상속하는 클래스는 create binding request와
delete binding request를 처리 할 때, 각각 해당 이벤트 정보를
abacus-collector에 전송하고 해당 처리에 대한 상태코드(HTTP 상태코드)를
리턴하는 처리를 구현해야 한다.

	package org.openpaas.servicebroker.service;
	import org.openpaas.servicebroker.exception.ServiceBrokerException;
	import org.openpaas.servicebroker.model.ServiceInstanceBinding;
	public interface SampleMeteringReportService {
	int reportServiceInstanceBinding(ServiceInstanceBinding serviceInstanceBinding, 
			String uaaToken)
			throws ServiceBrokerException;
	
	int reportServiceInstanceBindingDelete(ServiceInstanceBinding serviceInstanceBinding, 
			String uaaToken)
			throws ServiceBrokerException;	
	
	}




## <div id='17'/>2.4.  서비스 브로커 라이브러리

### <div id='18'/>2.4.1.  mongo-db 서비스 브로커 API

지금까지 서비스 브로커 라이브러리를 개수 하여, 미터링을 위한 추상화
클래스 및 모델 객체들을 준비 했다. 지금 부터는 서비스 브로커
라이브러리를 구현한 mongo-db 서비스 브로커 API에서 미터링을 구현 하는
것에 대해 기술 한다.

### <div id='19'/>2.4.2.  mongo-db 서비스 브로커 API 다운로드

mongo-db 서비스 브로커 API는 별도 제공되는 압축 파일 패키지를 사용한다.

### <div id='20'/>2.4.3.  mongo-db 서비스 브로커 API에 추가 및 수정 되는 파일

| 　　|유형 | 필수|
|---------|---|----|
|   수정      |build.gradle   |  빌드 설정 파일<br>미터링 사용량 객체 생성에 필요한 dependency 를 추가 한다.|     
|   수정      | application-mvc.properties  | 서비스 바인딩 request 의 정보들을 매핑한다.<br>미터링 서비스를 구현하기 위해 바인딩 되는 애플리케이션의 환경정보 필드를 추가 한다.|     
|   수정      | datasource.properties   | Mongo-db 서비스 정보   |     
|   수정     | MongoServiceInstanceBindingService  |service broker binding request parameter 로 입력 받은 미터링 정보를 ServiceInstanceBinding 에 매핑하는 프로세스를 추가 한다.    |     
|   추가      | SampleMeteringReportServiceImpl  | SampleMeteringReportService 를 구현 한다.   |     
|   추가     |SampleMeteringOAuthServiceImpl   | SampleMeteringOAuthService 를 구현 한다.   |     
|   수정     |Manifest.yml   | 앱을 CF에 배포할 때 필요한 설정 정보 및 앱 실행 환경에 필요한 설정 정보를 기술한다.   |


### <div id='21'/>2.4.4.  gradle build를 위한 dependency 추가
서비스브로커 라이브러리 mongo-db서비스 브로커 jar 파일을 적용

![Java_Service_Metering_Image03]

서비스 브로커 라이브러리를 gradle build 한다.

	@openpaas-service-broker/openpaas-service-java-broker$ gradle build -x test
	:compileJava
	:processResources UP-TO-DATE
	:classes
	:findMainClass
	:jar
	:bootRepackage
	:assemble
	:check
	:build
	
	BUILD SUCCESSFUL
	
	Total time: 58.845 secs



빌드가 성공하면
/openpaas-service-java-broker/build/libs/openpaas-service-java-broker.jar가
생성 된다.

이 jar 파일을 mongo-db 서비스 브로커의
/openpaas-service-java-broker-mongo/libs 경로로 복사 하고, mongo-db
서비스브로커 gradle build 파일에 dependency를 추가 한다.


mongo-db 서비스 브로커 build.gradle 파일의 dependencies 부분

	dependencies {
	    // 서비스브로커 라이브러리 
	    compile files('libs/openpaas-service-java-broker.jar')
	    // 미터링 사용량 객체 생성 dependency
	    compile("org.json:json:20160212")
	    compile("com.googlecode.json-simple:json-simple:1.1")
	
	    providedRuntime("org.springframework.boot:spring-boot-starter-tomcat:${springBootVersion}")
	    compile("org.springframework.boot:spring-boot-starter-web:${springBootVersion}")
	    compile("org.springframework.boot:spring-boot-starter-security:${springBootVersion}")
	    	//testCompile("org.cloudfoundry:spring-boot-cf-service-broker-tests:${springBootCfServiceBrokerVersion}")
	    testCompile("org.springframework.boot:spring-boot-starter-test:${springBootVersion}")
	    testCompile("com.jayway.jsonpath:json-path:${jsonPathVersion}")
	    testCompile("org.apache.httpcomponents:httpclient:4.4.1")
	    
	    testCompile("org.powermock:powermock-mockito-release-full:1.6.1")    
		compile("org.apache.commons:commons-dbcp2")    
	    compile("org.springframework.boot:spring-boot-starter-data-mongodb:${springBootVersion}")
	}


### <div id='22'/>2.4.5.  application-mvc.properties 설정

	# abacus usage collector RESTAPI 의 주소
	abacus.collector: https://abacus-usage-collector.<파스-타 도메인> /v1/metering/collected/usage
	# abacus usage collector 가 secured 모드 true / 아닐 경우 false
	abacus.secured: true
	# 개발형 플랫폼의 uaa server 
	uaa.server: https://uaa.<파스-타 도메인>
	# abacus usage collector RESTAPI 사용권한 (UAA server 에 미리 등록한다.)
	uaa.client.id: abacus-linux-container
	uaa.client.secret: secret
	uaa.client.scope: abacus.usage.linux-container.write,abacus.usage.linux-container.read 


uaa 계정 설정 방법에 관해서 별도의 **abacus****설치 가이드**의 **Secured
Abacus****를 위한****UAA****계정 등록**을 참고한다.


### <div id='23'/>2.4.6.  datasource.properties 설정

	# Mongo-DB 서비스 배포 manifest파일을 참조하여 설정한다.
	mongodb.hosts = 10.244.14.2, 10.244.14.14, 10.244.14.26
	mongodb.port = 27017
	mongodb.dbName = mongo-broker
	mongodb.userName = root
	mongodb.authSource = admin
	mongodb.password = openpaas

### <div id='24'/>2.4.7.  MongoServiceInstanceBindingService 구현체

애플리케이션 환경 정보는 service broker binding CLI 요청 시, parameter
객체를 통해서 입력된다. 이 정보 들을 ServiceInstanceBinding에 미터링
필드를 매핑한다.

-   service broker binding CLI 요청 예제

  	    $ cf bind-service sample-api-node-caller mongod_service -c 
		'{"app_organization_id":"test05","app_space_id":"testspaceId","metering_plan_id":"standard"}'


parameter 넘어온 정보들을 mongo-db 에 저장하기 위해
ServiceInstanceBinding 객체에 매핑한다. mongo-db repository 를 통해 저장
후, 바인딩 정보를 리턴 한다.
	
	// parameter 로 입력 받은 미터링 관련 정보를 취득하여 ServiceInstanceBinding 에 매핑한다.
	Map<String, Object> paraMap = request.getParameters();
	String appOrganizationId = (String) paraMap.get("app_organization_id");
	String appSpaceId = (String) paraMap.get("app_space_id");
	String meteringPlanId = (String) paraMap.get("metering_plan_id");
	binding = new ServiceInstanceBinding(request.getBindingId(), request.getServiceInstanceId(), credentials, null,request.getAppGuid(), appOrganizationId, appSpaceId, meteringPlanId);
	
	repository.save(binding);
	
	return binding;


### <div id='25'/>2.4.8.  SampleMeteringOAuthService 구현

application-mvc.properties의 UAA server에서 UAA 토큰을 취득하기 위한
정보들을 클래스로 호출한다.

	@Component
	@Service
	public class SampleMeteringOAuthServiceImpl implements SampleMeteringOAuthService {
		@Value("${uaa.server}")
		String authServer;
		
		@Value("${uaa.client.id}")
		String clientId;
		
		@Value("${uaa.client.secret}")
		String clientSecret;
		
		@Value("${uaa.client.scope}")
		String scope;
		
		@Value("${abacus.secured}")
		String abacusSecured;



SampleMeteringOAuthServiceImpl은 SampleMeteringOAuthService를 구현한다.

SampleMeteringOAuthServiceImpl는 https 커넥션을 생성하여 UAA 서버에
토큰을 요청한다. 이때 abacusSecured (abacus-collector 의 secured 설정
여부) 에 따라 공백({}) 또는 토큰을 리턴 한다.

	@Override
	public String getUAAToken() throws ServiceBrokerException {
		
	if(!SECURED.equals(abacusSecured)){
		return "";
	} else {
		String authToken = "";		
		StringBuffer sb = new StringBuffer();
		
		HttpsURLConnection conn = (HttpsURLConnection) getConnetionUAA();
	        conn.setRequestMethod("GET");
	        conn.setDoInput(true);
	        String authHeader = getAuthKey(clientId, clientSecret);
	        conn.setRequestProperty("authorization", authHeader);
	
		InputStreamReader in = new InputStreamReader((InputStream) conn.getContent());
		BufferedReader br = new BufferedReader(in);
	
		String line;
		while ((line = br.readLine()) != null) {
			sb.append(line).append("\n");
		}
	
		authToken= parseAuthToken(sb.toString());
		br.close();
		in.close();
		conn.disconnect();
		return authToken;
	} 


### <div id='26'/>2.4.9.  SampleMeteringReportService 구현

SampleMeteringReportServiceImpl에서는 SampleMeteringOAuthServiceImpl에서
취득한 uaa token 으로 https 커넥션을 생성하여, abacus-collector에 서비스
사용량 정보를 POST 한다.

abacus-collector 에서는 미터링 정책에 따라 POST 받을 양식에 대한
프로세스를 준비 하고 있기 때문에 abacus-collector가 알 수 있는 양식으로
JSON을 생성 후 POST 한다.

SampleMeteringReportServiceImpl 은 크게 나누어 2가지 처리를 하고 있다.


#### 1.  **ServiceInstanceBinding 정보를 참조 하여 ,사용량 정보 JSON을 생성 한다.**

#### 2.  **생성한 사용량 정보 JSON을 abacus-collector로 전송한다. (HTTPS, HTTP)**


사용량 정보 JSON 을 생성 한다.

RESOURCE_ID linux-container 와 STANDARD_PLAN_ID standard 는
abacus에서 sample 로 제공 되는 미터링 스키마이다.

본 가이드에서는 이 미터링 스키마를 mongo-db 서비스 바인딩과 언바인딩에
대한 미터링 스키마로 이용하여 기술 했다.

서비스 제공자는 제공 하려는 서비스에 맞는 정책을 정하여, 미터링 스키마를
abacus-프로비저닝에 등록 해야, abacus-collector 에 미터링을 전송할 수
있게 된다. (정책 등록에 대한 자세한 내용은 본문 하기의 **미터링/과금 정책 참조**)


다음 예제의 미터링 리포팅 용 상수들은 abacus의 linux-container 미터링
스키마에 맞게 기술 되었고, PLAN_STANDARD_QUANTITY,
PLAN_EXTRA_QUANTITY 등은 임의로 정한 수치 이다. 서비스에 맞게 해당
항목을 DB 또는 프로퍼티 등을 통해 설정한다.

	// 미터링 리포트용 상수
	private static final String RESOURCE_ID = "linux-container";
	private static final int BIND = 1;
	private static final int UNBIND = 0;
	private static final String MEASURE_1 = "sample_service_usage_param1";
	private static final String MEASURE_2 = "sample_service_usage_param2";
	private static final String MEASURE_3 = "previous_sample_service_usage_param1";
	private static final String MEASURE_4 = "previous_sample_service_usage_param2";
	private static final String STANDARD_PLAN_ID = "standard";
	private static final int PLAN_STANDARD_QUANTITY = 50000000;
	private static final int PLAN_EXTRA_QUANTITY = 1000000000;
	private static final String SECURED = "true";
	
	/***************************************************
	 * @description : 리포트 용 JSON 생성
	 * @title : buildServiceUsage
	 * @return : JSONObject
	 ***************************************************/
	public JSONObject buildServiceUsage(ServiceInstanceBinding binding, int mode) {
	
		String orgId = (String) binding.getAppOrganizationId();
		String spaceId = (String) binding.getAppSpaceId();
		String planId = (String) binding.getMeteringPlanId();
		String appId = (String) binding.getAppGuid();
	
		LocalDateTime now = LocalDateTime.now();
		Timestamp timestamp = Timestamp.valueOf(now);
	
		JSONObject jsonObjectUsage = new JSONObject();
		jsonObjectUsage.put("start", timestamp.getTime());
		jsonObjectUsage.put("end", timestamp.getTime());
		jsonObjectUsage.put("organization_id", orgId);
		jsonObjectUsage.put("space_id", spaceId);
		jsonObjectUsage.put("consumer_id", "app:" + appId);
		jsonObjectUsage.put("resource_id", RESOURCE_ID);
		jsonObjectUsage.put("plan_id", planId);
		jsonObjectUsage.put("resource_instance_id", appId);
		JSONArray measuredUsageArr = new JSONArray();
		JSONObject measuredUsage1 = new JSONObject();
		JSONObject measuredUsage2 = new JSONObject();
		JSONObject measuredUsage3 = new JSONObject();
		JSONObject measuredUsage4 = new JSONObject();
	
		int quantity = 0;
		if (STANDARD_PLAN_ID.equals(planId)) {
			quantity = PLAN_STANDARD_QUANTITY;
		} else {
			quantity = PLAN_EXTRA_QUANTITY;
		}
		if (mode == BIND) {
			measuredUsage1.put("measure", MEASURE_1);
			measuredUsage1.put("quantity", quantity);
			measuredUsageArr.put(measuredUsage1);
			measuredUsage2.put("measure", MEASURE_2);
			measuredUsage2.put("quantity", 1);
			measuredUsageArr.put(measuredUsage2);
			measuredUsage3.put("measure", MEASURE_3);
			measuredUsage3.put("quantity", 0);
			measuredUsageArr.put(measuredUsage3);
			measuredUsage4.put("measure", MEASURE_4);
			measuredUsage4.put("quantity", 0);
			measuredUsageArr.put(measuredUsage4);
		} else { // UNBIND
			measuredUsage1.put("measure", MEASURE_1);
			measuredUsage1.put("quantity", 0);
			measuredUsageArr.put(measuredUsage1);
			measuredUsage2.put("measure", MEASURE_2);
			measuredUsage2.put("quantity", 0);
			measuredUsageArr.put(measuredUsage2);
			measuredUsage3.put("measure", MEASURE_3);
			measuredUsage3.put("quantity", quantity);
			measuredUsageArr.put(measuredUsage3);
			measuredUsage4.put("measure", MEASURE_4);
			measuredUsage4.put("quantity", 1);
			measuredUsageArr.put(measuredUsage4);
		}
		jsonObjectUsage.put("measured_usage", measuredUsageArr);
		return jsonObjectUsage;
	}


**abacus-collector** **인터페이스 항목**

| 항목명                 |유형             | 설명                                          | 예시                                           |
|-----------------------|-----------------|----------------------------------------------|------------------------------------------------|
|   start               | UNIX Timestamp  |바인드/언바인드 처리 시작 시각                   |1396421450000                                   |
|   end                 | UNIX Timestamp  |  바인드/언바인드 처리 응답 시각                 |1396421451000                                   |
|  organization_id      | String          | 바인드 요청을 호출한 앱의 조직 ID               | us-south:54257f98-83f0-4eca-ae04-9ea35277a538  |
|   space_id            |String           | 서비스 바인드 요청을 호출한 앱의 영역 ID         |d98b5916-3c77-44b9-ac12-04456df23eae            |
|  consumer_id          | String          |서비스 바인드 요청을 호출한 앱 ID                | App: d98b5916-3c77-44b9-ac12-04d61c7a4eae      |
|  resource_id          |String           |서비스 자원 ID                                 |linux-container                                 |
|  plan_id              |String           | 서비스 미터링 Plan ID                         |standard                                        |
|  resource_instance_id | String          |바인드 요청을 호출한 앱 ID                      | d98b5916-3c77-44b9-ac12-04d61c7a4eae            |
|  measured_usage       | Array           | 미터링 항목                                   | -                                         |
|   measure             | String          | 미터링 대상 명                                |sample_service_usage_param1                     |
|  quantity             |Number           |  서비스 사용량 예제는 메모리 사용량 (byte)      |1000000000                                       |


※ JSON 변환 예제

	{  
	   "consumer_id":"app:d98b5916-3c77-44b9-ac12-04d61c7a4eae ",
	   "resource_instance_id":"d98b5916-3c77-44b9-ac12-04d61c7a4eae ",
	   "organization_id":" us-south:54257f98-83f0-4eca-ae04-9ea35277a538",
	   "measured_usage":[  
	      {  
	         "measure":"sample_service_usage_param1",
	         "quantity":50000000
	      },
	      {  
	         "measure":"sample_service_usage_param2",
	         "quantity":0
	      },
	      {  
	         "measure":"previous_sample_service_usage_param1",
	         "quantity":50000000
	      },
	      {  
	         "measure":"previous_sample_service_usage_param2",
	         "quantity":1
	      }
	   ],
	   "start":1396421450000,
	   "resource_id":"linux-container",
	   "end":1396421450000,
	   "space_id":" d98b5916-3c77-44b9-ac12-04456df23eae ",
	   "plan_id":"standard"
	}



# <div id='27'/>2.5.  미터링/등급/과금 정책

서비스, 그리고 서비스 제공자 마다 미터링/등급/과금 정책 다르기 때문에 본
가이드에서는 정책의 개발 예제를 다루지는 않는다. 다만 CF-ABACUS에 적용할
수 있는 형식에 대해 설명한다.


### <div id='28'/>2.5.1.  미터링 정책
미터링 정책 스키마
미터링 정책이란 수집한 미터링 정보에서 미터링 대상의 지정 및 집계 방식을
정의한 JSON 형식의 오브젝트이다. 서비스 제공자는 미터링 정책 스키마에
맞춰 서비스에 대한 정책을 개발한다.


#### 1.  **미터링 정책 스키마**

| 항목명  |유형 | 필수| 설명|
|---------|---|----|-----|
|plan_id      |  String | O   |  API 서비스 미터링 Plan ID   |
|measures     | Array  |  최소 하나  |  API 서비스 미터링 정보 수집 대상 정의   |
|    name     | String  | O   |  미터링 정보 수집 대상 명   |
|    unit     | String  |  O  |  미터링 정보 수집 대상 단위   |
|metrics     | Array  | 최소 하나   | API 서비스 미터링 집계 방식 정의    |
|    name     |  String | O   | 미터링 정보 수집 대상 명    |
|    unit      |  String |  O  | 미터링 정보 수집 대상 단위    |
|    meter     |  String | X   | 미터링 정보에 대해서 수집 단계에 적용하는 계산식 또는 변환 식    |
|    accumulate      |String   |  X  | 미터링 정보에 대해서 누적 단계에 적용하는 계산식 또는 변환식    |
|    aggregate      |  String |  X  | 미터링 정보에 대해서 집계 단계에 적용하는 계산식 또는 변환식    |
|    summarize      | String  |  X  | 미터링 정보를 보고할 때 적용하는 계산식 또는 변환식    |
|    title      |  String |   X | API 서비스 미터링 제목    |

#### 2.  **미터링 정책 예제**

	{
	  "plan_id": "basic-linux-container",
	  "measures": [
	    {
	      name: 'sample_service_usage_param1',
	      unit: ‘SAMPLE_UNIT’
	    },
	    {
	      name: 'sample_service_usage_param2',
	      unit: ‘SAMPLE_UNIT’
	    },
	    {
	      name: 'previous_service_usage_param1',
	      unit: ‘SAMPLE_UNIT’
	    },
	    {
	      name: 'previous_service_usage_param2',
	      unit: ‘SAMPLE_UNIT’
	    } ],
	  "metrics": [
	    {
	      name: 'sample_metric',
	      unit: ‘SAMPLE_UNIT’,
	      type: 'time-based',
	
	      meter: ((m) => ({
	        previous_consuming: new BigNumber(m.previous_instance_memory || 0)
	          .div(1073741824).mul(m.previous_running_instances || 0)
	          .mul(-1).toNumber(),
	        consuming: new BigNumber(m.current_instance_memory || 0)
	          .div(1073741824).mul(m.current_running_instances || 0).toNumber()
	      })).toString(),
	
	      accumulate: ((a, qty, start, end, from, to, twCell) => {
	        // Do not accumulate usage out of boundary
	        if (end < from || end >= to)
	          return null;
	
	        const past = from - start;
	        const future = to - start;
	        const td = past + future;
	        return {
	          // Keep the consuming & since to the latest value
	          consuming: a && a.since > start ? a.consuming : qty.consuming,
	          consumed: new BigNumber(qty.consuming).mul(td)
	            .add(new BigNumber(qty.previous_consuming).mul(td))
	            .add(a ? a.consumed : 0).toNumber(),
	          since: a && a.since > start ? a.since : start
	        };
	      }).toString(),
	
	      aggregate: ((a, prev, curr, aggTwCell, accTwCell) => {
	        // Usage was rejected by accumulate
	        if (!curr)
	          return a;
	
	        const consuming = new BigNumber(curr.consuming)
	          .sub(prev ? prev.consuming : 0);
	        const consumed = new BigNumber(curr.consumed)
	          .sub(prev ? prev.consumed : 0);
	        return {
	          consuming: consuming.add(a ? a.consuming : 0).toNumber(),
	          consumed: consumed.add(a ? a.consumed : 0).toNumber()
	        };
	      }).toString(),
	
	      summarize: ((t, qty, from, to) => {
	        // no usage
	        if (!qty)
	          return 0;
	        // Apply stop on running instance
	        const rt = Math.min(t, to ? to : t);
	        const past = from - rt;
	        const future = to - rt;
	        const td = past + future;
	        const consumed = new BigNumber(qty.consuming)
	          .mul(-1).mul(td).toNumber();
	        return new BigNumber(qty.consumed).add(consumed)
	          .div(2).div(3600000).toNumber();
	      }).toString()
	    }
	  ]
	};


### <div id='29'/>2.5.2.  등급 정책 

등급 정책이란 각 서비스의 사용 가중치를 정의한 JSON 형식의 오브젝트이다.
서비스 제공자는 등급 정책 스키마에 맞춰 서비스에 대한 정책을 개발한다.


#### 1.  **등급 정책 스키마**

| 항목명  |유형 | 필수| 설명|
|---------|---|----|-----|
|plan_id      |  String | O   |  API 서비스 미터링 Plan ID   |
|metrics     | Array  |  최소 하나  |  등급 정책 목록   |
|    name     | String  | O   |  등급 정의 대상 명   |
|    rate     | String  |  X  |  가중치 계산식 또는 변환식   |
|    charge     | String  | X   | 사용량에 대한 과금 계산식 또는 변환식    |
|    title     |  String | X   | 등급 정책 명    |


####2.  **등급 정책 예제**

	{
	  "plan_id": "standard",
	  "metrics": [
	    {
	      "name": "sample_metrics"
	    },
	    {
	      "name": "sample_service_usage_param1",
	      "rate": "(p, qty) => p ? p * qty : 0",
	      "charge": "(t, cost) => cost"
	    }
	  ]
	}


## <div id='30'/>#2.5.3. 과금 정책 

과금 정책이란 각 서비스에 대한 사용 단가를 정의한 JSON 형식의
오브젝트이다. 서비스 제공자는 과금 정책 스키마에 맞춰 서비스에 대한
정책을 개발한다.


#### 1.  **과금 정책 스키마**

| 항목명  |유형 | 필수| 설명|
|---------|---|----|-----|
|plan_id      |  String | O   |  API 서비스 미터링 Plan ID   |
|metrics     | Array  |  최소 하나  |  과금 정책 목록   |
|    name     | String  | O   |  과금 대상 명  |
|    price     | String  |  최소 하나  |  과금 정책 상세   |
|    country     | String  | O   | 서비스 사용 단가에 적용할 통화    |
|    price     |  String | O   | 서비스 사용 단가    |
|    title     |  String | X   | 과금 정책 제목    |


#### 2.  **과금 정책 예제**

	{
	  "plan_id": "standard",
	  "metrics": [
	    {
	      "name": "sample_service_usage_param1",
	      "prices": [
	        {
	          "country": "USA",
	          "price": 1
	        },
	        {
	          "country": "EUR",
	          "price": 0.7523
	        },
	        {
	          "country": "CAN",
	          "price": 1.06
	        }
	      ]
	    },
	    {
	      "name": " sample_service_usage_param2",
	      "prices": [
	        {
	          "country": "USA",
	          "price": 0.03
	        },
	        {
	          "country": "EUR",
	          "price": 0.0226
	        },
	        {
	          "country": "CAN",
	          "price": 0.0317
	        }
	      ]
	    }
	  ]
	}



### <div id='31'/>2.5.4.  정책 등록

정책은 2가지 방식 중 하나의 방법으로 CF-ABACUS에 등록할 수 있다.

#### **1.  js 파일을 등록하는 방식**

작성한 정책을 다음의 디렉토리에 저장한 후, CF에 CF-ABACUS를 배포 또는
재배포 한다.

-	미터링 정책의 경우

		cf-abacus/lib/plugins/provisioning/src/plans/metering

-	등급 정책의 경우

		cf-abacus/lib/plugins/provisioning/src/plans/pricing

-	과금 정책의 경우

		cf-abacus/lib/plugins/provisioning/src/plans/rating


#### **2.  DB에 등록하는 방식**

작성한 정책을 curl 등을 이용해 DB에 저장하는 방식으로 CF-ABACUS를
재배포할 필요는 없다. 정책 등록 시, 정책 ID는 고유해야 한다.

-   미터링 정책의 경우

  		POST /v1/metering/plans/:metering_plan_id


-   등급 정책의 경우

  		POST /v1/rating/plans/:rating_plan_id

-   과금 정책의 경우

  		POST /v1/pricing/plans/:pricing_plan_id


## <div id='32'/>2.6  배포
파스-타 플랫폼에 애플리케이션을 배포하면 배포한 애플리케이션과 파스-타
플랫폼이 제공하는 서비스를 연결하여 사용할 수 있다. 파스-타 플랫폼상에서
실행을 해야만 파스-타 플랫폼의 애플리케이션 환경변수에 접근하여 서비스에
접속할 수 있다.

### <div id='33'/>2.6.1파스-타 플랫폼 로그인

아래의 과정을 수행하기 위해서 파스-타 플랫폼에 로그인

  >$ cf api --skip-ssl-validation **https://api**.<***파스-타 도메인***> # **파스-타 플랫폼 TARGET 지정**

  >$ cf login -u *<**user name**>* -o *<**org name**>* -s *<**space name**>* **#로그인 요청**


### <div id='34'/>2.6.2.  mongo-db 서비스 브로커 생성

애플리케이션에서 사용할 서비스를 파스-타 플랫폼을 통하여 생성한다.
mongo-db 서비스 팩이 배포하고자 파스-타 플랫폼 환경에 release 되어
있어야 한다. 애플리케이션과 바인딩 과정을 통해 접속정보를 얻을 수 있다.

-   **서비스 생성 (cf marketplace 명령을 통해 서비스 목록과 각 서비스의
    플랜을 조회할 수 있다.)**

		## 서비스 브로커 CF 배포
		$ cd openpaas-service-java-broker-mongo
		$ cf push
		
		## 서비스 브로커 생성
		$ cf create-service-broker <서비스 브로커 명> <인증ID> <인증Password> <서비스 브로커 주소>
		
		예)
		$ cf create-service-broker openpaas-mongo-broker admin cloudfoundry http://openpaas-mongo-broker.bosh-lite.com
		
		## 서비스 브로커 확인
		$ cf service-brokers
		Getting service brokers as admin...
		
		name                url   
		openpaas-mongo-broker http://openpaas-mongo-broker.<파스-타 도메인>
		
		## 서비스 카탈로그 확인
		$ cf service-access
		Getting service access as admin...
		broker: sample-mongodb-broker
		   service                                   plan       access   orgs   
		   Mongo-DB                               default-plan none        
		   
		## 등록한 서비스 접근 허용
		$ cf enable-service-access <서비스명> -p <플랜 명>
		
		예)
		$ cf enable-service-access Mongo-DB
		
		# 서비스 생성
		$ cf create-service Mongo-DB default-plan  mongod_service


## <div id='35'/>2.6.3.  API 서비스 연동 샘플 애플리케이션 배포 및 서비스 연결

애플리케이션과 서비스를 연결하는 과정을 '바인드(bind)라고 하며, 이
과정을 통해 서비스에 접근할 수 있는 접속정보를 생성한다.

-   애플리케이션과 서비스 연결
-   이때 -c 옵션으로 미터링에 필요한 애플리케이션 환경정보를 세팅한다.

		## API 서비스 연동 샘플 애플리케이션 배포
		$ cd /binding-test-app
		$ cf push
		
		## 서비스 바인드
		$ cf bind-service <APP_NAME> <SERVICE_INSTANCE> -c <PARAMETERS_AS_JSON>
		
		예) 
		$ cf bind-service binding-test-app mongod_service -c '{"app_organization_id":"test05","app_space_id":"testspaceId","metering_plan_id":"standard"}'
		
		## 서비스 연결 확인
		$ cf services
		Getting services in org real / space ops as admin...
		OK
		
		name                       service                                   plan       bound apps               last operation   
		binding-test-app mongod_service standard   binding-test-app create succeeded
		
		## 애플리케이션 실행
		$ cf start <APP_NAME>
		
		예)
		$ cf start binding-test-app
		
		## 형상 확인
		$ cf a
		Getting apps in org real / space ops as admin...
		OK
		
		name                      requested state   instances   memory   disk   urls   
		binding-test-app          started           1/1         512M     512M   binding-test-app.<파스-타 도메인>
		openpaas-mongo-broker     started           1/1         512M     1G     openpaas-mongo-broker.<파스-타 도메인>


## <div id='36'/>2.7.  서비스 바인딩 CF-Abacus 연동 테스트

binding-test-app 과 mongo-db 서비스를 바인딩 실행해, CF-Abacus 연동
테스트를 진행 할 수 있다.

CF-Abacus 연동 확인

	## 테스트 바인딩
	$ cf bind-service binding-test-app mongod_service -c '{"app_organization_id":"testOrgGuid","app_space_id":"testSpaceGuId","metering_plan_id":"standard"}'
	
	<<후략>> 
	
	## API 사용량 확인
	$ curl 'http://abacus-usage-reporting.<파스-타 도메인>/v1/metering/organizations/<샘플 애플리케이션을 배포한 조직>/aggregated/usage'
	
	예)
	$ curl 'http://abacus-usage-reporting.bosh-lite.com/v1/metering/organizations/testOrgGuid /aggregated/usage'


## <div id='37'/>2.8.  단위 테스트

Junit 테스트로 구현 되어 있으며, 테스트 service class 에 대한 부분적
mock 적용을 위하여, owermock-mockito-release-full:1.6.1 을 사용하였다.


-   테스트를 위한 gradle.build dependency 작성


		dependencies {
		
		// 서비스브로커 라이브러리 
		compile files('libs/openpaas-service-java-broker-ex.jar')
		
		// 미터링 사용량 객체 생성 의존 라이브러리
		compile("org.json:json:20160212")
		
		…중략
		
		:${springBootCfServiceBrokerVersion}")
		
		testCompile("org.springframework.boot:spring-boot-starter-test:${springBootVersion}")
		testCompile("com.jayway.jsonpath:json-path:${jsonPathVersion}")
		testCompile("org.apache.httpcomponents:httpclient:4.4.1")
		testCompile("org.powermock:powermock-mockito-release-full:1.6.1")    
		…후략
		}


1.  테스트 실행
	
	-   Spring Tool Suite 의 네비게이터 트리의 /meteringTest 경로에서 오른쪽
			마우스 클릭 > Run As > JUNIT 테스트

## <div id='38'/>2.9. 샘플코드

샘플 코드는 아래의 사이트에 다운로드 할 수 있다.

[다운로드](https://paas-ta.kr/data/packages/2.0/PaaSTA-Metering.zip)


[Java_Service_Metering_Image01]:images/Java_Service_Metering/service_broker_api_architecture.png
[Java_Service_Metering_Image02]:images/Java_Service_Metering/service_metering_deployment_range.png
[Java_Service_Metering_Image03]:images/Java_Service_Metering/mongo-db-jar.png
[Java_Service_Metering_Image04]:images/Java_Service_Metering/service_broker_library_architecture.png
