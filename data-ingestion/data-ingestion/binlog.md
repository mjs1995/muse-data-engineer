# 이진로그(binlog) 
- MySQL 데이터베이스에서 데이터 추출
  - SQL을 사용한 전체 또는 증분 추출
  - 이진 로그(binlog) 복제 (스트리밍 데이터 수집을 수행하는 하나의 경로)
  - SQL을 사용한 전체 또는 증분 추출은 구현하기가 훨씬 간단하지만 자주 변경되는 대규모 데이터세트에서는 확장성이 떨어집니다.
  - 전체 추출과 증분 추출 사이에도 트레이드오프가 있습니다.
  - 이진 로그 복제는 구현이 더 복잡하지만 원본 테이블의 변경되는 데이터 볼륨이 크거나 MySQL 소스에서 데이터를 더 자주 수집해야 하는 경우에 더 적합합니다. 
- 전체 또는 증분 MySQL 테이블 추출
  - 증분 추출이 최적의 성능에 이상적이지만 어떤 테이블에 대해서는 가능하지 않을 수 있는 몇 가지 단점과 이유가 있습니다.
    - 삭제된 행은 캡쳐되지 않습니다.
    - 원본 테이블에는 마지막으로 업데이트된 시간에 대한 신뢰할 수 있는 타임스탬프가 있어야 합니다.
- MySQL 데이터의 이진 로그 복제
  - <img width="631" alt="image" src="https://user-images.githubusercontent.com/47103479/223445077-9a784ad4-786c-4599-92db-cf08ced2ccba.png">

    - https://www.oreilly.com/library/view/mysql-high-availability/9781449341107/ch04.html
  - 이진 로그는 CDC(변경 데이터 캡처)의 한 형태로 대부분의 원본 데이터 저장소에는 사용할 수 있는 CDC 형식이 있습니다.
  - MySQL 이진 로그는 데이터베이스에서 수행된 모든 작업에 대한 기록을 보관하는 로그
    - 구성된 방식에 따라 모든 테이블의 생성 또는 수정 사항뿐만 아니라 모든 INSERT, UPDATE 및 DELETE 작업도 기록됩니다.
    - 이진 로그의 원래 목적은 다른 MySQL 인스턴스로 데이터를 복제하기 위한 것이지만, 이진 로그의 내용은 데이터 웨어하우스로 데이터를 수집하려는 데이터 엔지니어에게 매우 매력적입니다.
    - ```sql
      SELECT variable_value as bin_log_status
      FROM performance_schema.global_variables
      WHERE variable_name='log_bin';
      ```
  - 이진 로깅 형식
    - STATEMENT : 이진 로그에 행을 삽입하거나 수정하는 행동들에 대해 SQL 문 자체를 기록합니다. 하나의 MySQL 데이터베이스에서 다른 MySQL 데이터베이스로 데이터를 복제하려는 경우 이 형식이 유용합니다.
    - ROW : 주로 사용하는 기본 형식으로 테이블의 행에 대한 모든 변경 사항이 SQL문이 아니라 행 자체의 데이터로 이진 로그 행에 표시됩니다.
    - MIXED : 이진 로그에 STATE 형식 레코드와 ROW 형식 레코드를 모두 기록합니다. 나중에 ROW 데이터만 걸러낼 수도 있지만, 이진 로그를 다른 용도로 사용하지 않는 한 결국 디스크 공간을 추가로 사용하게 되기 때문에 MIXED를 활성화할 필요는 없습니다.
    - ```sql
      SELECT variable_value as bin_log_format
      FROM performance_schema.global_variables
      WHERE variable_name = 'binlog_format
      ```
  - 이진 로그에서 가져올 ROW 형식 이벤트의 3가지 유형
    - WRITE_ROWS_EVENT
    - UPDATE_ROWS_EVENT
    - DELETE_ROWS_EVENT
- 카프카 및 Debezium을 통한 스트리밍 데이터 수집
  - <img width="1045" alt="image" src="https://user-images.githubusercontent.com/47103479/223445308-a2dd2d98-9bd4-4e3d-ab6d-8741c79ece46.png"> 

    - https://debezium.io/documentation/reference/stable/architecture.html
  - MySQL 이진 로그 또는 Postgres WALs와 같은 CDC 시스템을 통해 데이터를 수집할 경우, 훌륭한 프레임워크의 도움이 꼭 필요합니다.
  - Debezium은 여러 오픈소스 서비스로 구성된 분산 시스템으로 일반적인 CDC 시스템에서 행수준 변경을 캡처한 후 다른 시스템에서 사용할 수 있는 이벤트로 스트리밍 해주는 시스템입니다.
    - 아파치 주키퍼 : 분산 환경을 관리하고 각 서비스의 구성을 처리합니다.
    - 아파치 카프카 : 확장성이 뛰어난 데이터 파이프라인을 구축하는 데 일반적으로 사용되는 분산 스트리밍 플랫폼
    - 아파치 카프카 커넥트 
      - 데이터를 카프카를 통해 쉽게 스트리밍 할 수 있도록 카프카를 다른 시스템과 연결하는 도구로 커넥터는 MySQL 및 Postgres와 같은 시스템용으로 구축되었으며 CDC 시스템(이진 로그 및 WAL)의 데이터를 카프카 토픽으로 변환합니다.
      - 카프카 클러스터를 통해 데이터 베이스, 하둡 HDFS, 검색 같은 외부 시스템 및 파일 시스템에 연결하고 데이터를 가져오는/내보내는 프레임워크를 제공
      - 커넥터(Connector) : 데이터 베이스, 키-값 저장소, 검색 엔진, 파일 시스템에 대한 연결 인터페이스
  - 카프카는 토픽별로 정리된 메시지를 교환하며 하나의 시스템은 토픽에 게시(publish)할 수 있는 반면, 하나 이상의 시스템은 토픽을 소비(consume)하거나 구독(subscribe)할 수 있습니다.
  - Debezium 커넥터
    - MongoDB
    - MySQL
    - PostgreSQL
    - Microsoft SQL Server
    - Oracle
    - Db2
    - Cassandra

# Reference
- https://www.oreilly.com/library/view/the-self-service-data/9781492075240/
- https://www.oreilly.com/library/view/data-pipelines-pocket/9781492087823/
- https://netflixtechblog.com/dblog-a-generic-change-data-capture-framework-69351fb9099b
