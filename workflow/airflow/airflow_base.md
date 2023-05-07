# 개요 
## [Airflow](https://airflow.apache.org/)
- Airflow는 파이썬으로 배치, 스케줄링, 모니터링 등을 한 번에 해결하는 워크플로 관리 플랫폼입니다. 
- 일상적인 tasks 는 airflow를 통해서 데이터 파이프라인이 잘 작동하고 있는지 살펴보고, 이에 따라서 예상범위 내에서 데이터가 잘 축적되고 흐르고 있는 지를 계속 감시합니다. 
- Airflow를 사용함으로써 전체 등록된 배치 프로그램들을 한눈에 살펴 보는 것 뿐만 아니라 각각의 프로그램의 단계별 현황까지 확인 후 쉽게 데이터 재생성과 같은 조치를 취할 수 있습니다. Slack과의 연동을 통해 실패할 경우 바로 파악해 볼 수도 있습니다. 
- 백필(Backfilling)이라는 기능이 있는데 하나의 플로(Airflow에서는 DAG)를 특정 옵션(기간) 기준으로 다시 실행할 수 있는 기능으로 태스크가 며칠 동안 실패하거나 새롭게 만든 플로를 과거의 특정 시점부터 순차적으로 실행하고 싶을 때 수행합니다. 

## 사용자 인터페이스
- Dags : 환경에 있는 모든 DAG의 개요입니다
  - ![image](https://user-images.githubusercontent.com/47103479/212094754-05778469-dac6-4d7a-9c65-1c5d248f9b69.png)
- Grid : 시간에 걸쳐 있는 DAG의 그리드 표현입니다.
  - ![image](https://user-images.githubusercontent.com/47103479/212095112-2a6ae2fd-ec48-45aa-82c0-df35e1c5fb26.png)
- Graph : 특정 실행에 대한 DAG의 종속성 및 현재 상태를 시각화합니다.
  - ![image](https://user-images.githubusercontent.com/47103479/212095186-e70f6dc8-6f85-47e0-ab02-0048b722a570.png)
- Task Duration : 시간 경과에 따라 다른 작업에 소요된 총 시간입니다.
  - ![image](https://user-images.githubusercontent.com/47103479/212095272-fe299aaf-93f9-4e63-8ce0-60f6a757d0c8.png)
- Gantt : 각 수행에 대해서 태스크들의 수행 순서에 따라서 소모된 시간과 함께 차트로 표시해줍니다.
  - ![image](https://user-images.githubusercontent.com/47103479/212095863-d82d1226-07bf-45ee-b0d5-34fba8e586a4.png)
- Code: DAG의 소스 코드를 확인할 수 있습니다.
  - ![image](https://user-images.githubusercontent.com/47103479/212095901-e6ede0f4-86ae-48e2-b106-aa718b210686.png)

## Airflow 도입 
- Airflow를 선택하는 이유는 다음과 같습니다.
  - 파이썬 코드를 이용해 파이프라인을 구현할 수 있기 때문에 파이썬 언어에서 구현할 수 있는 대부분의 방법을 사용하여 복잡한 커스텀 파이프라인을 만들 수 있습니다.
  - 파이썬 기반의 Airflow는 쉽게 확장 가능하고 다양한 시스템과 통합이 가능합니다.
  - 수많은 스케줄링 기법은 파이프라인을 정기적으로 실행하고 점진적(증분, incremental) 처리를 통해, 전체 파이프라인을 재실행할 필요 없는 효율적인 파이프라인 구축이 가능합니다.
  - 백필 기능을 사용하면 과거 데이터를 손쉽게 재처리할 수 있기 때문에 코드를 변경한 후 재생성이 필요한 데이터 재처리가 가능합니다.
  - Airflow의 훌륭한 웹 인터페이스는 파이프라인 실행 결과를 모니터링할 수 있고 오류를 디버깅하기 위한 편리한 뷰를 제공합니다.
  - 오픈 소스 기반의 특정 벤더에 종속되지 않고 Airflow를 사용할 수 있습니다.
- Airflow가 적합하지 않은 경우는 다음과 같습니다.
  - 반복적이거나 배치 태스크(batch-oriented task)를 실행하는 기능에 초점이 맞춰져있기 때문에, 스트리밍(실시간데이터 처리) 워크플로 및 해당 파이프라인 처리에 적합하지 않을 수 있습니다.
  - 추가 및 삭제 태스크가 빈번한 동적 파이프라인의 경우에는 적합하지 않을 수 있습니다.
  - 파이썬 언어로 DAG를 구현하기 때문에 파이썬 프로그래밍 경험이 전혀(또는 거의0 없는 팀은 적합하지 않을 수 있습니다.
  - 파이썬 코드로 DAG를 작성하는 것은 파이프라인 규모가 커지면 굉장히 복잡해질 수 있습니다.
  - 워크플로 및 파이프라인 관리 플랫폼이며, 데이터 계보(lineage) 관리, 데이터 버전 관리와 같은 확장 기능은 제공하지 않기 때문에 필요한 경우 언급한 기능을 제공하는 특정 도구를 Airflow와 직접 통합해야합니다.

## 클라우드에서 Airflow
- 관리형 Airflow 서비스
  - 자체 서버에서 에어플로우를 실행하는 것보다 훨씬 더 많은 월별 요금이 발생하지만 에어플로우의 관리 편리성이 높아집니다.
    - [Astronomer.io](https://www.astronomer.io/)
      - ![image](https://user-images.githubusercontent.com/47103479/212096589-dd91585b-e631-47ff-9798-3c809b62ac44.png)
        - [Link](https://docs.astronomer.io/astro/astro-architecture)
        - 쿠버네티스 기반의 솔루션, Airflow를 SaaS형 솔루션(Astronomer Enterprise), 바닐라 Airflow와 비교하면 Astronomer는 Airflow 인스턴스를 쉽게 배포하는 별도의 툴을 제공합니다.
        - UI와 별도로 만들어진 CLI로 제공합니다.
        - CLI는 배포를 위해 Airflow의 로컬 인스턴스를 실행해 주고, 쉽게 DAG를 개발할 수 있게 합니다.
    - [구글 Cloud Composer](https://cloud.google.com/composer)
      - ![image](https://user-images.githubusercontent.com/47103479/212096459-94ea0f82-f44b-41c8-9c87-5659f39d6f68.png)
        - [Link](https://cloud.google.com/composer/docs/concepts/architecture?hl=ko)
        - 구글 클라우드 플랫폼의 최상단에서 구동되는 Airflow의 관리형 버전으로 one-click으로 GCP의 서비스들을 통합하여 Airflow를 배포할 수 있음
    - [아마존 Managed Workflows for Apache Airflow(MWAA)](https://aws.amazon.com/ko/managed-workflows-for-apache-airflow/)
      - ![image](https://user-images.githubusercontent.com/47103479/212096163-bfaa4e7f-8549-4557-b701-e7ec1d4b3ed5.png)
        - [Link](https://docs.aws.amazon.com/ko_kr/mwaa/latest/userguide/what-is-mwaa.html)
        - AWS 클라우드에서 배포를 관리형으로 쉽게 생성해주는 AWS 서비스입니다.
        - AWS 서비스로는 S3, RedShift, SageMaker, 로깅과 알람을 위한 AWS CloudWatch,그리고 웹 인터페이스와 데이터에 대한 액세스 보안을 싱글로그인(single login)으로 제공하는 AWS IAM이 있습니다.
        - 다른 관리형 솔루션과 유사하게 CeleryExecutor를 사용하여 현재 워크로드(workload)에 따라 워커를 스케일링(scaling)하며 사용자 대신 기반 인프라를 관리합니다
- 관리형 솔루션을 선택하는 것은 특정 상황에 따라 달라집니다.
  - 자체 호스팅을 도와줄 수 있는 시스템 운영 팀이 있는가?
  - 관리형 서비스에 지출할 예산이 있는가?
  - 얼마나 많은 DAG와 작업이 파이프라인에 있는가? 
  - 더 복잡한 에어플로우 실행기가 필요한 만큼 충분히 높은 규모로 실행하고 있는가?
- 기타 오케스트레이션 프레임워크
  - [Luigi](https://github.com/spotify/luigi) 및 [Dagster](https://dagster.io/)와 같은 다른 훌륭한 오케스트레이션 프레임워크가 있습니다. 
  - 머신러닝 파이프라인 오케스트레이션에 맞춰진 [Kubeflow Pipelines](https://www.kubeflow.org/)도 ML 커뮤니티에서 인기가 있습니다.
  - 데이터 모델에 대한 변환 단계의 오케스트레이션과 관련하여 피시타운 애널리틱스(Fishtown Analytics)의 [dbt](https://docs.getdbt.com/)는 탁월한 옵션입니다.
- 오픈소스 파이프라인 오케스트레이터인 아파치 Airflow, 우버의 Piper, 넷플릭스의 Meson, 우버의 Cadence(일반 오케스트레이터)에 의해 구현됩니다.
  - 효율적으로 관리할 수 있는 어플리케이션 [Pinball](https://github.com/pinterest/pinball), [Azkaban](https://azkaban.github.io/), [Oozie](https://oozie.apache.org/) 등이 있습니다.

## AWS 환경에서의 Airflow
- S3는 확장 가능한 오브젝트 스토리지 시스템, 상대적으로 저렴한 비용으로 높은 내구성(durability)과 가용성(availability)을 제공하여 일반적으로 많은 데이터를 저장하는 데 매우 적합합니다.
- S3는 용량이 큰 데이터 세트(DAG에서 처리되는 데이터 세트)를 저장하거나 Airflow 워커 로그들(Airflow가 S3에 기본적으로 쓸 수 있음)과 같이 임시적인 파일들을 저장할때 유용합니다. 
- 단점은 웹 서버나 스케줄러 시스템의 로컬 파일 시스템으로 마운트할 수 없다는 것, DAG와 같이 Airflow가 로컬 액세스가 필요한 파일들을 저장할 때 S3를 사용하는 것은 그다지 좋은 생각이 아닙니다.
- AWS Athena를 사용한 서버리스 영화 랭킹 구축 사례 
  - Glue는 S3에 저장된 데이터의 테이블 뷰를 생성하여, 영화 랭킹을 계산할 때 데이터를 쿼리할 수 있도록 합니다.
  - AWS Athena(서버리스 SQL 쿼리 엔진)를 사용하여 평점 테이블에서 SQL 쿼리를 실행하고 영화랭킹을 계산합니다.
  - S3및 Glue/Athena는 매우 확장성이 좋은 기술이고 서버리스로 구성하기 때문에 서버를 별도로 구축할 필요가 없고 한 달에 한 번 실행되는 프로세스에 대해 서버 비용을 사용할 때만 지불하여 비용을 절감할 수 있습니다.
  - 재사용성을 위해 CloudFormation(코드로 클라우드 리소스를 정의하는 AWS 템플릿 솔루션)과 같은 코드형 인프라(infrastructure-as-code,IaC) 솔루션으로 리소스를 정의하고 관리하는 것이 좋습니다.

## Reference
- http://www.yes24.com/Product/Goods/107878326
- https://airflow.apache.org/docs/apache-airflow/stable/index.html
- https://www.reddit.com/r/dataengineering/
