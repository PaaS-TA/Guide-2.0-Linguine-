## Table of Contents
1. [문서개요](#1)
	* [목적](#2)
	* [범위](#3)
2. [GET org_app_usage_summary](#4)
	* [Request Parameters](#5)
	* [cURL](#6)
	* [Response body](#7)
3. [GET monthly_org_usage](#8)
	* [Request Parameters](#9)
	* [cURL](#10)
	* [Response body](#11)
4.  [GET app_monthly_usage](#12)
	* [Request Parameters](#13)
	* [cURL](#14)
	* [Response body](#15)

# <div id='1'/>1.  문서 개요

## <div id='2'/>1.1.  목적

본 문서는 PaaS-TA Usage Reporting의 API 호출에 대해 기술하였다.

## <div id='3'/>1.2.  범위 

본 문서에서는 PaaS-TA Usage Reporting의 API의 인터페이스 정보 및 호출
방법에 대해 작성되었다.


# <div id='4'/>2. GET org_app_usage_summary

해당 조회 월 1일부터 현재 시간까지의 org / space 에서 동작 중인 각 app의
메모리 사용량을 조회한다. space_id parameter 가 ‘all’ 인 경우, org
전체의 값을 리턴한다.

## <div id='5'/>2.1.  Request Parameters 

|  이름  |설명 |기본값| 유효값| 예시값|
|--------|----|-----|------|-------|
|org_id   |조직 아이디       |-|	org_guid	        |‘7726b51e-b7b4-4b9f-a1cf-78eab2710e2d’|
|space_id |	스페이스 아이디	|-|	space_guid, ‘all’	|‘b5e7f478-6f26-457f-97ce-c57a31afe157’|

## <div id='6'/>2.2.  cURL

	curl -i -X GET http://paasta-usage-reporting.your-domain.com/v1/org/:org_id/space/:space_id

## <div id='7'/>2.3.  Response body

| 프로퍼티 이름  |설명 | 유효값| 예시값|
|---------|---|----|-----|
|org_id	|조직 guid|	org_guid	| ‘7726b51e-b7b4-4b9f-a1cf-78eab2710e2d’      |
|from	|집계 시작 timestamp|	timestamp	|1470009600000      |
|end	|집계 종료 timestamp|	timestamp	|1472192759652         |
|sum	|합계|	number (소수1자리)	|6.5         |
|app_usage_array |  사용량 배열 객체|	array	|N/A |
|space_id|	스페이스 guid|	space_guid | ‘7b85dc3f-85f0-40bc-8532-0dabd0bc7bae’ |
|app_id	|app guid (CF이벤트)|	app guid  |  ‘4566f4e8-ec03-4aae-a006-3d6b12ce7a9c’ |
|app_name|	CF push 시 정의된 앱 명칭|	String | ‘java_demo’ |
|app_state|	앱의 현재 상태|	‘STARTED’,’STOPED’,’PENDDING’ |	‘STARTED’         |
|app_instance|	할당된 인스턴스(CF이벤트  기준)|	number (정수값)|	 1         |
|app_memory|	할당된 메모리(CF이벤트  기준) 단위 M |	512, 1024 | 512 |
|  app_usage|	현재까지의 집계 사용량|  number (소수1자리) | 8.5 |


	{
	    "org_id": "e655c52a-6ee5-4cf1-89d4-eac31fb6b36f",
	    "from": 1470009600000,
	    "to": 1472192759652,
	    "sum": 6,
	    "appUsage":[
	            {
	                "space_id": "b5e7f478-6f26-457f-97ce-c57a31afe157",
	                "app_id": "657c7e08-34eb-4d69-88a4-61262ce07685",
	                "app_name": "spring-crt",
	                "app_state": "STARTED",
	                "app_instance": 1,
	                "app_memory": 512,
	                "app_usage": 0.5
	            },
	            {
	                "space_id": "7b85dc3f-85f0-40bc-8532-0dabd0bc7bae",
	                "app_id": "204f83ba-7b3a-4627-9b51-168237fd1695",
	                "app_name": "node_demo",
	                "app_state": "STARTED",
	                "app_instance": 1,
	                "app_memory": 512,
	                "app_usage": 1
	            }
	    ]
	}

# <div id='8'/>3.  GET monthly_org_usage

org /space 내 동작 중인 각 app의 집계 기간내 월별 메모리 사용량을 조회한다.

space_id parameter 가 ‘all’ 인 경우, org 전체의 값을 리턴한다.


## <div id='9'/>3.1.  Request Parameters

|  이름  |설명 |기본값| 유효값| 예시값|
|--------|---|------|-------|-----|
|org_id	      |조직 아이디           |-|org_guid        |    ‘7726b51e-b7b4-4b9f-a1cf-78eab2710e2d’|
|space_id     |	스페이스 아이디	    |-|space_guid,‘all’|	‘b5e7f478-6f26-457f-97ce-c57a31afe157’|
|from_month   |	집계 시작 년월 yyyymm|-|yyyymm 형식 String |  ‘201601’|
|to_month     |	집계 종료 년월 yyyymm|-|yyyymm 형식 String | ‘201603’|

## <div id='10'/>3.2.  cURL 

	curl -i -X GET http://pasta-usage-reporting.your-domain.com/v1/org/:org_id/space/:space_id/from/:from_month/to/:to_month


## <div id='11'/>3.3.  Response body

| 프로퍼티 이름  |설명 | 유효값| 예시값|
|---------|---|----|-----|
|org_id	|조직 guid	|org_guid | ‘7726b51e-b7b4-4b9f-a1cf-78eab2710e2d’|
|from_month |	집계 시작 년월 yyyymm|	yyyymm 형식String   |‘201601’|
|to_month|	   집계 종료 년월yyyymm|	yyyymm 형식String  | ‘201603’|
|sum	  |         집계 기간 사용량 합계  |number|	6.5|
|monthly_usage_arr|  월별 사용량 배열 객체|	array|	N/A|
|month|	          집계 월|	       String|	‘201603’|
|sum|	         월 사용량 합계	|       number |    6.5|
|spaces   |        스페이스 배열 객체  |    -  |  N/A|
|space_id|	스페이스 guid	|       space_guid|	‘7b85dc3f-85f0-40bc-8532-0dabd0bc7bae’|
|sum |스페이스 사용량 합계|  - | -	|
|app_usage_arr|   앱 사용량 배열 객체 | -  |-	|
|app_id  | 앱 guid | app guid |  ‘4566f4e8-ec03-4aae-a006-3d6b12ce7a9c’|
|app_name | 앱 명칭  | string   |  ‘java_demo’|
|app_instance  |인스턴스(CF이벤트  기준)  |number|  1|
|app_memory|	메모리(CF이벤트  기준)|	512,1024|	512|
|app_usage|	현재까지의 집계 사용량|	number	|8.5|
|total_app_usage_arr|	집계 기간내 앱별 총사용량 배열 객체|   -  |  -	|	
|app_id	|앱 guid | app guid |   ‘4566f4e8-ec03-4aae-a006-3d6b12ce7a9c’|
|app_name|	앱 명칭 |  string |   ‘java_demo’|
|app_usage|	집계 기간내 앱별 총사용량	|number|	3345|

	{
	    "org_id": "e655c52a-6ee5-4cf1-89d4-eac31fb6b36f",
	    "from_month": "201601",
	    "to_month": "201602",
	    "sum": 8,
	    "monthly_usage_arr": [{
	        "month": "201601",
	        "sum": 5,
	        "spaces": [
	            {
	                "space_id": "7b85dc3f-85f0-40bc-8532-0dabd0bc7bae",
	                "sum": 2,
	                "app_usage": [
	                    {
	                        "app_id": "204f83ba-7b3a-4627-9b51-168237fd1695",
	                        "app_name": "node_demo",
	                        "app_instance": 1,
	                        "app_memory": 512,
	                        "app_usage": 1
	                    },
	                    {
	                        "app_id": "4566f4e8-ec03-4aae-a006-3d6b12ce7a9c",
	                        "app_name": "java_demo",
	                        "app_instance": 1,
	                        "app_memory": 512,
	                        "app_usage": 1
	                    }
	                ]
	            },
	            {
	                "space_id": "7b85dc3f-85f0-40bc-8532-0dabd0bc7bae",
	                "sum": 3,
	                "app_usage": [
	                    {
	                        "app_id": "204f83ba-7b3a-4627-9b51-168237fd1695",
	                        "app_name": "node_demo",
	                        "app_instance": 1,
	                        "app_memory": 512,
	                        "app_usage": 1
	                    },
	                    {
	                        "app_id": "4566f4e8-ec03-4aae-a006-3d6b12ce7a9c",
	                        "app_name": "java_demo",
	                        "app_instance": 1,
	                        "app_memory": 512,
	                        "app_usage": 2
	                    }
	                ]
	            }
	        ]
	    },
	    {
	        "month": "201602",
	        "sum": 3,
	        "spaces": [
	            {
	                "space_id": "b5e7f478-6f26-457f-97ce-c57a31afe157",
	                "sum": 0.5,
	                "app_usage": [
	                    {
	"app_id": "657c7e08-34eb-4d69-88a4-61262ce07685",
	                        "app_name": "spring-crt",
	                        "app_instance": 1,
	                        "app_memory": 512,
	                        "app_usage": 0.5
	                    }
	                ]
	            },
	            {
	                "space_id": "7b85dc3f-85f0-40bc-8532-0dabd0bc7bae",
	                "sum": 2.5,
	                "app_usage_arr": [
	                    {
	"app_id": "657c7e08-34eb-4d69-88a4-61262ce07685",
	                        "app_name": "spring-crt",
	                        "app_instance": 3,
	                        "app_memory": 512,
	                        "app_usage": 2.5
	                    }
	                ]
	            }
	        ]
	    }
	    ] ,
	    "total_app_usage_arr": [
	        {
	            "app_id": "657c7e08-34eb-4d69-88a4-61262ce0768",
	            "app_name": "spring-demo",
	            "app_usage": 223
	        },
	        {
	            "app_id": "7b85dc3f-85f0-40bc-8532-0dabd0bc7bae",
	            "app_name": "spring-crt",
	            "app_usage": 234
	        },
	        {
	            "app_id": "7b85dc3f-85f0-40bc-8532-0dabd0bc7bae",
	            "app_name": "node_demo",
	            "app_usage": 123
	        },
	        {
	            "app_id": "7b85dc3f-85f0-40bc-8532-0dabd0bc7bae",
	            "app_name": "java_demo",
	            "app_usage": 423
	        }
	    ]
	}


# <div id='12'/>4. GET app_monthly_usage

해당 app의 집계기간 내의 월별 메모리 사용량을 조회한다.

## <div id='13'/>4.1.  Request Parameters
<table>
  <tr>
    <th>이름</th>
    <th>설명</th>
    <th>기본값</th>
    <th>유효값</th>
    <th>예시값</th>
  </tr>
  <tr>
    <td>org_id</td>
    <td>조직 아이디</td>
    <td>-</td>
    <td>org_guid</td>
    <td>‘7726b51e-b7b4-4b9f-a1cf-78eab2710e2d’</td>
  </tr>
  <tr>
    <td>space_id</td>
    <td>스페이스 아이디</td>
    <td>-</td>
    <td>space_guid</td>
    <td>‘b5e7f478-6f26-457f-97ce-c57a31afe157’</td>
  </tr>
  <tr>
    <td>app_id</td>
    <td>앱 아이디</td>
    <td>-</td>
    <td>app_guid</td>
    <td>‘204f83ba-7b3a-4627-9b51-168237fd1695’</td>
  </tr>
  <tr>
    <td>from_month</td>
    <td>집계 시작 년월 yyyymm</td>
    <td>-</td>
    <td>yyyymm 형식 String</td>
    <td>‘201601’</td>
  </tr>
  <tr>
    <td>to_month</td>
    <td>집계 종료 년월 yyyymm</td>
    <td>-</td>
    <td>yyyymm 형식 String</td>
    <td>‘201603’</td>
  </tr>
</table>


## <div id='14'/>4.2.  cURL

	curl -i -X GET http://pasta-usage-reporting.your-domain.com/v1/ org/:org_id/space/:space_id/app/:app_id/from/:from_month/to/:to_month


## <div id='15'/>4.3.  Response body
<table>
  <tr>
    <th>프로퍼티 이름</th>
    <th>설명</th>
    <th>유효값</th>
    <th>예시값</th>
  </tr>
  <tr>
    <td>org_id</td>
    <td>조직 아이디</td>
    <td>org_guid</td>
    <td>‘7726b51e-b7b4-4b9f-a1cf-78eab2710e2d'</td>
  </tr>
  <tr>
    <td>space_id</td>
    <td>스페이스 아이디</td>
    <td>space_guid</td>
    <td>‘7b85dc3f-85f0-40bc-8532-0dabd0bc7bae’</td>
  </tr>
  <tr>
    <td>app_id</td>
    <td>앱 guid (CF이벤트)</td>
    <td>app_guid</td>
    <td>‘4566f4e8-ec03-4aae-a006-3d6b12ce7a9c’</td>
  </tr>
  <tr>
    <td>app_name</td>
    <td>앱 명칭</td>
    <td>string</td>
    <td>‘java_demo’</td>
  </tr>
  <tr>
    <td>from_month</td>
    <td>집계 시작 년월 yyyymm</td>
    <td>yyyymm 형식 String</td>
    <td>‘201601’</td>
  </tr>
  <tr>
    <td>to_month</td>
    <td>집계 종료 년월 yyyymm</td>
    <td>yyyymm 형식 String</td>
    <td>‘201603’</td>
  </tr>
  <tr>
    <td>sum</td>
    <td>전체 사용량 합계</td>
    <td>number</td>
    <td>5.5</td>
  </tr>
  <tr>
    <td>monthly_usage_arr</td>
    <td>월별 사용량 배열 객체</td>
    <td>array</td>
    <td>N/A</td>
  </tr>
  <tr>
    <td>month</td>
    <td>집계 월 yyyymm</td>
    <td>yyyymm 형식 String</td>
    <td>‘201601’</td>
  </tr>
  <tr>
    <td>app_instance</td>
    <td>할당된 인스턴스(CF이벤트 기준)</td>
    <td>number</td>
    <td>1</td>
  </tr>
  <tr>
    <td>app_memory</td>
    <td>할당된 메모리(CF이벤트 기준)</td>
    <td>number</td>
    <td>512</td>
  </tr>
  <tr>
    <td>app_usage</td>
    <td>현재까지의 집계 사용량</td>
    <td>number</td>
    <td>8.5</td>
  </tr>
</table>   

	{
	    "org_id": "e655c52a-6ee5-4cf1-89d4-eac31fb6b36f",
	    "space_id": "7b85dc3f-85f0-40bc-8532-0dabd0bc7bae",
	    "app_id": "e655c52a-6ee5-4cf1-89d4-eac31fb6b36f",
	    "app_name": "node_demo",
	    "from_month" : "201601",
	    "to_month" : "201608",
	    "sum": 5,
	    "monthly_usage_arr":[
	        {
	            "month" : "201601",
	            "app_instance": 1,
	            "app_memory": 512,
	            "appUsage": 5
	        },
	        {
	            "month" : "201602",
	            "app_instance": 1,
	            "app_memory": 512,
	            "appUsage": 8.5
	        }
	    ]
	}

