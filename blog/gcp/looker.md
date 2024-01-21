# Looker
## Looker 개요
- Looker는 Google Cloud의 혁신적인 데이터 플랫폼으로, 사용자가 데이터를 대화형으로 분석하고 시각화할 수 있게 해줍니다. Looker를 통해 사용자는 심층적인 데이터 분석을 수행하고, 다양한 데이터 소스를 통합하여 통찰력을 얻으며, 데이터 기반의 워크플로와 맞춤형 데이터 애플리케이션을 구축할 수 있습니다.
- ![image](https://github.com/mjs1995/muse-data-engineer/assets/47103479/6ff3fd27-c231-4d98-b3f7-bf02e4b30c26)
  - https://www.youtube.com/watch?v=kxgcj9T0G1Q&t=215s
- 사용자는 Looker를 통해 데이터를 심층적으로 분석하고, 다양한 데이터 소스를 통합하며, 데이터 기반 워크플로와 맞춤형 데이터 애플리케이션을 구축할 수 있습니다.
- 플랫폼 특징
  - 브라우저 기반 SaaS: Looker는 SQL 데이터베이스에 직접 연결되는 SaaS 플랫폼입니다.
  - 다양한 데이터 소스 연결: Salesforce, Mailchimp, Zendesk 등의 SaaS 애플리케이션과 연결 가능합니다.
  - 멀티 클라우드 지원
- Looker의 주요 이점
  - 민첩한 모델링 계층: Looker의 모델링 계층을 통해 SQL 쿼리를 수동으로 작성하고 편집하는 시간을 절약할 수 있습니다.
  - LookML: SQL의 추상화 계층으로, 개발자는 LookML을 사용해 데이터베이스 구성과 테이블 간의 관계를 정의할 수 있습니다.
  - 데이터 거버넌스: 모든 사용자가 이해하고 신뢰할 수 있는 단일 정보 소스를 제공합니다.
- 사용자 인터페이스 및 데이터 보안
  - Looker UI를 통해 사용자 역할 할당, 필드 및 데이터 행에 대한 접근 제어 등을 관리할 수 있습니다.
- 데이터 표시 및 공유
  - 웹 인터페이스, 이메일 또는 Cloud Storage를 통한 데이터 전송, 다른 웹사이트나 애플리케이션에 대한 통합 등 다양한 방법으로 쿼리 결과를 표시하고 공유할 수 있습니다.
- REST API 및 개발 프레임워크
  - Looker는 REST API를 제공하여 플랫폼에서 직접 데이터와 메타데이터를 검색, 분석, 변환할 수 있도록 합니다.
  - 이 아키텍처는 엔터프라이즈급 워크플로와 정확하고 최신 버전의 데이터에 대한 액세스를 지원합니다.
- 개발 모드와 프로덕션 모드
  - Looker는 개발 모드와 프로덕션 모드를 제공합니다. 개발 모드를 사용하면 LookML 파일을 변경하고, 이 변경이 인스턴스 콘텐츠에 미치는 영향을 미리 볼 수 있습니다. 이 모드에서의 변경사항은 프로덕션 환경에 반영되지 않아 안전한 개발 환경을 제공합니다. 또한, Git과의 통합을 통해 버전 관리가 용이해집니다.
  - 개발 모드로 전환하면 Looker는 변경 사항을 적용하고 테스트할 수 있는 로컬 분기를 자동으로 생성합니다.이 브랜치는 본인만 수정할 수 있지만, 동일한 프로젝트에 액세스할 수 있는 개발자는 원할 경우 다른 사람의 브랜치를 볼 수 있습니다.
- 버전 관리와 Git 통합
  - Looker는 Git과 통합되어 있어, 개발자는 LookML 파일의 사본을 안전하게 수정하고 버전 관리할 수 있습니다. Looker의 IDE는 변경사항을 커밋하고, 가져오며, 내보내는 Git 워크플로를 자동으로 관리합니다.

## 파생 테이블과 쿼리 최적화
- 파생 테이블
  - Looker는 SQL Runner를 통해 데이터베이스에 직접 액세스할 수 있습니다. 이를 통해 파생 테이블을 생성하고, 쿼리를 최적화할 수 있습니다. 파생 테이블은 복잡하거나 자주 사용되는 쿼리를 빠르게 실행하기 위해 사용되며, 캐시되거나 지속될 수 있습니다. 이를 통해 데이터 업데이트를 신속하게 반영하고, 쿼리 런타임을 단축할 수 있습니다.
  - Looker의 영구 파생 테이블(PDT)
    - Looker에서 사용하는 영구 파생 테이블(PDT)은 SQL 및 LookML 쿼리를 이용해 데이터베이스에 임시 테이블로 생성된 테이블입니다.
    - 용도: 복잡하거나 자주 사용되는 쿼리를 반복적으로 실행하고, 빠른 데이터 액세스를 위해 결과를 캐시하는 데 사용됩니다.
    - 이점: 쿼리 런타임 감소, 비즈니스 사용자가 필요할 때 즉시 데이터 사용 가능.
    - 단점: 데이터베이스 저장 공간 사용 증가(비용 증가 가능성).
    - PDT 매개변수
      - Datagroup Trigger: 모델에 구성된 데이터 그룹 또는 캐싱 정책 사용.
      - SQL Trigger Value: 하나의 값을 반환하는 SELECT 문을 사용, 데이터베이스에 반복적으로 보내 PDT 재작성 유발.
      - Persist For: 설정된 기간 동안 PDT를 유지(예: "1시간" 또는 "4시간").
    - PDT 사용 예시 : 이 예시는 order_items를 원본으로 사용하는 order_details_summary 뷰의 파생 테이블을 정의합니다. 데이터 그룹 트리거를 사용하여 PDT 캐싱 정책을 적용합니다.
- 쿼리 최적화
  - 방법 
    - 증분 영구 파생 테이블 생성
      - ![image](https://github.com/mjs1995/muse-data-engineer/assets/47103479/38a0b7b2-23f4-4b9a-8c2e-188826aebb8d)
      - 전체 테이블을 다시 작성하지 않고 자동으로 업데이트됩니다.
      - 새로운 데이터를 기존 테이블에 추가하여 쿼리 크기를 줄입니다.
    - 데이터 그룹 사용
      - 매일 새로고침을 통해 객체를 유지합니다.
      - 예시: datagroup: daily_datagroup은 최대 24시간 동안 캐시를 유지합니다.
    - 증분 집계 테이블
      - 여러 기간에 걸친 주문 데이터를 요약합니다.
      - LookML의 aggregate_table 매개변수를 사용해 정의됩니다.
    - 효율적인 뷰 조인
      - 탐색 정의 시 필요한 뷰만 조인합니다.
      - 기본 필드를 기본 키로 사용하고, 가능한 한 many_to_one 조인을 사용합니다.
    - PDT 빌드 모니터링
      - Looker 인스턴스에서 PDT의 상태, 빌드 시간, 캐싱을 모니터링합니다.
  - 증분 영구 파생 테이블 예시 : 이 예시는 order_items를 기반으로 한 증분 파생 테이블을 정의합니다. daily_datagroup 데이터 그룹을 사용하여 캐시 정책을 적용합니다.
    - ```yaml
      view: incremental_pdt {
        derived_table: {
          datagroup_trigger: daily_datagroup
          increment_key: "created_date"
          increment_offset: 3
          explore_source: order_items {
            column: order_id {}
            column: sale_price {}
            column: created_date {}
            column: created_week {}
            column: created_month {}
            column: state { field: users.state }
          }
        }
        // 추가적인 차원과 측정 정의
      }
      ```

# LookML(Looker Modeling Language)
- Looker는 LookML을 사용하여 데이터를 모델링합니다. 예를 들어, 사용자 테이블에서 다양한 차원과 측정치를 정의할 수 있으며, 이를 통해 데이터를 더욱 효과적으로 분석할 수 있습니다. 또한, 여러 데이터 소스를 조인하여 복합적인 데이터 모델을 구축할 수도 있습니다.
- LookML 구조 개요
  - ![image](https://github.com/mjs1995/muse-data-engineer/assets/47103479/3d6b3bae-e7a6-4ac2-a67b-5d68d8208786)
    - https://www.cloudskillsboost.google/focuses/18478?parent=catalog
  - LookML은 데이터 모델링을 위한 Looker의 언어로, 구조화된 계층 시스템을 사용하여 데이터를 정의하고 조작합니다. 이를 통해 SQL 쿼리를 보다 효율적으로 활용하고, 데이터 분석과 시각화를 위한 강력한 도구를 만들 수 있습니다.
  - LookML의 주요 구성 요소는 다음과 같습니다.
    - 프로젝트 (Projects)
      - 개념: LookML 코드 라이브러리.
      - 특징: 각 프로젝트는 일반적으로 데이터 소스나 데이터베이스 연결에 매핑되며, Git을 통한 버전 관리가 가능합니다.
      - 사용: 서로 다른 스키마의 데이터셋은 서로 다른 프로젝트에 속합니다.
    - 모델 (Models)
      - 구성: 프로젝트 내부에 존재하며, 하나 이상의 탐색(Explore)을 포함합니다.
      - 기능: 데이터베이스 연결을 정의하고, 탐색 및 그 조인 논리를 설정합니다.
      - 목적: 사용자가 특정 탐색에 액세스하는 것을 제한하거나, 비즈니스 영역별로 탐색을 분리하고 구성합니다.
    - 탐색 (Explores)
      - 정의: 하나 이상의 뷰(Views)가 결합된 것.
      - 역할: 사용자 인터페이스에서 분석의 주요 동력 역할을 하며, 특정 비즈니스 질문에 대한 데이터를 제공합니다.
      - 구성: 탐색은 비즈니스 주제를 중심으로 구성되어야 하며, 사용자가 쉽게 이해하고 활용할 수 있어야 합니다.
    - 뷰 (Views)
      - 설명: 데이터베이스 테이블 또는 그 논리적 표현.
      - 종류: 표준 뷰(데이터베이스 테이블의 추상화) 및 파생 테이블(가상 테이블)
      - 기능: 데이터의 차원(Dimensions)과 측정값(Measures)을 정의합니다.
    - 차원 (Dimensions) 및 측정값 (Measures)
      - 차원: 테이블의 열 또는 그 논리적 표현을 나타냅니다. 데이터의 속성을 설명합니다.
      - 측정값: 데이터베이스 테이블에 명시적으로 존재하지 않는 집계 함수입니다. 차원을 기반으로 합계, 평균, 카운트 등을 계산합니다.
- 기본 구조와 사용 예제
  - LookerML에서는 view를 정의하여 특정 데이터 테이블을 나타냅니다. 각 view 내에서는 다양한 dimension, measure, dimension_group 등을 정의하여 데이터를 다룹니다.
  - 사용자 데이터를 분석하기 위한 view는 다음과 같이 구성될 수 있습니다
  - ```yaml
    view: users {
      sql_table_name: `database_schema.table_name`;;

      dimension: id {
        primary_key: yes
        type: number
        sql: ${TABLE}.id ;;
      }

      dimension: age {
        type: number
        sql: ${TABLE}.age ;;
      }

      dimension_group: created {
        type: time
        timeframes: [date, week, month]
        sql: ${TABLE}.created_at ;;
      }

      measure: total_sales {
        type: sum
        sql: ${sale_price} ;;
      }
    }
    ```
