# Hands-On Essentials - data warehouse 
- https://learn.snowflake.com/en/courses/uni-essdww101/
- 해당 랩에서는 다음과 같은 것을 배웁니다.
  - Snowflake의 아키텍처
  - 데이터 웨어하우스, 역할, 스키마 개념
  - 파일 형식
  - 스테이지 생성 및 스테이지에서 데이터 가져오기
  - 데이터베이스와 테이블에 대한 개념 - 관계, 정규화, 구조화 및 비구조화 데이터 가져오기
  - XML, JSON 데이터 읽기 및 쿼리하기
  - 중첩 데이터 읽기에 대한 개념

## Role
- ![image](https://user-images.githubusercontent.com/47103479/196945830-ef1e06f2-9faa-415a-92fb-8836ab193b4d.png)
- ![image](https://github.com/mjs1995/Book_review/assets/47103479/adda0163-8c02-4a45-adf6-1c9bbbd01535)
- ACCOUNTADMIN
  - Snowflake에서 최상위 레벨의 역할입니다. 
  - 모든 Snowflake 객체와 모든 데이터를 관리할 수 있으며 모든 권한이 있습니다.
- ORGADMIN - 조직 관리자
  - 계정 내의 조직을 관리할 수 있습니다.
  - ORGADMIN은 새로운 Snowflake 계정을 생성하고 계정 속성을 볼 수 있습니다. 그러나 ORGADMIN은 기본적으로 계정 데이터에 액세스 할 수 없습니다.  
- PUBLIC 
  - Snowflake에서 미리 정의된 역할로, 권한을 부여하지 않아도 모든 사용자가 가지게 되는 기본 역할입니다. 모든 사용자가 공통으로 사용할 수 있는 테이블이나 뷰 등의 객체를 생성하려면 이 역할에 대한 권한이 필요합니다.
  - PUBLIC 역할은 Snowflake 계정의 모든 역할과 사용자에게 자동으로 부여됩니다. 다른 역할과 마찬가지로 PUBLIC 역할은 보안 가능한 객체를 소유할 수 있습니다.
  - PUBLIC 역할에 제공된 권한 또는 객체 접근은 계정의 모든 역할과 사용자에게 사용 가능합니다.
- SECURITYADMIN - 보안 관리자
  - 보안 관리자 역할입니다. 보안과 관련된 설정과 권한을 관리할 수 있습니다.
  - MANAGE GRANTS 권한을 기본적으로 갖고 있으며 USERADMIN 역할의 모든 권한을 상속합니다.
- SYSADMIN - 시스템 관리자
  - Snowflake 계정에서 가상 데이터 웨어하우스, 데이터베이스 및 기타 객체를 생성할 수 있는 권한을 갖는 시스템 정의 역할입니다. 사용자 관리자가 생성하는 가장 일반적인 역할은 SYSADMIN 역할에 할당되어 사용자 지정 역할이 생성한 객체를 관리할 수 있도록 합니다.
- USERADMIN - 사용자 관리자
  - 사용자 관리자는 사용자와 역할을 생성하고 관리하는 책임이 있습니다.

## SHOW SCHEMAS
- ![image](https://user-images.githubusercontent.com/47103479/196950570-5c4ebeaa-79f6-4c1e-9dcd-267a8fefa15f.png)
- ```sql
  SHOW SCHEMAS;
  ```
- SHOW SCHEMAS 명령을 사용하면 새로운 스키마의 보존 기간이 생성된 데이터베이스로 보존 기간이 1일인 것을 확인할 수 있습니다.

## 임시 데이터베이스에서 테이블을 생성
- ![image](https://user-images.githubusercontent.com/47103479/197323610-3e43e780-ca5d-4c7a-9596-9388756b71c4.png)
- ```sql
  use role sysadmin;
  create or replace TABLE ROOT_DEPTH (
  ROOT_DEPTH_ID NUMBER(1,0),
  ROOT_DEPTH_COED VARCHAR(1),
  ROOT_DEPTH_NAME VARCHAR(7),
  UNIT_OF_MEASURE VARCHAR(2),
  RANGE_MIN NUMBER(2,0),
  RANGE_MAX NUMBER(2,0)
  );
  ```
- 스노우플레이크는 영구 데이터베이스(permanent databases)에 대해서는 혼합된 접근 방식(mixed approach)을 사용하며 다른 종류의 객체를 저장할 수 있습니다. 
- 영구 데이터베이스 내에 임시 테이블(transient tables)을 저장할 수 있지만, 임시 데이터베이스(transient databases)에 영구 테이블(permanent tables)을 저장할 수는 없습니다. 임시 테이블은 영구 테이블보다 보호 및 복구 수준이 낮지만 세션(session)을 초과하여 유지해야 하는 단기 데이터(transitory data)를 저장하는 데 사용됩니다.

## 클라우드 스토리지 테이블 
- ![image](https://user-images.githubusercontent.com/47103479/197337415-5c469598-b667-495a-b42a-d6eb5eba1bfe.png)
- S3 버킷 내부의 파일 및 디렉토리 목록을 반환합니다.
- ```sql
  list @LIKE_A_WINDOW_INTO_AN_S3_BUCKET;
  ```
- ![image](https://user-images.githubusercontent.com/47103479/197378487-e7be55f1-9816-4c0d-9c26-3e99ff027a8b.png)
- S3 버킷 내부의 "VEG_NAME_TO_SOIL_TYPE_PIPE.txt" 파일을 읽어와서 "vegetable_details_soil_type" 테이블에 데이터를 복사합니다. 파일의 포맷은 "PIPECOLSEP_ONEHEADROW"로 지정합니다.
- ```sql
  create or replace table vegetable_details_soil_type
  ( plant_name varchar(25)
  ,soil_type number(1,0)
  );

  copy into vegetable_details_soil_type
  from @like_a_window_into_an_s3_bucket
  files = ( 'VEG_NAME_TO_SOIL_TYPE_PIPE.txt')
  file_format = ( format_name=PIPECOLSEP_ONEHEADROW );
  ```
- ![image](https://user-images.githubusercontent.com/47103479/197378476-ad5075a8-0740-4339-8174-235eaefbf43e.png)

## Semi-Structured Data
- 반정형 데이터로 Parquet, Avro, JSON, ORC, XML 파일을 지원합니다.
- XML 데이터 테이블
  - ```sql
    select * from LIBRARY_CARD_CATALOG.PUBLIC.AUTHOR_INGEST_XML;
    ```
  - ![image](https://user-images.githubusercontent.com/47103479/197380647-c023384f-e33e-401d-a392-f40f1015057f.png)
  - ![image](https://user-images.githubusercontent.com/47103479/197380699-b04a4092-82c5-4854-a377-7c411a6f59ef.png)
  - object_col에서 "level2" 요소를 가진 레코드들을 찾고 해당 레코드들의 "level2" 요소의 "$" 속성값을 가져와서 JSON 배열 형태로 나열합니다.
  - ```sql
    SELECT object_col,
      GET(XMLGET(object_col, 'level2'), '$')
    FROM xml_demo;
    +-------------------------+----------------------------------------+
    | OBJECT_COL              | GET(XMLGET(OBJECT_COL, 'LEVEL2'), '$') |
    |-------------------------+----------------------------------------|
    | <level1>                | [                                      |
    |   1                     |   2,                                   |
    |   <level2>              |   {                                    |
    |     2                   |     "$": "3A",                         |
    |     <level3>3A</level3> |     "@": "level3"                      |
    |     <level3>3B</level3> |   },                                   |
    |   </level2>             |   {                                    |
    | </level1>               |     "$": "3B",                         |
    |                         |     "@": "level3"                      |
    |                         |   }                                    |
    |                         | ]                                      |
    +-------------------------+----------------------------------------+
    ```
- JSON 데이터 테이블
-  JSON 객체의 각 속성값을 하나의 행으로 출력하고, 속성의 이름을 열의 이름으로 사용하여 문자열 형태의 새로운 정규화된 테이블을 만듭니다.
- ```sql
  select 
  raw_author:AUTHOR_UID
  ,raw_autohr:FIRST_NAME::STRING as FIRST_NAME
  ,raw_autohr:MIDDLE_NAME::STRING as MIDDLE_NAME
  ,raw_autohr:LAST_NAME::STRING as LAST_NAME
  from AUTHOR_INGEST_JSON;
  ```
- ![image](https://user-images.githubusercontent.com/47103479/197381285-e65ed102-6e9c-4b0c-8c1c-c97dbd6f5318.png)

## Nested Semi-Structured Data
- Nested Semi-Structured Data는 트리와 같은 계층 구조를 가지면서, 데이터의 스키마가 고정되어 있지 않은 데이터 형식으로 대부분 XML 또는 JSON과 같은 형식으로 저장됩니다. 
- 관련된 샘플 데이터입니다. [Nested JSON DATA](https://learn.snowflake.com/assets/courseware/v1/1f53ea71147d46fd6b2a54195c64e9c4/asset-v1:snowflake+ESS-DWW-KR+A+type@asset+block/nutrition_tweets.json)
- 배열 평면화
  - ```sql
    create or replace table events as
    select
      src:device_type::string                             as device_type
    , src:version::string                                 as version
    , value:f::number                                     as f
    , value:rv::variant                                   as rv
    , value:t::number                                     as t
    , value:v.ACHZ::number                                as achz
    , value:v.ACV::number                                 as acv
    , value:v.DCA::number                                 as dca
    , value:v.DCV::number                                 as dcv
    , value:v.ENJR::number                                as enjr
    , value:v.ERRS::number                                as errs
    , value:v.MXEC::number                                as mxec
    , value:v.TMPI::number                                as tmpi
    , value:vd::number                                    as vd
    , value:z::number                                     as z
    from
      raw_source
    , lateral flatten ( input => SRC:events );

    SELECT * FROM EVENTS;

    +-------------+---------+---------+----------------------------------------------------------------------+---------------+-------+--------+-----+-------+------+------+------+------+-----+---------------+
    | DEVICE_TYPE | VERSION |       F | RV                                                                   |             T |  ACHZ |    ACV | DCA |   DCV | ENJR | ERRS | MXEC | TMPI |  VD |             Z |
    |-------------+---------+---------+----------------------------------------------------------------------+---------------+-------+--------+-----+-------+------+------+------+------+-----+---------------|
    | server      | 2.6     |      83 | "15219.64,783.63,48674.48,84679.52,27499.78,2178.83,0.42,74900.19"   | 1437560931139 | 42869 | 709489 | 232 | 62287 | 2599 |  205 |  487 |    9 |  54 | 1437644222811 |
    | server      | 2.6     | 1000083 | "8070.52,54470.71,85331.27,9.10,70825.85,65191.82,46564.53,29422.22" | 1437036965027 |  6953 | 346795 | 250 | 46066 | 9033 |  615 |    0 |  112 | 626 | 1437660796958 |
    +-------------+---------+---------+----------------------------------------------------------------------+---------------+-------+--------+-----+-------+------+------+------+------+-----+---------------+
    ```
- ```sql
  SELECT src:device_type::string,
    src:version::String,
    VALUE
  FROM
      raw_source,
      LATERAL FLATTEN( INPUT => SRC:events );
  
    +-------------------------+---------------------+-------------------------------------------------------------------------------+
  | SRC:DEVICE_TYPE::STRING | SRC:VERSION::STRING | VALUE                                                                         |
  |-------------------------+---------------------+-------------------------------------------------------------------------------|
  | server                  | 2.6                 | {                                                                             |
  |                         |                     |   "f": 83,                                                                    |
  |                         |                     |   "rv": "15219.64,783.63,48674.48,84679.52,27499.78,2178.83,0.42,74900.19",   |
  |                         |                     |   "t": 1437560931139,                                                         |
  |                         |                     |   "v": {                                                                      |
  |                         |                     |     "ACHZ": 42869,                                                            |
  |                         |                     |     "ACV": 709489,                                                            |
  |                         |                     |     "DCA": 232,                                                               |
  |                         |                     |     "DCV": 62287,                                                             |
  |                         |                     |     "ENJR": 2599,                                                             |
  |                         |                     |     "ERRS": 205,                                                              |
  |                         |                     |     "MXEC": 487,                                                              |
  |                         |                     |     "TMPI": 9                                                                 |
  |                         |                     |   },                                                                          |
  |                         |                     |   "vd": 54,                                                                   |
  |                         |                     |   "z": 1437644222811                                                          |
  |                         |                     | }                                                                             |
  +-------------------------+---------------------+-------------------------------------------------------------------------------+
  ```
- 배열 내에 중첩된 배열을 분해
  - ```sql
     create or replace table persons as
    select column1 as id, parse_json(column2) as c
    from values
      (12712555,
      '{ name:  { first: "John", last: "Smith"},
        contact: [
        { business:[
          { type: "phone", content:"555-1234" },
          { type: "email", content:"j.smith@company.com" } ] } ] }'),
      (98127771,
      '{ name:  { first: "Jane", last: "Doe"},
        contact: [
        { business:[
          { type: "phone", content:"555-1236" },
          { type: "email", content:"j.doe@company.com" } ] } ] }') v;

    -- 각각의 LATERAL 뷰는 이전 뷰를 기반으로 하여 배열의 여러 수준에서 요소를 참조합니다.

    SELECT id as "ID",
      f.value AS "Contact",
      f1.value:type AS "Type",
      f1.value:content AS "Details"
    FROM persons p,
      lateral flatten(input => p.c, path => 'contact') f,
      lateral flatten(input => f.value:business) f1;

    +----------+-----------------------------------------+---------+-----------------------+
    |       ID | Contact                                 | Type    | Details               |
    |----------+-----------------------------------------+---------+-----------------------|
    | 12712555 | {                                       | "phone" | "555-1234"            |
    |          |   "business": [                         |         |                       |
    |          |     {                                   |         |                       |
    |          |       "content": "555-1234",            |         |                       |
    |          |       "type": "phone"                   |         |                       |
    |          |     },                                  |         |                       |
    |          |     {                                   |         |                       |
    |          |       "content": "j.smith@company.com", |         |                       |
    |          |       "type": "email"                   |         |                       |
    |          |     }                                   |         |                       |
    |          |   ]                                     |         |                       |
    |          | }                                       |         |                       |
    | 12712555 | {                                       | "email" | "j.smith@company.com" |
    |          |   "business": [                         |         |                       |
    |          |     {                                   |         |                       |
    |          |       "content": "555-1234",            |         |                       |
    |          |       "type": "phone"                   |         |                       |
    |          |     },                                  |         |                       |
    |          |     {                                   |         |                       |
    |          |       "content": "j.smith@company.com", |         |                       |
    |          |       "type": "email"                   |         |                       |
    |          |     }                                   |         |                       |
    |          |   ]                                     |         |                       |
    |          | }                                       |         |                       |
    | 98127771 | {                                       | "phone" | "555-1236"            |
    |          |   "business": [                         |         |                       |
    |          |     {                                   |         |                       |
    |          |       "content": "555-1236",            |         |                       |
    |          |       "type": "phone"                   |         |                       |
    |          |     },                                  |         |                       |
    |          |     {                                   |         |                       |
    |          |       "content": "j.doe@company.com",   |         |                       |
    |          |       "type": "email"                   |         |                       |
    |          |     }                                   |         |                       |
    |          |   ]                                     |         |                       |
    |          | }                                       |         |                       |
    | 98127771 | {                                       | "email" | "j.doe@company.com"   |
    |          |   "business": [                         |         |                       |
    |          |     {                                   |         |                       |
    |          |       "content": "555-1236",            |         |                       |
    |          |       "type": "phone"                   |         |                       |
    |          |     },                                  |         |                       |
    |          |     {                                   |         |                       |
    |          |       "content": "j.doe@company.com",   |         |                       |
    |          |       "type": "email"                   |         |                       |
    |          |     }                                   |         |                       |
    |          |   ]                                     |         |                       |
    |          | }                                       |         |                       |
    +----------+-----------------------------------------+---------+-----------------------+
    ```

# Data Warehouse badge
- 모든 과정을 완료하면 배지를 얻을 수 있습니다.
- https://www.credly.com/badges/b9c2e316-fdf9-4187-9bfc-345286ab9fdb/linked_in_profile

# Reference
- https://learn.snowflake.com/en/courses/uni-essdww101/
- [스노우 플레이크 공홈 문서](https://docs.snowflake.com/en/guides-overview)
