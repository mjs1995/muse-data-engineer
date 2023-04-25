# dbt
- ![image](https://user-images.githubusercontent.com/47103479/233097856-1c204bec-732b-4b12-bec3-4ac75f8a8c7e.png)
- dbt(Data Build Tool)는 개발자가 BigQuery, Snowflake, Redshift 등과 같은 최신 데이터 웨어하우스에서 변환을 정의, 오케스트레이션 및 실행할 수 있도록 하여 데이터 모델 구축을 간소화하는 Python 오픈 소스 라이브러리입니다. 
- ETL/ELT 프로세스의 T에 초점을 맞춘 거버넌스 도구라고 말할 수 있습니다. 이를 통해 SQL에서 모든 데이터 변환을 중앙 집중화하고 구축하여 재사용 가능한 모듈(모델)로 구성할 수 있습니다 
- dbt는 데이터 모델링을 SQL로 정의하고 관리하며, 모델이 실행될 때 데이터를 추출, 가공 및 저장할 수 있도록 도와줍니다. 
- dbt는 Git 버전 관리 시스템을 사용하여 데이터 모델링 파일을 관리하며, 테스트, 문서화 및 모델 종속성 관리 기능도 제공합니다.
- 소프트웨어 엔지니어링 관행에서 영감을 받아 검증 테스트를 만들고 데이터 파이프라인에서 전체 CI/CD 주기를 구현할 수 있습니다. 
- dbt의 주요 기능
  - 코드 재사용 : 데이터 모델의 정의 및 패키지의 변환 구성을 허용합니다.
  - 품질 검사에 대한 강조 : 데이터 품질을 보장하고 변환 오류를 방지하기 위해 자동화된 테스트의 사용을 권장합니다.
  - 버전 제어 및 협업 : Git, Bitbucket 등과 같은 버전 제어 시스템과 함께 작동하도록 설계되어 변경 사항을 쉽게 추적하고 개발 파이프라인에서 협업할 수 있습니다.
  - 확장성 : BigQuery, Snowflake, Redshift 등과 같은 최신 데이터 웨어하우스와 함께 작동하도록 설계되어 대용량 데이터 처리를 쉽게 확장할 수 있습니다.
- dbt 사용
  - dbt Core: 로컬 또는 자체 서버에 설치된 오픈 소스 버전입니다. CLI 콘솔을 통해 이루어집니다.
  - dbt Cloud: Core 버전에 추가 기능(실행 예약, BI, 모니터링 및 경고)을 제공하는 클라우드(SaaS)에서 호스팅 되는 플랫폼입니다. GUI가 있어 사용하기가 더 쉽습니다. 유료 요금제 외에도 개발자를 위한 제한된 무료 버전을 제공합니다.
- |Resource|설명|
  |:---:|:---:|
  |models|각 모델은 단일 파일에 있으며 원시 데이터를 분석 준비가 된 데이터 세트로 변환하거나 이러한 변환의 중간 단계인 로직을 포함합니다.
  |snapshots|나중에 참조할 수 있도록 변경 가능한 테이블의 상태를 캡처하는 방법입니다.
  |seeds|dbt를 사용하여 데이터 플랫폼에 로드할 수 있는 정적 데이터가 포함된 CSV 파일.
  |tests|프로젝트의 모델 및 리소스를 테스트하기 위해 작성할 수 있는 SQL 쿼리입니다.
  |macros|여러 번 재사용할 수 있는 코드 블록입니다.
  |docs|빌드할 수 있는 프로젝트용 문서입니다.
  |sources|추출 및 로드 도구로 웨어하우스에 로드된 데이터의 이름을 지정하고 설명하는 방법입니다.
  |exposures|프로젝트의 다운스트림 사용을 정의하고 설명하는 방법입니다.
  |metrics|프로젝트에 대한 메트릭을 정의하는 방법입니다.
  |analysis|프로젝트에서 분석 SQL 쿼리를 구성하는 방법입니다. 

## Materialization 
- SQL 모델을 물리적으로 저장하는 방식을 의미하며 4가지의 방식이 있습니다.
- Table
  ```sql  
  {{ config(materialized='table', sort='timestamp', dist='user_id') }}

  select *
  from ...
  ```
  - SQL 모델을 특정 테이블로 저장합니다. 
  - 매우 간단한 방식으로 SQL 모델을 저장하며, 향후 쿼리 수행 시 직접 테이블에서 데이터를 읽습니다.
- View 
  ```sql
  {{ config(materialized='view',
          indexes=[{'columns': ['col_a'], 'cluster': 'cluster_a'}]) }}
          indexes=[{'columns': ['symbol']}]) }}

  select ...
  ```
  - SQL 모델을 뷰(View)로 저장합니다. 
  - SQL 모델을 쿼리할 때마다 실행되며, 결과를 즉시 반환합니다. 
- Incremental
  - ```sql
    {{
    config(
      materialized='incremental',
      unique_key='date_day',
      incremental_strategy='delete+insert',
      ...
      )
    }}

    select ...
    ```
  - dbt가 마지막으로 실행된 이후로 dbt가 테이블에 레코드를 삽입하거나 업데이트할 수 있습니다.
  - 이전 실행 결과와 현재 실행 결과를 비교하여 변경된 데이터만 처리합니다. 
- Ephemeral
  - ```sql
    {{ 
    config(
      materialized='ephemeral'
      ...
      )
    }}

    select ...
    ```
  - SQL 모델을 쿼리할 때마다 실행되며, 결과를 메모리에 저장합니다. 
  - 모델은 데이터베이스에 직접 구축되지 않습니다. 대신 dbt는 이 모델의 코드를 종속 모델에 공통 테이블 로 보관합니다.

## Sources 
- 데이터 소스를 설정하고 이를 사용하여 SQL 모델을 작성할 때 데이터에 액세스할 수 있는 방법을 정의합니다.
- schema.yml 파일에는 버전, 소스 이름, 데이터베이스, 스키마 및 테이블이 포함되어 있습니다. yml 파일을 이용해서 단일 위치에서 모든 모델에 대한 연결을 변경할 수 있습니다.
- ![image](https://user-images.githubusercontent.com/47103479/233846968-03bd4d2c-744e-43ca-a417-03bfa44a8e7b.png)
- ```yml
  version: 2

  sources:
      - name: staging
        database: dtc-de-382512
        schema: trips_data_all

        tables:
            - name: green_tripdata
            - name: yellow_tripdata
  ```
- ```sql
  {{ config(materialized='table') }}

  SELECT *
  FROM {{ source('staging','green_tripdata') }}
  ```
- ![image](https://user-images.githubusercontent.com/47103479/233847108-5f702ea6-814f-4798-b3f5-1f4106aef6fc.png)

## Seeds
- 자주 변경되지 않는 데이터에 권장되며 dbt seed -s file_name으로 실행됩니다.
- 시드를 생성하려면 저장소의 /seeds 디렉토리에 CSV 파일을 업로드하고 명령을 실행하기만 하면 됩니다.
- dbt seed --full-refresh : dbt 프로젝트의 시드 데이터를 다시 생성할 때 사용됩니다. --full-refresh 옵션은 시드 데이터를 완전히 새로 고침하고 기존 데이터를 모두 삭제한 후, 새로운 시드 데이터를 다시 생성합니다.
- ```yaml
  seeds: 
    taxi_rides_ny:
        taxi_zone_lookup:
            +column_types:
                locationid: numeric
  ```
- ![image](https://user-images.githubusercontent.com/47103479/233999620-4cc70dbe-a43b-4222-89d8-e17716234ddf.png)

## Ref
- 다른 모델에 대한 참조(reference)를 만드는 데 사용되는 함수입니다. ref 함수를 사용하여 다른 모델에 대한 참조를 만들면, 해당 모델의 결과를 현재 모델에서 사용할 수 있습니다.
- 모델 간의 종속성을 정의할 수 있으며, 이를 통해 dbt는 모델 실행 순서를 자동으로 결정할 수 있습니다. 데이터 파이프라인의 실행 순서를 자동으로 관리할 수 있으며, 데이터 소스의 변경 내용에 대한 업데이트도 간편하게 수행할 수 있습니다.
- models/<schema>.yml
  - ```yml    
    models:
      - name: model_name
        latest_version: 2
        versions:
          - v: 2
          - v: 1
    ```
- ```sql
  select * from {{ ref('model_name', version=1) }}
  ```

## Macros
- DBT에서 사용할 수 있는 사용자 정의 함수로 SQL 쿼리나 모델 코드에서 호출될 수 있으며, DBT가 실행되는 동안 코드를 생성할 때 자동으로 확장됩니다.
- 코드 재사용성을 향상시킬 수 있으며, 반복적인 작업을 수행하는 데 필요한 코드를 줄일 수 있습니다. DBT 모델의 가독성과 유지 보수성을 향상시킬 수 있습니다.
- Jinja 템플릿 언어
  - Expressions {{ ... }}: 표현식은 문자열을 출력하고자 할 때 사용합니다. 식을 사용하여 변수를 참조 하고 매크로를 호출할 수 있습니다.
    - ```sql
      SELECT *
      FROM my_table
      WHERE my_column = '{{ my_variable }}'
      ```
  - Statements {% ... %}: 제어문(if 문, for 문 등)이나 매크로를 정의합니다.
    - ```sql
      {% if condition %}
      SELECT *
      FROM my_table
      WHERE my_column = 'some_value'
      {% else %}
      SELECT *
      FROM my_table
      WHERE my_column = 'other_value'
      {% endif %} 
      ```
  - Comments {# ... #}: 모델 내에서 주석을 추가할 때 사용합니다.
    - ```sql
      {# 주석 추가 #}
      SELECT *
      FROM my_table
      ```
- payment_type 의 값을 매개변수로 받고 해당 값을 반환 하는 것을 확인 하는 get_payment_type_description 매크로를 만듭니다.
- ```sql
    {#
  This macro returns the description of the payment_type 
  #}

  {% macro get_payment_type_description(payment_type) -%}

      case {{ payment_type }}
          when 1 then 'Credit card'
          when 2 then 'Cash'
          when 3 then 'No charge'
          when 4 then 'Dispute'
          when 5 then 'Unknown'
          when 6 then 'Voided trip'
      end

  {%- endmacro %}
  ```
- ![image](https://user-images.githubusercontent.com/47103479/233994947-9e697e51-2292-4337-8ee7-9f8bbc45674d.png)


## Packages
- 다른 프로그래밍 언어의 라이브러리나 모듈과 유사하게 서로 다른 프로젝트 간에 매크로를 재사용할 수 있습니다. 
- 프로젝트에서 패키지를 사용하려면 dbt 프로젝트의 root 디렉토리에 packages.yml 구성 파일을 생성해야 합니다.
- 데이터 모델링 작업의 생산성을 높일 수 있으며, 일관된 방식으로 데이터 모델링을 수행할 수 있습니다.
- ```yml
  packages:
    - package: dbt-labs/dbt_utils
      version: 0.8.0
  ```
- > dbt deps : 프로젝트 내 패키지의 모든 종속성 및 파일을 다운로드하는 명령을 실행합니다. 완료되면 프로젝트에 dbt_packages/dbt_utils 디렉토리가 생성됩니다
  - ![image](https://user-images.githubusercontent.com/47103479/233995888-3eb1f76f-5ce8-4f77-9a42-f2135e3ae7de.png)

## Variables
- 프로젝트 전체에서 사용해야 하는 값을 정의하는 데 유용하며 여러 모델에서 공통으로 사용하는 값을 중앙에서 관리할 수 있습니다.
-  dbt_project.yml 파일이나 command line에서 사용할 수 있습니다.
- ```sql
  {{ config(materialized='table') }}

  SELECT *
  FROM {{ source('staging','green_tripdata') }}
  {% if var('is_test_run', default=true) %}

      limit 100

  {% endif %}
  ```
- ![image](https://user-images.githubusercontent.com/47103479/233996540-09269444-ad3f-4603-9507-6257e39df6e7.png)
- > dbt build --var 'is_test_run: false' : 모델을 빌드할 때 명령줄에서도 사용 가능합니다. 

## Tests
- dbt 모델의 유효성을 검사하는 데 사용됩니다. dbt test는 dbt 모델이 데이터의 정합성과 일관성을 보장하는지 검증하는 데 사용되는 유용한 도구입니다.
- dbt test는 모델을 실행하기 전에 실행됩니다. 모델의 실행 결과가 올바른지 확인하고, 예상한 결과와 실제 결과가 일치하는지 검증합니다.
- dbt는 열 값이 다음과 같은지 확인하는 기본 테스트를 제공합니다.
  - Unique : 특정 컬럼에 중복된 값이 없는지 확인합니다.
  - Not null : 특정 컬럼에 null 값을 포함하지 않는지 확인합니다.
  - Accepted values : 특정 컬럼에 허용되는 값 목록이 지정한 값들과 일치하는지 확인합니다.
  - relationship : 두 모델 간의 관계가 유효한지 확인합니다.
- ```yml
  columns:
    - name: tripid
      description: Primary key for this table, generated with a concatenation of vendorid+pickup_datetime
      tests:
          - unique:
              severity: warn
          - not_null:
              severity: warn
    - name: Pickup_locationid
      description: locationid where the meter was engaged.
      tests:
        - relationships:
            to: ref('taxi_zone_lookup')
            field: locationid
            severity: warn
    - name: Payment_type 
      description: >
        A numeric code signifying how the passenger paid for the trip.
      tests: 
        - accepted_values:
            values: "{{ var('payment_type_values') }}"
            severity: warn
            quote: false
  ```
- ![image](https://user-images.githubusercontent.com/47103479/234008735-22a36fe5-792d-436a-a8a1-54123a9d3fe8.png)

## Documentation
- DBT에서 제공하는 자동화된 문서화 도구로 DBT 모델의 메타데이터와 설명을 문서화하여 데이터 웨어하우스의 모델링 프로세스를 관리하고 문서화하는 데 사용됩니다.
- dbt docs generate 명령어를 실행하면 모델링 코드와 schema.yml 파일을 기반으로 문서화가 생성됩니다.
- ![image](https://user-images.githubusercontent.com/47103479/234305315-1c7be59c-c976-4ec5-825c-29850cabea48.png)
- ![image](https://user-images.githubusercontent.com/47103479/234305857-dae72279-7872-4a2d-9842-b9b9117fef6e.png)
  
# Reference
- https://docs.getdbt.com/docs/build/projects
