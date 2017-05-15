
## Table of Contents

1. [개요](#1)
	* [문서 개요](#2)
	 * [목적](#3)
	 * [범위](#4)
	 * [참고자료](#5)

1. [JAVA API 서비스 미터링 개발가이드](#6)
    * [개요](#7)
    * [개발 환경 구성](#8)
     * [CF-Abacus 설치](#9)
    * [샘플 API 서비스 개발](#10)
     * [gradle 프로젝트를 생성](#11)
     * [샘플 API 서비스 형상](#12)
     * [의존성 및 프로퍼티의 설정](#13)
     * [MeteringAuthService 클래스](#14)
     * [MeteringService 클래스](#15)
     * [SampleApiJavaServiceController 클래스](#16)
    * [API 서비스 연동 샘플 애플리케이션](#17)
    * [API 서비스 연동 샘플 애플리케이션 인터페이스 항목](#18)
    * [미터링/등급/과금 정책](#19)
     * [미터링 정책](#20)
     * [등급 정책](#21)
     * [과금 정책](#22)
     * [정책 등록](#23)
    * [배포](#24)
     * [파스-타 플랫폼 로그인](#25)
     * [API 서비스 브로커 생성](#26)
     * [API 서비스 애플리케이션 배포 및 서비스 등록](#27)
     * [API 서비스 연동 샘플 애플리케이션 배포 및 서비스 연결](#28)
    * [API 및 CF-Abacus 연동 테스트](#29)
    * [샘플 코드](#30)
	
#<div id='1'/>1. 개요

##<div id='2'/>1.1. 문서 개요

###<div id='3'/>1.1.1. 목적

본 문서(Java API 서비스 미터링 적용개발 가이드)는 파스-타 플랫폼
프로젝트의 미터링 플러그인과 Java API 미터링 서비스 애플리케이션을
연동시켜 API 서비스를 미터링 하는 방법에 대해 기술 하였다.



###<div id='4'/>1.1.2. 범위
본 문서의 범위는 파스-타 플랫폼 프로젝트의 JAVA API 서비스
애플리케이션에 대한 미터링 방법에 대한 개발과 CF-Abacus 연동에 대한
내용으로 한정되어 있다.

본 문서는 API 미터링 서비스 애플리케이션을 Java 언어로 작성 하는 것에
대해 기술 한다.

본 문서는 API 서비스 고유의 비즈니스 로직은 구현 하지 않으며, API 서비스
호출 시의 미터링을 하는 기능만 구현 한다.

본 문서에서 언급 하는 “API 서비스를 사용하는 애플리케이션”은 별도로 제공
하는 **Node.js API 미터링 개발 가이드**를 참고 하여 개발 한다.

###<div id='5'/>1.1.3. 참고 자료
-   [https://docs.cloudfoundry.org/devguide/](https://docs.cloudfoundry.org/devguide/)
-   [https://github.com/cloudfoundry-incubator/cf-abacus](https://github.com/cloudfoundry-incubator/cf-abacus)


##<div id='6'/>2. JAVA API 서비스 미터링 개발가이드
###<div id='7'/>2.1 개요


API 서비스 애플리케이션을 Java 언어로 작성 한다. API 서비스는 서비스
요청을 처리함과 동시에 API 사용 내역을 CF-ABACUS에 전송하는
애플리케이션을 작성 한다.

![Java_Api_Service_Metering_Image01]

<table border = "1px;">
  <tr>
    <th colspan ="2">기능</th>
     <th>설명</th>
  </tr>
  <tr>
     <td rowspan="4">Runtime</td>
     <td>미터링/등급/과금 정책</td>
     <td>API 서비스 제공자가 제공하는 서비스에 대한 각종 정책 정의 정보. JSON 형식으로 되었으며, 해당 정책을 CF-ABACUS에 등록하면 정책에 정의한 내용에 따라 API 사용량을 집계 한다.<br>
정책은 서비스 제공자가 정의해야 하며, JSON 스키마는 다음을 참조한다.<br>
https://github.com/cloudfoundry-incubator/cf-abacus/blob/master/doc/api.md
</td>
  </tr>
   <tr>
     <td width="160px">서비스 브로커 API</td>
     <td>Cloud Controller와 Service Broker 사이의 규약으로써 서비스 브로커 API 개발에 대해서는 다음을 참조한다.<br>
https://github.com/OpenPaaSRnD/Documents/blob/master/Development-Guide/ServicePack_develope_guide.md#11
</td>
  </tr> 
  <tr>
     <td>서비스 API</td>
     <td>서비스 제공자가 제공하는 API 서비스 기능 및 API 사용량을 CF-ABACUS에 전송하는 기능으로 구성되었다.</td>
  </tr> 
   <tr>
     <td>대시보드</td>
     <td>서비스를 제공하기 위한 인증, 서비스 모니터링 등을 위한 대시보드 기능으로 서비스 제공자가 개발해야 한다.</td>
  </tr> 
  <tr>
  	<td colspan ="2">CF-ABACUS</td>
    <td>CF-ABACUS 핵심 기능으로써 수집한 사용량 정보를 집계한다.<br>
CF-ABACUS은 CF 설치 후, CF에 마이크로 서비스 형태로 설치한다. 자세한 사항은 다음을 참조한다.<br>
https://github.com/cloudfoundry-incubator/cf-abacus
</td>
  </tr>
</table>                                              

※ 본 개발 가이드는 ***API 서비스*** 개발에 대해서만 기술하며, 다른
컴포넌트의 개발 또는 설치에 대해서 링크한 사이트를 참조한다.



##<div id='8'/>2.2 개발환경 구성


Java 애플리케이션 개발을 위해 다음과 같은 환경으로 개발환경을 구성 한다.



-   CF release: v226 이상
-   java version "1.8.0_101"
-   springBootVersion : 1.3.0.BUILD-SNAPSHOT
-   gradle 2.14
-   Spring Tool Suite 혹은 Eclipse



###<div id='9'/>2.2.1 CF-Abacus 설치
    

별도 제공하는 Abacus 설치 가이드를 참고하여 CF-Abacus를 설치한다.


##<div id='10'/>2.3 샘플 API 서비스 개발 
    
샘플 api 서비스는 서비스 요청이 있는 경우, 해당 요청에 대한 응답 처리와
api 서비스 요청에 대한 미터링 정보를 CF-ABACUS에 전송하는 처리를 한다.


###<div id='11'/>2.3.1 gradle 프로젝트를 생성

프로젝트 디렉터리를 생성하고, gradle 프로젝트로 초기화 한다


	$ mkdir sample_api_java_service // 프로젝트 디렉토리
	$ cd sample_api_java_service/
	~/sample_api_java_service $ gradle init --type java-library // gradle 초기화
	: wrapper
	: init

	BUILD SUCCESSFU

	Total time: 2.435 secs

	This build could be faster, please consider using the Gradle Daemon: 
	https://docs.gradle.org/2.14/userguide/gradle_daemon.html
  

###<div id='12'/>2.3.2 샘플 API 서비스 형상

![Java_Api_Service_Metering_Image02]

의존성 및 프로퍼티 형상 설명

| **파일**       |           **목적**          |
|---------------|-----------------------------|
|build.gradle   |애플리케이션에 필요한 의존성 정보를 기술    |
|.gitignore     |Git을 통한 형상 관리 시, 형상 관리를 할 필요가 없는 파일 또는 디렉토리를 설정한다.               |
|manifest.yml   |애플리케이션을 파스-타 플랫폼에 배포 시 적용하는 애플리케이션에 대한 환경 설정 정보 <br> 애플리케이션의 이름, 배포 경로, 인스턴스 수 등을 정의할 수 있다.        |
|gradlew        |Linux 환경에서 사용하는 gradlew 빌드 실행 파일 <br> gradle 초기화 시 자동 생성 된다.     |
|gradlew.bat    |Window 환경에서 사용하는 gradle 빌드 실행 파일 <br> gradle 초기화 시 자동 생성 된다.      |
|settings.gradle    |gradlew 실행 시 적용하는 환경 설정 파일  <br> gradle 초기화 시 자동 생성 된다.    |


Java 파일 형상 설명
  
| **파일**       |           **목적**          |
|---------------|-----------------------------|
|MeteringConfig   |애플리케이션 구동 시 metering.properties를 로드 한다    |
|MeteringAuthService     |파스-타 플랫폼 상의 UAA 서버에서 abacus-usage-collector 에 대한 접근 권한 토큰을 취득하여 리턴 한다.              |
|MeteringService   |API 서비스 사용 요청 이 SampleApiJavaServiceController 에서 처리 될 때 API 서비스 처리에 대해 미터링이 적용된 사용량 보고서를 abacus-usage-collector 에 리포팅 한다.       |
|SampleApiJavaServiceApplication        |SpringBoot이 구동할 때, SpringBoot애플리케이션에 필요한 context 객체 들을 로드 한다.   |
|SampleApiJavaServiceController   |API 서비스 사용 요청을 처리하는 REST Controller.<br> 본 샘플 애플리케이션에서는 API 서비스 고유의 비즈니스 로직은 구현 하지 않았으며, API 사용량을 abacus-collector에 전송하는 기능만 수행 한다.|
|application.properties     |SpringBoot이 구동할 때, spring 에 필요한 property    |
|metering.properties      |API 사용량을 abacus-collector 전송 시에 설정 할 property 들이 정의 되어 있다.    |

###<div id='13'/>2.3.3 의존성 및 프로퍼티의 설정


-  build.gradle

	샘플 Api 서비스 애플리케이션이 사용하는 의존성에 대해 기술한다.

  
		중략..

		dependencies {

			// https://mvnrepository.com/artifact/org.springframework/spring-test
			compile group: 'org.springframework', name: 'spring-test', version: '2.5'
		     
		    providedRuntime("org.springframework.boot:spring-boot-starter-tomcat:${springBootVersion}")
		    compile("org.springframework.boot:spring-boot-starter-web:${springBootVersion}")
		
		    // 미터링 사용량 객체 생성 dependency
		    compile("org.json:json:20160212")    // Json object 생성시
		    compile("com.sun.jersey:jersey-bundle:1.18.1") ")    // https connection 생성시
		    compile("com.googlecode.json-simple:json-simple:1.1") // json parse 
		    compile("commons-codec:commons-codec:1.5") // https connection 생성시
		}	

		후략..

  



-   manifest.yml

	앱을 CF에 배포할 때 필요한 설정 정보 및 앱 실행 환경에 필요한 설정
	정보를 기술 한다.
```yml
applications:
- name: sample-api-node-service  # 애플리케이션 이름
  memory: 512M # 애플리케이션 메모리 사이즈
  instances: 1 # 애플리케이션 인스턴스 개수
  host: sample-api-java-service
  path: ./build/libs/sample_api_java_service.jar # 배포될 애플리케이션의 위치
  env:
    SPRING_PROFILES_ACTIVE : cloud
```


-   metering.properties

	API 서비스의 사용량 정보를 abacus-collector에 전송 할 때, 필요한 설정
	정보 및 계정 정보를 기술 한다.


		# abacus usage collector RESTAPI 의 주소

		abacus.collector = https://abacus-usage-collector.<CF도메인/v1/metering/collected/usage

		# abacus usage collector 가 secured 모드 true / 아닐 경우 false

		abacus.secured = true

		# 파스-타 플랫폼의 uaa server

		uaa.server = https://uaa.<CF도메인>

		# abacus usage collector RESTAPI 계정 정보 및 사용권한 (UAA server에 미리 설정)

		uaa.client.id = <abacus.usage.read/write scope 권한을 가진 ID>

		uaa.client.secret = <abacus.usage.read/write scope 권한을 가진 ID 비밀번호>

		uaa.client.scope = abacus.usage.object-storage.write,abacus.usage.object-storage.read
  

###<div id='14'/>2.3.4 MeteringAuthService 클래스
    
UAA 서버 URL 및 계정 정보를 참조 하여, UAA token 을 취득하여 리턴 한다.
 
	public String getUaacTokenHTTPS () throws MalformedURLException {
	
		String authToken = "";
		String urlStr = authServer + "/oauth/token?grant_type=client_credentials&scope=" + encodeURIComponent(scope);
		StringBuffer sb = new StringBuffer();
	
		try {
	
			TrustManager[] trustAllCerts = new TrustManager[] { new X509TrustManager() {
				public java.security.cert.X509Certificate[] getAcceptedIssuers() {
					return null;
				}
	
				public void checkClientTrusted(X509Certificate[] certs, String authType) {
				}
	
				public void checkServerTrusted(X509Certificate[] certs, String authType) {
				}
			} };
	
			SSLContext sc = SSLContext.getInstance("SSL");
			sc.init(null, trustAllCerts, new java.security.SecureRandom());
			HttpsURLConnection.setDefaultSSLSocketFactory(sc.getSocketFactory());
			URL url = new URL(urlStr);
			HttpURLConnection conn = (HttpURLConnection) url.openConnection();
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
	
		} catch (Exception e) {
			System.out.println(e.toString());
		}
	
		return authToken;
	}


metering.properties 에서 취득한 계정 정보를 BASE64 로 인코딩 한다.

	public String getAuthKey(String id, String secret) throws Exception {	
		String authKey = ""; 	
		try {	
			String encodedConsumerKey = URLEncoder.encode(id, "UTF-8");
			String encodedConsumerSecret = URLEncoder.encode(secret, "UTF-8");
			String fullKey = encodedConsumerKey + ":" + encodedConsumerSecret;
			byte[] encodedBytes = Base64.encodeBase64(fullKey.getBytes());		
			authKey = "Basic " + new String(encodedBytes);	
		} catch (Exception e) {
			e.printStackTrace();			
			throw e;
		}	
		return authKey;
	}


UAA SEVER 에서 리턴 받은 JSON 오브젝트 에서 access_token 을 추출한다.

  
	private String parseAuthToken(String jsonStr) throws ParseException{		
		String barerStr;		
		JSONParser jsonParser = new JSONParser();
		JSONObject jsonObject = (JSONObject) jsonParser.parse(jsonStr);
		barerStr = (String) jsonObject.get("access_token");		
		return barerStr;
	}





###<div id='15'/>2.3.5 MeteringService 클래스
abacus-collector의 Auth 설정 정보에 따라, 전송 방식에 대한 분기 처리를
한다.

	public void reportUsageData(String orgId, String spaceId, String appId, String planId) throws Exception {
		JSONObject serviceUsage = buildServiceUsage(orgId, spaceId, appId, planId);
	
		if (SECURED.equals(abacusSecured)) {
			reportUsageDataHTTPS(serviceUsage);
		} else {
			reportUsageDataHTTP(serviceUsage);
		}
	}


API 사용량을 Abacus-collector에 전송하기 위해, CF 또는 인증 서버로부터
토큰을 취득하여 HTTP header에 설정하고 HTTPS Connection을 생성 한다.

	public void reportUsageDataHTTPS(JSONObject serviceUsage) throws Exception {
		StringBuffer sb = new StringBuffer();
		try {
			TrustManager[] trustAllCerts = new TrustManager[] { new X509TrustManager() {
				public java.security.cert.X509Certificate[] getAcceptedIssuers() {
					return null;
				}
	
				public void checkClientTrusted(X509Certificate[] certs, String authType) {
				}  // 인증서를 생성 한다.
	
				public void checkServerTrusted(X509Certificate[] certs, String authType) {
				}
			} };
	
			SSLContext sc = SSLContext.getInstance("SSL");
			sc.init(null, trustAllCerts, new java.security.SecureRandom());
			HttpsURLConnection.setDefaultSSLSocketFactory(sc.getSocketFactory());
	
			URL url = new URL(collectorUrl);
			HttpURLConnection conn = (HttpURLConnection) url.openConnection();
			conn.setRequestMethod("POST");
			conn.setDoInput(true);
			conn.setDoOutput(true);
			conn.setUseCaches(false);
	
			conn.setRequestProperty("Content-Type", "application/json; charset=UTF-8");
	
			String bareStr = "bearer " + meteringAuthService.getUaacTokenHTTPS();
			conn.setRequestProperty("Authorization", bareStr);
	
			byte[] out = serviceUsage.toString().getBytes(StandardCharsets.UTF_8);
	
			DataOutputStream dout = new DataOutputStream(conn.getOutputStream());
			dout.write(out);
			dout.close();
	
			InputStreamReader in = new InputStreamReader((InputStream) conn.getInputStream());
			BufferedReader br = new BufferedReader(in);
	
			String line;
			while ((line = br.readLine()) != null) {
				sb.append(line).append("\n");
			}
	
			System.out.println(sb.toString());
			System.out.println(serviceUsage + " was repoerted.");
	
			br.close();
			in.close();
			conn.disconnect();
	
		} catch (Exception e) {
			Exception se = new Exception(e);
			throw se;
		}
	}


API 사용량을 Abacus-collector에 전송하기 위한 HTTP header를 설정하고
HTTP Connection을 생성 한다.

	public void reportUsageDataHTTP(JSONObject serviceUsage) throws Exception {
	
		try {
			URL url = new URL(collectorUrl);
			URLConnection con = url.openConnection();
			HttpURLConnection http = (HttpURLConnection) con;
			http.setRequestMethod("POST"); // PUT is another valid option
			http.setDoOutput(true);
			http.setDoInput(true);
			http.setUseCaches(false);
	
			byte[] out = serviceUsage.toString().getBytes(StandardCharsets.UTF_8);
			int length = out.length;
	
			http.setFixedLengthStreamingMode(length);
			http.setRequestProperty("Content-Type", "application/json; charset=UTF-8");
			http.connect();
	
			try (OutputStream os = http.getOutputStream()) {
				os.write(out);
			}
	
		} catch (IOException e) {
			e.printStackTrace();
			throw new Exception(e);
		}
	}


Abacus-collector에 전송 할 API 서비스 사용량 JSON을 생성 한다.

	private JSONObject buildServiceUsage (String orgId, String spaceId, String appId, String planId)
			throws JSONException {
	
		LocalDateTime now = LocalDateTime.now();
		Timestamp timestamp = Timestamp.valueOf(now);
	
		JSONObject jsonObjectUsage = new JSONObject ();
	
		jsonObjectUsage.put ("start", timestamp.getTime());
		jsonObjectUsage.put ("end", timestamp.getTime());
		jsonObjectUsage.put ("organization_id", orgId);
		jsonObjectUsage.put ("space_id", spaceId);
		jsonObjectUsage.put ("consumer_id", "app:" + appId);
		jsonObjectUsage.put ("resource_id", RESOURCE_ID);
		jsonObjectUsage.put ("plan_id", planId);
		jsonObjectUsage.put ("resource_instance_id", appId);
	
		JSONArray measuredUsageArr = new JSONArray ();
		JSONObject measuredUsage1 = new JSONObject ();
		JSONObject measuredUsage2 = new JSONObject ();
		JSONObject measuredUsage3 = new JSONObject ();
	
		int quantity = 0;
	
		if (STANDARD_PLAN_ID.equals(planId)) {
			quantity = PLAN_STANDARD_QUANTITY;
		} else if (EXTRA_PLAN_ID.equals(planId)) {
			quantity = PLAN_EXTRA_QUANTITY;
		}
	
		measuredUsage1.put ("measure", MEASURE_1);
		measuredUsage1.put ("quantity", quantity);
		measuredUsageArr.put(measuredUsage1);
		measuredUsage2.put ("measure", MEASURE_2);
		measuredUsage2.put ("quantity", 1);
		measuredUsageArr.put(measuredUsage2);
		measuredUsage3.put ("measure", MEASURE_3);
		measuredUsage3.put ("quantity", 0);
		measuredUsageArr.put(measuredUsage3);
	
		jsonObjectUsage.put ("measured_usage", measuredUsageArr);
		return jsonObjectUsage;
	}	


-   API 서비스 미터링 전송 항목 (전송 리포트 JSON 상세)

 | 항목명  |유형 | 설명| 예시|
 |---------|---|----|-----|
 |  start  |  UNIX |  Timestamp  |   API처리 시작 시각  | 1396421450000 |
 |  end       |  UNIX | Timestamp   | API처리 응답 시각    | 1396421451000
 |  space_id  |   String| API를 호출한 앱의 영역 ID   | d98b5916-3c77-44b9-ac12-04456df23eae    |
 |  resource_id       | String  | API 자원 ID   | sample_api    |
 | plan_id        | String  | API 미터링 Plan ID   | basic    |
 |  resource_instance_id       | String  | API를 호출한 앱 ID   | d98b5916-3c77-44b9-ac12-04d61c7a4eae    |
 | measured_usage      | Array  |  미터링 항목  |   -  |
 | measure        |  String | 미터링 대상 명   |  api_calls   |
 | quantity        |Number   | 해당 API 요청에 대한 API 처리 횟수    | 10    |
 					
※ JSON 변환 예제

	{
	  "start": 1396421450000,
	  "end": 1396421451000,
	  "organization_id": "us-south:54257f98-83f0-4eca-ae04-9ea35277a538",
	  "space_id": "d98b5916-3c77-44b9-ac12-04456df23eae",
	  "consumer_id": "app:d98b5916-3c77-44b9-ac12-045678edabae",
	  "resource_id": "sample_api",
	  "plan_id": "basic",
	  "resource_instance_id": "d98b5916-3c77-44b9-ac12-04d61c7a4eae",
	  "measured_usage": [
	    {
	      "measure": "api_calls",
	      "quantity": 10
	    }
	  ]


###<div id='16'/>2.3.6 SampleApiJavaServiceController 클래스

서비스 사용 요청을 처리하는 REST Controller. 본 샘플 애플리케이션에서는
미터링을 하는 기능만 수행 한다.

	중략..
	@RequestMapping (value = "/plan1", method = RequestMethod.POST)
	public ResponseEntity<String> serviceAPIPlan01(@RequestBody String input) throws Exception {	
		JSONParser jsonParser = new JSONParser ();
		JSONObject jsonObject = (JSONObject) jsonParser.parse(input);	
		String orgId = (String) jsonObject.get("organization_id");
		String spaceId = (String) jsonObject.get("space_id");	
		String appId = (String) jsonObject.get("consumer_id");	
		String planId = (String) jsonObject.get("plan_id");	
		JSONObject serviceKeyOBJ = (JSONObject) jsonObject.get("credential");	
		String serviceKey = (String) serviceKeyOBJ.get("serviceKey");	
		
		if(!SERVICE_KEY.equals(serviceKey))
			return new ResponseEntity<>("credential is wrong", HttpStatus.UNAUTHORIZED);
		
		meteringService.reportUsageData(orgId, spaceId, appId, planId);		
		
		String successStr = "orgId:" + orgId + "/ spaceId:" + spaceId + "/ appId:" + appId + "/ planId:" + planId + " was reported to abacus collector.";
		
		return new ResponseEntity<>(successStr, HttpStatus.OK);
	} 
	후략..



##<div id='17'/>2.4 API 서비스 연동 샘플 애플리케이션

본 가이드에서는 API 서비스를 호출하는 애플리케이션의 개발에 대해서는
기술하지 않는다. 샘플 애플리케이션의 개발에 대해서는 **Node.js
API****미터링 개발 가이드의****Api****서비스 연동 애플리케이션 개발**을
참고 한다.

###<div id='18'/>2.4.1 API 서비스 연동 샘플 애플리케이션 인터페이스 항목

####1.  **API 서비스 엔드 포인트**

	GET|POST|PUT|DELETE <api_service_restful_api>


####2.  **API 서비스 미터링 전송 항목**

| 항목명  |유형 | 설명| 예시|
|---------|---|----|-----|
|  org_id       | String  | API 서비스를 요청한 앱의 조직 ID    | 54257f98-83f0-4eca-ae04-9ea35277a538    |
|  space_id       | String  |API 서비스를 요청한 앱의 영역 ID     |d98b5916-3c77-44b9-ac12-04456df23eae     |
|consumer_id         |String   |API 서비스를 요청한 앱 ID         |d98b5916-3c77-44b9-ac12-045678edabae     |
|instance_id         |String   |API 서비스를 요청한 앱의 자원 인스턴스 ID     |d98b5916-3c77-44b9-ac12-045678edabad     |
|plan_id         |String   |앱의 요청한 API 서비스의 plan ID    |basic     |
|credentials         |JSON   |서비스 요청에 필요한 credential 항목을 설정한다.    |credentials: {<br>key: value,<br>…<br>}     |
| inputs        |JSON   |서비스 요청에 필요한 입력 정보를 설정한다.    | inputs: {<br>key:value,<br>...<br>}    |




####3.  **API 서비스 미터링 전송 항목 예제**

	{
	  organization_id: 'd6ce3670-ab9c-4453-b993-f2821f54846b',
	  space_id: 'ab63eaed-7932-4f24-804d-dccb40a68752',
	  consumer_id: 'ff7476f9-f5b6-420c-96f0-ac39be43de8c',
	  instance_id: 'ff7476f9-f5b6-420c-96f0-ac39be43de8c',
	  plan_id: 'standard',
	  credential: {
		'serviceKey': '[cloudfoundry]',
		'url': 'http://localhost:9602/plan1'
	  },
	  inputs: {
		key1: 'val1',
		key2: 'val2'
	  }
	}



##<div id='19'/>2.5. 미터링/등급/과금 정책

서비스, 그리고 서비스 제공자 마다 미터링/등급/과금 정책 다르기 때문에 본
가이드에서는 정책의 개발 예제를 다루지는 않는다. 다만 CF-ABACUS에 적용할
수 있는 형식에 대해 설명한다.


###<div id='20'/>2.5.1. 미터링 정책

미터링 정책이란 수집한 미터링 정보에서 미터링 대상의 지정 및 집계 방식을
정의한 JSON 형식의 오브젝트이다. 서비스 제공자는 미터링 정책 스키마에
맞춰 서비스에 대한 정책을 개발한다.


####1.  **미터링 정책 스키마**

| 항목명  |유형 | 필수| 예시|
|---------|---|----|-----|
|plan_id         |String   | O   |API 미터링 Plan ID     |
|measures         |Array   |최소 하나    |API 미터링 정보 수집 대상 정의     |
|  name         |String   |O    |미터링 정보 수집 대상 명     |
|  unit         |String   |O    |미터링 정보 수집 대상 단위     |
|metrics        |Array   |최소 하나    |API 미터링 집계 방식 정의     |
|  name         |String   |O    |미터링 정보 수집 대상 명     |
|  unit         |String   |O    |미터링 정보 수집 대상 단위     |
|  meter         |String   |X    |미터링 정보에 대해서 수집 단계에 적용하는 계산식 또는 변환식     |
|  accumulate         |String   |X    |미터링 정보에 대해서 누적 단계에 적용하는 계산식 또는 변환식     |
|  aggregate         |String   |X   |미터링 정보에 대해서 집계 단계에 적용하는 계산식 또는 변환식     |
|  summarize         |String   |X   |미터링 정보를 보고할 때 적용하는 계산식 또는 변환식     |
|  title         |String   |X   |API 미터링 제목     |


####2.  **미터링 정책 예제**

	{
	  "plan_id": "basic-object-storage",
	  "measures": [
	    {
	      "name": "storage",
	      "unit": "BYTE"
	    },
	    {
	      "name": "api_calls",
	      "units": "CALL"
	    }
	  ],
	  "metrics": [
	    {
	      "name": "storage",
	      "unit": "GIGABYTE",
	      "meter": "(m) => m.storage / 1073741824",
	      "accumulate": "(a, qty) => Math.max(a, qty)"
	    },
	    {
	      "name": "thousand_api_calls",
	      "unit": "THOUSAND_CALLS",
	      "meter": "(m) => m.light_api_calls / 1000",
	      "accumulate": "(a, qty) => a ? a + qty : qty",
	      "aggregate": "(a, qty) => a ? a + qty : qty",
	      "summarize": "(t, qty) => qty"
	    }
	  ]
	}


###<div id='21'/>2.5.2. 등급 정책

등급 정책이란 각 서비스의 사용 가중치를 정의한 JSON 형식의 오브젝트이다.
서비스 제공자는 등급 정책 스키마에 맞춰 서비스에 대한 정책을 개발한다.

####1.  **등급 정책 스키마**

| 항목명  |유형 | 필수| 설명|
|---------|---|----|-----|
| plan_id|   String |  O        |   API 등급 Plan ID    |
| metrics |   Array  |  최소 하나 |  등급 정책 목록	|
| name    |   String |  O        |   등급 정의 대상 명|
| rate    |   String |  X        |   가중치 계산식 또는 변환식|
| charge  |   String |  X        |   사용량에 대한 과금 계산식 또는 변환식|
| title   |   String |  X        |   등급 정책 명|


####2.  **등급 정책 예제**

	{
	  "plan_id": "object-rating-plan",
	  "metrics": [
	    {
	      "name": "storage"
	    },
	    {
	      "name": "thousand_api_calls",
	      "rate": "(p, qty) => p ? p * qty : 0",
	      "charge": "(t, cost) => cost"
	    }
	  ]
	}


###<div id='22'/>2.5.3. 과금 정책

과금 정책이란 각 서비스에 대한 사용 단가를 정의한 JSON 형식의
오브젝트이다. 서비스 제공자는 과금 정책 스키마에 맞춰 서비스에 대한
정책을 개발한다.

####1.  **과금 정책 스키마**

| 항목명  |유형 | 필수| 설명|
|---------|---|----|-----|
| plan_id|   String |  O        |   API 과금 Plan ID|
| metrics  |  Array   | 최소 하나 |  과금 정책 목록|
| name     |  String  | O        |   과금 대상 명|
| price    |  Array   | 최소 하나 |  과금 정책 상세|
| country  |  String  | O        |   서비스 사용 단가에 적용할 통화|
| price    |  Number  | O        |   서비스 사용 단가|
| title    |  String  | X        |   과금 정책 제목|


####2.  **과금 정책 예제**
		
	{
	  "plan_id": "object-pricing-basic",
	  "metrics": [
	    {
	      "name": "storage",
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
	      "name": "thousand_api_calls",
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



###<div id='23'/>2.5.4. 정책 등록
정책은 2가지 방식 중 하나의 방법으로 CF-ABACUS에 등록할 수 있다.

####1.  **js 파일을 등록하는 방식**

작성한 정책을 다음의 디렉토리에 저장한 후, CF에 CF-ABACUS를 배포 또는 재
배포 한다.

-   미터링 정책의 경우
		
		cf-abacus/lib/plugins/provisioning/src/plans/metering

-   등급 정책의 경우

		cf-abacus/lib/plugins/provisioning/src/plans/pricing

-   과금 정책의 경우

		cf-abacus/lib/plugins/provisioning/src/plans/rating


####2.  **DB에 등록하는 방식**

작성한 정책을 curl 등을 이용해 DB에 저장하는 방식으로 CF-ABACUS를
재배포할 필요는 없다. 정책 등록 시, 정책 ID는 고유해야 한다.


-   미터링 정책의 경우

		POST /v1/metering/plans/:metering_plan_id
	>
	
		## 예제
		$ curl -k -X POST 'http://abacus-provisioning-plugin.bosh-lite.com/v1/metering/plans/sample-linux-container' \
			 -H "Content-Type: application/json" \
			 -d '{"plan_id":"sample-linux-container","measures":[{"name":"current_instance_memory","unit":"GIGABYTE"},{"name":"current_running_instances","unit":"NUMBER"},{"name":"previous_instance_memory","unit":"GIGABYTE"},{"name":"previous_running_instances","unit":"NUMBER"}],"metrics":[{"name":"memory","unit":"GIGABYTE","type":"time-based","meter":"((m)=>({previous_consuming:newBigNumber(m.previous_instance_memory||0).div(1073741824).mul(m.previous_running_instances||0).mul(-1).toNumber(),consuming:newBigNumber(m.current_instance_memory||0).div(1073741824).mul(m.current_running_instances||0).toNumber()})).toString()","accumulate":"((a,qty,start,end,from,to,twCell)=>{if(end<from||end>=to)returnnull;constpast=from-start;constfuture=to-start;consttd=past+future;return{consuming:a&&a.since>start?a.consuming:qty.consuming,consumed:newBigNumber(qty.consuming).mul(td).add(newBigNumber(qty.previous_consuming).mul(td)).add(a?a.consumed:0).toNumber(),since:a&&a.since>start?a.since:start};}).toString()","aggregate":"((a,prev,curr,aggTwCell,accTwCell)=>{if(!curr)returna;constconsuming=newBigNumber(curr.consuming).sub(prev?prev.consuming:0);constconsumed=newBigNumber(curr.consumed).sub(prev?prev.consumed:0);return{consuming:consuming.add(a?a.consuming:0).toNumber(),consumed:consumed.add(a?a.consumed:0).toNumber()};}).toString()","summarize":"((t,qty,from,to)=>{if(!qty)return0;constrt=Math.min(t,to?to:t);constpast=from-rt;constfuture=to-rt;consttd=past+future;constconsumed=newBigNumber(qty.consuming).mul(-1).mul(td).toNumber();returnnewBigNumber(qty.consumed).add(consumed).div(2).div(3600000).toNumber();}).toString()"}]}' \
			-H "Authorization: $(cf oauth-token | grep bearer)"



-   등급 정책의 경우

	 	POST /v1/rating/plans/:rating_plan_id
	>

		## 예제
		$ curl -k -X POST 'http://abacus-provisioning-plugin.bosh-lite.com/v1/rating/plans/linux-rating-sample' \
			 -H "Content-Type: application/json" \
		     -d '{"plan_id":"linux-rating-sample","metrics":[{"name":"memory","rate":"((price,qty)=>({price:price,consuming:qty.consuming,consumed:qty.consumed})).toString(),charge:((t,qty,from,to)=>{if(!qty)return0;constrt=Math.min(t,to?to:t);constpast=from-rt;constfuture=to-rt;consttd=past+future;constconsumed=newBigNumber(qty.consuming).mul(-1).mul(td).toNumber();constgbhour=newBigNumber(qty.consumed).add(consumed).div(2).div(3600000).toNumber();returnnewBigNumber(gbhour).mul(qty.price).toNumber();}).toString()"}]}' \
			 -H "Authorization: $(cf oauth-token | grep bearer)"



-   과금 정책의 경우

		POST /v1/pricing/plans/:pricing_plan_id
	>

		## 예제
		$ curl -k -X POST 'http://abacus-provisioning-plugin.bosh-lite.com/v1/pricing/plans/linux-pricing-sample' \
			 -H "Content-Type: application/json" \
			 -d '{"plan_id":"linux-pricing-sample","metrics":[{"name":"memory","prices":[{"country":"USA","price":0.00014}]}]}' \
			-H "Authorization: $(cf oauth-token | grep bearer)"



##<div id='24'/>2.6. 배포

파스-타 플랫폼에 애플리케이션을 배포하면 배포한 애플리케이션과 파스-타
플랫폼이 제공하는 서비스를 연결하여 사용할 수 있다. 파스-타 플랫폼상에서
실행을 해야만 파스-타 플랫폼의 애플리케이션 환경변수에 접근하여 서비스에
접속할 수 있다.


###<div id='25'/>2.6.1 파스-타 플랫폼 로그인

아래의 과정을 수행하기 위해서 파스-타 플랫폼에 로그인
	
`$ cf api --skip-ssl-validation`*`https://api.`**`<파스-타 도메인> `**`#파스-타 플랫폼TARGET지정`*

`$ cf login -u <user name> -o <org name> -s <space name>#로그인 요청`


###<div id='26'/>2.6.2. API 서비스 브로커 생성

애플리케이션에서 사용할 서비스를 파스-타 플랫폼을 통하여 생성한다.
별도의 서비스 설치과정 없이 생성할 수 있으며, 애플리케이션과
바인딩과정을 통해 접속정보를 얻을 수 있다.

-   서비스 생성 (cf marketplace 명령을 통해 서비스 목록과 각 서비스의
    플랜을 조회할 수 있다.)

		##서비스 브로커 CF 배포
		$ cd <샘플 서비스 브로커 경로>/sample_api_java_broker
		$ cf push


  		##서비스 브로커 생성
  		$ cf create-service-broker <서비스 브로커 명> <인증ID> <인증Password> <서비스 브로커 주소>

  		예)
  		$ cf create-service-broker sample-api-broker admin cloudfoundry http://sample-api-java-broker.bosh-lite.com

  		##서비스 브로커 확인
  		$ cf service-brokers
  		Getting service brokers as admin...

 		name url
  		sample-api-broker http://sample-api-java-broker.bosh-lite.com

  		##서비스 카탈로그 확인
  		$ cf service-access
  		Getting service access as admin...
  		broker: sample-api-broker
  		service plan access orgs
  		standard_obejct_storage_light_api_calls standard none
  		standard_obejct_storage_heavy_api_calls basic none

  		##등록한 서비스 접근 허용
  		$ cf enable-service-access <서비스명> -p <플랜 명>

		  예)
  		$ cf enable-service-access standard_obejct_storage_light_api_calls -p standard


###<div id='27'/>2.6.3. API 서비스 애플리케이션 배포 및 서비스 등록

API 서비스 애플리케이션을 파스-타 플랫폼에 배포한다. 서비스 등록한 API는
다른 애플리케이션과 바인다 하여 API 서비스를 할 수 있다.

1.  **애플리케이션 배포**

	-	gradle build -x test 명령으로 빌드 한다.

	-	cf push 명령으로 배포한다. 별도의 값을 넣지않으면 manifest.yml의 설정을 사용한다

			## API 서비스 배포
			$ cd <샘플api서비스 경로>/sample_api_java_service
  			## gradle 빌드
  			$ gradle build -x test
  			:compileJava
  			:processResources
  			:classes
  			:findMainClass
  			:jar
  			:bootRepackage
  			:assemble
  			:check
	  		:build
 
			BUILD SUCCESSFUL

			Total time: 13.426 secs
	  		$ cf push
	
			##서비스 생성
	  		$ cf create-service <서비스명> <플랜 명> <서비스 인스턴스 명>
	  		예)
	  		$ cf create-service standard_obejct_storage_light_api_calls standard sampleNodejslightCallApi
	
	  		##서비스 확인
	  		$ cf services
	  		Getting services in org real / space ops as admin...
	  		OK

			name service plan bound apps last operation
	  		sampleNodejslightCallApi standard_obejct_storage_light_api_calls standard create succeeded


###<div id='28'/>2.6.4. API 서비스 연동 샘플 애플리케이션 배포 및 서비스 연결

애플리케이션과 서비스를 연결하는 과정을 '바인드(bind)라고 하며, 이
과정을 통해 서비스에 접근할 수 있는 접속정보를 생성한다.

-   애플리케이션과 서비스 연결

		## API 서비스 연동 샘플 애플리케이션 배포
		$ cd <샘플 애플리케이션 경로>/sample_api_node_caller
		$ npm install && npm run babel && npm run cfpack && ./cfpush.sh
		
		## 서비스 바인드
		$ cf bind-service <APP_NAME> <SERVICE_INSTANCE> -c <PARAMETERS_AS_JSON>
		
		예) 
		$ cf bind-service sample-api-node-caller sampleNodejslightCallApi -c '{"serviceKey": "cloudfoundry"}'
		
		## 서비스 연결 확인
		$ cf services
		Getting services in org real / space ops as admin...
		OK
		
		name                       service                                   plan       bound apps               last operation   
		sampleNodejslightCallApi   standard_obejct_storage_light_api_calls   standard   sample-api-node-caller   create succeeded
		
		## 애플리케이션 실행
		$ cf start <APP_NAME>
		
		예)
		$ cf start sample-api-node-caller
		
		## 형상 확인
		$ cf a
		Getting apps in org real / space ops as admin...
		OK
		
		name                      requested state   instances   memory   disk   urls   
		sample-api-node-service   started           1/1         512M     512M   sample-api-node-service.bosh-lite.com   
		sample-api-java-broker    started           1/1         512M     1G     sample-api-java-broker.bosh-lite.com   
		sample-api-node-caller    started           1/1         512M     512M   sample-api-node-caller.bosh-lite.com



##<div id='29'/>2.7. API 및 CF-Abacus 연동 테스트

API 연동 샘플 애플리케이션의 url을 통해 웹 브라우저에서 접속하면 API
연동 및 API 사용량에 대한 CF-Abacus 연동 테스트를 진행 할 수 있다.

1.  **CF-Abacus 연동 확인**


		## 조직 guid 확인
		$ cf org <샘플 애플리케이션을 배포한 조직> --guid
		
		예)
		$ cf org real --guid
		877d01b2-d177-4209-95b0-00de794d9bba
		
		## 샘플 애플리케이션 guid 확인
		$ cf env <샘플 애플리케이션 명>
		예)
		$ cf env sample-api-node-caller
		Getting env variables for app sample-api-node-caller in org real / space ops as admin...
		OK
		
		<<중략>>
		{
		 "VCAP_APPLICATION": {
		  "application_id": "58872d8a-edfc-44df-97f0-df67cf9033a7",
		  "application_name": "sample-api-node-caller",
		  "application_uris": [
		   "sample-api-node-caller.bosh-lite.com"
		  ],
		  "application_version": "55678102-584c-4fca-8304-82f727506b1d",
		  "limits": {
		   "disk": 512,
		   "fds": 16384,
		   "mem": 512
		  },
		  "name": "sample-api-node-caller",
		  "space_id": "2ce08996-f463-406c-a971-adbbaf4e4ca5",
		  "space_name": "ops",
		  "uris": [
		   "sample-api-node-caller.bosh-lite.com"
		  ],
		  "users": null,
		  "version": "55678102-584c-4fca-8304-82f727506b1d"
		 }
		}
		
		<<후략>> 
		
		## API 사용량 확인
		$ curl 'http://abacus-usage-reporting.<파스-타 도메인>/v1/metering/organizations/<샘플 애플리케이션을 배포한 조직>/aggregated/usage'
		
		예)
		$ curl 'http://abacus-usage-reporting.bosh-lite.com/v1/metering/organizations/877d01b2-d177-4209-95b0-00de794d9bba/aggregated/usage'

##<div id='30'/>2.8. 샘플코드

샘플코드는 아래의 사이트에 다운로드 할 수 있다.

[다운로드](https://paas-ta.kr/data/packages/2.0/PaaSTA-Metering.zip)

[Java_Api_Service_Metering_Image01]:images/Java_Api_Service_Metering/meteringAPI_development_range.png
[Java_Api_Service_Metering_Image02]:images/Java_Api_Service_Metering/sampleAPI_Service.png
