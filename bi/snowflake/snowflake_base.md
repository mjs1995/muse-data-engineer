# Snowflake
- ![image](https://github.com/mjs1995/muse-data-engineer/assets/47103479/b796aa4c-3b0c-4584-856d-6cf56d3765eb)
  - https://www.snowflake.com/blog/beyond-modern-data-architecture/?lang=ko
- SaaS(Software-as-a-Service) 모델로 제공되는 구조화된 데이터와 반 구조화된 데이터 모두를 지원하는 데이터웨어 하우스입니다.
- Snowflake는 빠르고 사용자 친화적이며 기존 데이터웨어 하우스보다 더 많은 유연성을 제공합니다.
- Snowflake는 Snowflake Elastic Data Warehouse의 형태로 클라우드 기반 데이터 스토리지 및 분석을 제공합니다. 사용자는 클라우드 기반 하드웨어 및 소프트웨어를 사용하여 데이터를 분석하고 저장할 수 있습니다.
- 특징 
  - 클라우드 기반: 클라우드 기반 데이터 웨어하우스로, 클라우드에서 데이터 분석을 지원하며 더 높은 확장성과 유연성을 제공합니다.
  - 분산 아키텍처: 클러스터를 사용하여 데이터 처리 능력을 확장합니다.
  - 새로운 데이터 타입 지원: 반정형 및 비정형 데이터를 저장하고 처리하기 위해 다양한 데이터 포맷을 지원합니다.
  - 적은 유지 보수 비용: 스노우 플레이크는 클라우드에서 호스팅 되므로 사용자는 하드웨어, 소프트웨어 라이선스 및 유지 보수 비용을 지불할 필요가 없습니다.
  - 보안: 스노우 플레이크는 보안 기능이 내장되어 있으며, 클라우드 공급자와의 협력으로 사용자 데이터를 안전하게 보호합니다.

## Snowflake 플랫폼
- ![image](https://user-images.githubusercontent.com/47103479/237041217-7adc1cfa-d23b-4fd9-8304-73e1b862df75.png)
  - https://www.snowflake.com/product/architecture/
- Snowflake 플랫폼 장점
  - 컴퓨팅과 스토리지의 분리
  - 멀티 클라우드 지원
  - 단일 플랫폼과 단일 원본 데이터
  - 성능을 위한 무한 확장 및 비용 최적화
  - 제로에 가까운 유지보수(SaaS)
  - 데이터마켓플레이스 통한 분석의다양성, 정확도향상
- 데이터 보호
  - End to End 암호화로 항상 암호화 된 통신 채널 사용(TLS 1.2)
  - 저장된 데이터는 Snowflake 드라이버 및 시스템에 의해 처리 되는 동안 항상 암호화 (AES-256)
  - 감사 로그 : 모든 로그인, 트랜잭션, 데이터 전송 등에 대한 이력관리 및 고객사의 보안 툴과 연계 기능 제공
  - 사용자 정의 역할을 통하여 객체, 작업 및 컴퓨팅 자원에 대한 제어
- Snowflake 워크로드
  - ![image](https://user-images.githubusercontent.com/47103479/237044471-bfa54697-c2dd-4ea6-9edc-9a90fa40e808.png)
  - 데이터 워크로드(Data Workload)란 데이터 획득 및 접근을 도와주고 데이터에서 가치를 추출하거나 다른 사용자들이 사용할 수 있는 기초를 제공하는 능력, 서비스 또는 프로세스를 말합니다.
  - Snowflake 워크로드에는 데이터 엔지니어링, 데이터 웨어하우징, 데이터 레이크, 데이터 분석 및 데이터 과학 워크로드, 데이터 협업 워크로드를 다루어 데이터 수익화를 위한 Snowflake의 안전한 데이터 공유 기능이 있습니다.

## Streamlit
- ![image](https://github.com/mjs1995/muse-data-engineer/assets/47103479/6ec5116d-48b2-4fe5-8faa-bf52248e261e)
  - https://quickstarts.snowflake.com/guide/data_apps_summit_lab/index.html#5
- Snowflake 데이터 과학에서는 2022년 3월 Streamlit을 인수를 했습니다. Streamlit은 복잡하고 동적인 응용 프로그램을 구축하는 데 사용할 수 있는 많은 기계 학습 및 데이터 과학 사례를 갖추고 있으며 Streamlit 프레임 워크 내에서 사용자는 Python 코드로 풍부한 대화형 시각화를 만들 수 있습니다. 
- Streamlit 관련해서 ML 모델 대시보드는 다음의 [프로젝트](https://mjs1995.tistory.com/188)에서 확인할 수 있습니다.

## 마켓 플레이스
- ![image](https://github.com/mjs1995/Book_review/assets/47103479/c37a1472-c3c2-4f53-8f07-dffa7cd39f7a)
- Snowflake 데이터 플랫폼과 통합된 데이터 제품 및 서비스의 시장으로 Snowflake 사용자들은 필요한 데이터 솔루션을 쉽게 검색하고 구매할 수 있습니다. 
- Marketplace는 데이터 통합, 보안, 분석, 인공지능 및 기계학습, 데이터 시각화, ETL 등 다양한 분야에서 솔루션을 제공합니다

# Reference
- [Snowflake: The Definitive Guide](https://www.oreilly.com/library/view/snowflake-the-definitive/9781098103811/)
- [스노우 플레이크 공홈 문서](https://docs.snowflake.com/en/guides-overview)
