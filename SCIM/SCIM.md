## SCIM ( System for Cross-domain Identity Management)

### 0. SCIM이 나오게된 배경
 - 기존에 유저 정보를 교환하거나 서술하는 표준들이 많으며, 이러한 것들은 어려우며 사용하기도 힘들다.
 - 그들의 wire protocols이 방화벽을 쉽게 넘어오지 못하고, 기존 웹 프로토콜에 쉽게 겹쳐지지 않는다.
 - 이러한 현상은 개발자로 하여금 중복 통합 개발을 하도록 한다.

### 1. SCIM이란?
 - 클라우드 기반 어플리케이션과 서비스에서, 사용자 정보를 관리하기 쉽도록 정의되었다.
 - SCIM은 공통 사용자 스키마와 확장 모델을 제공하고, 표준 프로토콜을 이용하여 스키마를 교환하기 위한 패턴을 제공하기 위해 문서를 바인딩 함으로서(?) 유저 관리 체제의 비용과 복잡도를 줄인다.

  ![Alt text](images/scim-works.png)



### 2. 모델
 - SCIM은 Resource라는 공통 분모에서 모든 객체가 파생된다.
 - Id, externalId, 속성값으로 meta를 가지고 있으며, RFC7643은 User, Group, EnterpriseUser를 공통 속성으로 정의한다.
 (이러한 형식을 RFC7643 프로토콜로 정의한듯)

  ![Alt text](images/scim-model.png)


### 3. User 예제
 - 어떻게 유저 정보가 SCIM JSON 객체로 인코딩 되는지 보여주는 예제이다.

 ```JAVA
 {
   "schemas": ["urn:ietf:params:scim:schemas:core:2.0:User"],
   "id":"2819c223-7f76-453a-919d-413861904646",
   "externalId":"bjensen",
   "meta":{
     "resourceType": "User",
     "created":"2011-08-01T18:29:49.793Z",
     "lastModified":"2011-08-01T18:29:49.793Z",
     "location":"https://example.com/v2/Users/2819c223...",
     "version":"W\/\"f250dd84f0671c3\""
   },
   "name":{
     "formatted": "Ms. Barbara J Jensen, III",
     "familyName": "Jensen",
     "givenName": "Barbara",
     "middleName": "Jane",
     "honorificPrefix": "Ms.",
     "honorificSuffix": "III"
   },
   "userName":"bjensen",
   "phoneNumbers":[
     {
       "value":"555-555-8377",
       "type":"work"
     }
   ],
   "emails":[
     {
       "value":"bjensen@example.com",
       "type":"work",
       "primary": true
     }
   ]
 }
 ```
 - 기본적인 String type인 id, username도 지원하며, name과 address와 같이 sub-attributes가 있는 복잡한 type, e-mail, phoneNumbers와 같은 Multivalued types도 지원한다.

### 4. Group 예제
 - SCIM은 users와 더불어 groups 정의를 지원한다.
 - 그룹은 제공된 리소스의 조직 구조를 모델링하는데 사용된다 / 그룹은 유저나 그룹을 담을 수 있다.
```JAVA
{
  "schemas": ["urn:ietf:params:scim:schemas:core:2.0:Group"],
  "id":"e9e30dba-f08f-4109-8486-d5c6a331660a",
  "displayName": "Tour Guides",
  "members":[
    {
      "value": "2819c223-7f76-453a-919d-413861904646",
      "$ref":
"https://example.com/v2/Users/2819c223-7f76-453a-919d-413861904646",
      "display": "Babs Jensen"
    },
    {
      "value": "902c246b-6245-4190-8e05-00816be7344a",
      "$ref":
"https://example.com/v2/Users/902c246b-6245-4190-8e05-00816be7344a",
      "display": "Mandy Pepperidge"
    }
  ],
  "meta": {
    "resourceType": "Group",
    "created": "2010-01-23T04:56:22Z",
    "lastModified": "2011-05-13T04:42:34Z",
    "version": "W\/\"3694e05e9dff592\"",
    "location":
"https://example.com/v2/Groups/e9e30dba-f08f-4109-8486-d5c6a331660a"
  }
}
```
 - members를 보면 value로 위 user의 id값을 가져오는 것을 보아, 이렇게 관리할 수 있는 것 같다.

### 5. Operations(작업)
 - SCIM은 리소스 조작을 위해 REST API에 특정 사용자의 특정 속성에 패치를 적용한 후, 대량 업데이트를 수행하는 모든 작업을 지원하는 풍부하면서도 간단한 작업 셋을 지원한다.

 ```JAVA
Create = POST https://example.com/{v}/{resource}
Read = GET https://example.com/{v}/{resource}/{id}
Replace = PUT https://example.com/{v}/{resource}/{id}
Delete = DELETE https://example.com/{v}/{resource}/{id}
Update = PATCH https://example.com/{v}/{resource}/{id}
Search = GET https://example.com/{v}/{resource}?ﬁlter={attribute}{op}{value}&sortBy={attributeName}&sortOrder={ascending|descending}
Bulk = POST https://example.com/{v}/Bulk
 ```

### 6. Discovery(검색)
 - 상호운용을 간단히 하게 위해, SCIM은 지원되는 기능과 특정 세부 속성을 검색하기 위한 3개의 end points를 제공한다.

 ```JAVA
 GET /ServiceProviderConfigs
 ```
 - 응답 명세(?), 인증 스키마, 데이터 모델

  ```JAVA
  GET /ResourceTypes
  ```
 - 이용가능한 자원의 type들을 찾기 위한 endpoint

  ```JAVA
  GET /Schemas
  ```
 - 자원과 속성 확장을 조사


### 7. Create Request
 - resource를 만들기 위해서, HTTP POST 요청을 resource 각자의 end point로 보낸다.
 - 아래 예제는 User 생성 예시이다.
    + URL은 version 정보를 담고있기에, 여러 버젼의 SCIM API가 공존할 수 있다.
    + 이용 가능한 버젼은 ServiceProviderConfig end point로 검색 가능하다.

 ```JAVA
 POST /v2/Users  HTTP/1.1
 Accept: application/json
 Authorization: Bearer h480djs93hd8
 Host: example.com
 Content-Length: ...
 Content-Type: application/json

 {
   "schemas":["urn:ietf:params:scim:schemas:core:2.0:User"],
   "externalId":"bjensen",
   "userName":"bjensen",
   "name":{
     "familyName":"Jensen",
     "givenName":"Barbara"
   }
 }
 ```

### 8. Create Response
 - 응답은 만들어진 자원과 성공 코드인 HTTP code 201을 포함한다.
 - POST 요청 보다 더 많은 것들이 user에 포함되어 리턴된다는 것을 참고하라. (id, meta data)
 - location 헤더는 후속 요청에서 리소스를 가져올 수 있는 위치를 나타낸다.

 ```JAVA
 HTTP/1.1 201 Created
 Content-Type: application/scim+json
 Location: https://example.com/v2/Users/2819c223-7f76-453a-919d-413861904646
 ETag: W/"e180ee84f0671b1"

 {
   "schemas":["urn:ietf:params:scim:schemas:core:2.0:User"],
   "id":"2819c223-7f76-453a-919d-413861904646",
   "externalId":"bjensen",
   "meta":{
     "resourceType":"User",
     "created":"2011-08-01T21:32:44.882Z",
     "lastModified":"2011-08-01T21:32:44.882Z",
     "location":
 "https://example.com/v2/Users/2819c223-7f76-453a-919d-413861904646",
     "version":"W\/\"e180ee84f0671b1\""
   },
   "name":{
     "familyName":"Jensen",
     "givenName":"Barbara"
   },
   "userName":"bjensen"
 }
 ```

### 9. GET request
 - HTTP GET 요청을 통해 리소스를 가져온다.

 ``` JAVA
GET /v2/Users/2819c223-7f76-453a-919d-413861904646
Host: example.com
Accept: application/scim+json
Authorization: Bearer h480djs93hd8
 ```

### 10. GET Response
 - GET 요청의 응답으로 자원을 포함한다.
 - Etag 헤더는 자원을 이후 요청에서 수정할 수 없도록 막는다. (병행성)

```JAVA
HTTP/1.1 200 OK
Content-Type: application/json
Location: https://example.com/v2/Users/2819c223-7f76-453a-919d-413861904646
Etag: W/"e180ee84f0671b1"

{
  "schemas":["urn:ietf:params:scim:schemas:core:2.0:User"],
  "id":"2819c223-7f76-453a-919d-413861904646",
  "externalId":"bjensen",
  "meta":{
    "resourceType":"User",
    "created":"2011-08-01T18:29:49.793Z",
    "lastModified":"2011-08-01T18:29:49.793Z",
    "location":
"https://example.com/v2/Users/2819c223-7f76-453a-919d-413861904646",
    "version":"W\/\"f250dd84f0671c3\""
  },
  "name":{ ...
```

### 11. Filter Request
 - Resource의 end point에, id 없이 질의를 하여서, 하나의 자원 뿐 아니라 여러 자원 set도 가지고 올 수 있다.
 - 일반적으로, fetch(검색) 요청은 자원에 적용될 filter를 포함한다. (검색에 전체 검색이 포함된다?)
 - SCIM은 equals, contains, starts with 등을 지원한다.
 - filtering을 위해 서비스 제공자에게 정렬 요청 역시 가능하다.
 - 정렬해서 특정 요소만 리턴도 가능하다.
   + https://example.com/{resource}?ﬁlter={attribute} {op} {value} & sortBy={attributeName}&sortOrder={ascending|descending}&attributes={attributes}
   + https://example.com/Users?ﬁlter=title pr and userType eq “Employee”&sortBy=title&sortOrder=ascending&attributes=title,username

### 12. Filter Response
 ```JAVA
 {
   "schemas":["urn:ietf:params:scim:api:messages:2.0:ListResponse"],
   "totalResults":2,
   "Resources":[
     {
       "id":"c3a26dd3-27a0-4dec-a2ac-ce211e105f97",
       "title":"Assistant VP",
       "userName":"bjensen"
     },
     {
       "id":"a4a25dd3-17a0-4dac-a2ac-ce211e125f57",
       "title":"VP",
       "userName":"jsmith"
     }
   ]
 }
 ```


### 99. 의문점들
 - resource를 어떻게 들고있는지? 보내는지?
 - schema는 무엇인지 : resource 전체의 컨텐츠를 정의하는, 속성들의 집합체
    + "urn:ietf:params:scim:schemas:core:2.0:User".
    + 한쪽의 요청을 받아서, 해당 스키마를 통해서, 다른쪽으로 요청을 보내는듯.
