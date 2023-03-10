# Hive Architecture
- ![image](https://user-images.githubusercontent.com/47103479/217844082-f04a2a6d-0e8b-4b12-ad19-5f75b64831d0.png)
  - https://www.simplilearn.com/tutorials/hadoop-tutorial/hive
- HiveQL
  - HiveQL은 하이브의 SQL언어인 HiveQL은 SQL-92, MySQL, 오라클 SQL을 혼합한 것입니다.
  - Apache Hive는 데이터 웨어하우스 소프트웨어 프로젝트로, 본래 하둡 에코시스템용으로 개발되었습니다. Hive는 온프레미스와 클라우드 내에서 다양한 스토리지 매체(예: HDFS, Azure 클라우드 스토리지, Amazon Web Services S3 오브젝트 스토리지, Google Cloud 스토리지 등)와 함께 사용할 수 있습니다. 
  - Hive는 데이터를 행, 열, 데이터 유형을 포함한 테이블로 표현하는 추상화 계층을 제공하며 SQL 인터페이스, HiveQL로 쿼리하고 분석합니다. 
  - Hive는 인메모리 분산형 엔진 Apache Tez로 데이터를 처리함
  - Apache Hive는 Hive LLAP와의 트랜잭션(ACID)을 지원합니다. 트랜잭션을 이용하면 동시에 여러 사용자/프로세스가 데이터에 액세스하여 생성, 읽기, 업데이트 및 삭제(CRUD) 작업을 하는 환경에서도 데이터의 일관된 보기를 보장합니다.
  - Hive는 Optimized Row Columnar(ORC) 파일 형식에 최적화되었고 Parquet도 지원합니다.
- Hive web interface
  - 명령줄 인터페이스와 별도로 Hive는 Hive 쿼리 및 명령을 실행하기 위한 웹 기반 GUI도 제공합니다.
  - hwi(하이브 웹 인터페이스)는 클러스터로 로그인하거나 CLI를 사용하지 않고 쿼리나 다른 명령을 수행하기 위한 간단한 웹 인터페이스
- JDBC / ODBC Driver
  - Apache Hive 드라이브는 클라이언트가 CLI, 웹 UI, Thrift, ODBC 또는 JDBC 인터페이스를 통해 제출한 쿼리를 수신하는 역할을 합니다. 드라이버는 metastore에 있는 스키마의 도움으로 구문 분석, 유형 검사 및 의미 체계 분석이 수행되는 컴파일러에 쿼리를 전달합니다 . 
  - map-reduce 작업과 HDFS 작업의 DAG(Directed Acyclic Graph) 형태로 최적화된 논리적 계획을 생성하고 실행 엔진은 Hadoop을 사용하여 종속성 순서대로 이러한 작업을 실행합니다.
  - ODBC - 애플리케이션 하이브를 포함한 SQL 시스템에 접근할 수 있도록 제공하는 오픈 데이터베이스 연결 API(Open Database Connection API)를 뜻하며 자바 애플리케이션은 보통 JDBC API를 대신 사용합니다.
  - JDBC - 자바 데이터베이스 연결 API(Java Database Connection API)는 자바 코드를 이용해 하이브를 포함한 SQL 시스템에 접속하는 방법을 제공합니다.
- Hive CLI(Command Line Interface)
  - Hive 쿼리 및 명령을 직접 실행할 수 있는 Hive에서 제공하는 기본 셸입니다.
  - CLI(명령행 인터페이스)는 테이블을 정의하거나 쿼리 등을 수행하기 위해 사용되며 다른 서비스가 명시되어 있지 않으면 기본 서비스로 동작합니다.
- Thrift 
  - 하이브는 하이브 서버(HiveServer) 또는 하이브 쓰리프트(HiveThrift)라 불리는 구성요소를 가지고 있으며 클라이언트는 하나의 포트로 하이브에 접근할 수 있습니다.
  - 쓰리프트는 확장성(scalable)과 교차 언어적(cross language. 서로 다른 언어 간에 통신이 가능하다는 의미) 특징을 갖는 소프트웨어 프레임워크입니다.
  - 쓰리프트는 페이스북에서 고안한 RPC 시스템으로 하이브와 통합되어 있으며 원격 프로세스에서 쓰리프트를 통해 하이브에 명령을 보낼 수 있습니다.
- Metastore
  - ![image](https://user-images.githubusercontent.com/47103479/217844466-9ef45e3a-f633-4797-b70a-72eea5b827d5.png)
    - https://hive.apache.org/
  - 하이브는 HDFS에 저장된 데이터(디렉터리/파일)에 구조(스키마)를 입히는 방식으로 데이터를 테이블로 구조화시킵니다. 테이블 스키마와 같은 메타데이터는 메타스토어라 불리는 데이터베이스에 저장됩니다. 
  - 다중 클라이언트를 지원하기 위해서 하이브 외부에 메타스토어를 구동하는 서비스
  - 메타스토어 RDBMS는 Apache Hive 메타데이터의 중앙 리포지토리입니다. 스키마 정보(예: 열 이름, 데이터 유형, 데이터 파일 위치, 파티션, 기타 메타데이터)를 포함하는 테이블 메타데이터를 저장합니다. 
  - 하이브 메타데이터의 핵심 저장소, 메타스토어는 서비스와 데이터 보관 저장소로 나뉩니다.
  - ![image](https://user-images.githubusercontent.com/47103479/217844598-d3c3dc60-b893-47bc-9354-c131e82cba35.png)
    - https://wikidocs.net/28353
    - 내장형 메타 스토어(embedded metastore) 설정 
      - 메타데이터 서비스는 하이브 서비스와 동일한 JVM에서 실행되고 로컬 디스크에 저장되는 내장형 더비(Derby)데이터베이스 인스턴스를 포함합니다. 내장형 더비 데이터베이스 인스턴스는 한번에 디스크에 위치한 데이터베이스 파일 하나에만 접근할 수 있습니다. 
    - 로컬 메타스토어(local metastore) 
      - 다중 세션, 즉 다중 사용자를 지원하는 방법은 독립형 데이터베이스를 사용하는 것입니다. 
    - 원격 메타스토어(remote metastore) 
      - 하나 이상의 메타스토어 서버가 하이브 서비스와 별도의 프로세스로 실행됨, 데이터베이스 계층이 방화벽의 역할을 대신하고, 따라서 클라이언트는 데이터베이스 자격 증명을 더 이상 얻을 필요가 없기 때문에 관리성과 보안성이 더 높아집니다. 
- Hiveserver(하이브 서버)
  - 별도의 프로세스에서 쓰리프트 연결을 위해 대기하고 있는 데몬
- YARN
  - ![image](https://user-images.githubusercontent.com/47103479/217845044-f7807b30-1912-45fe-aa8c-243fac7934aa.png)
    - https://www.edureka.co/blog/hadoop-yarn-tutorial/
  - YARN을 사용하면 그래프 처리, 대화형 처리, 스트림 처리, 일괄 처리와 같은 다양한 데이터 처리 방법을 통해 HDFS에 저장된 데이터를 실행하고 처리할 수 있음
  - 따라서 YARN은 Hadoop을 MapReduce 이외의 다른 유형의 분산 애플리케이션에 개방함
  - 리소스 관리 외에도 YARN은 작업 예약도 수행
  - YARN은 리소스를 할당하고 작업을 예약하여 모든 처리 활동을 수행함
  - 구성요소
    - 리소스 관리자 :  마스터 데몬에서 실행되며 클러스터의 리소스 할당을 관리함
    - 노드 관리자: 슬레이브 데몬에서 실행되며 모든 단일 데이터 노드에서 작업 실행을 담당함 
    - 애플리케이션 마스터:  개별 애플리케이션의 사용자 작업 수명 주기 및 리소스 요구 사항을 관리함. Node Manager와 함께 작동하며 작업 실행을 모니터링함
    - 컨테이너: 단일 노드에 있는 RAM, CPU, 네트워크, HDD 등의 리소스 패키지
- 맵리듀스(MapReduce) 
  - 데이터 컬렉션 각 요스를 한 형태에서 다른 형태로 맵핑(맵 단계)하고 이 맵핑된 데이터 컬렉션을 하나의 값 또는 더 작은 컬렉션으로 줄이는 (리듀스 단계) 데이터 연산에 기반을 둔 계산 패러다임으로 구글에서 개발했습니다.
  - 맵리듀스는 맵과 리듀스 단계를 태스크로 나누고 클러스터에 이 태스크를 분산시킴으로써 수평 확장된 계산이 가능하도록 설계되었습니다.
  - 하둡에서 제공하는 맵리듀스 런타임은 잡을 태스크로 분리하고 클러스터에 태스크를 분산시키며 태스크의 입력 데이터를 가지고 있는 장비로 해당 태스크를 이동시키거나 필요에 따라서는 데이터를 태스크로 이동시킵니다. 실패한 태스크를 자동으로 재실행하고 에러를 복구하거나 로깅 서비스를 제공하기도 합니다.
  - 데이터 읽기
    - 맵리듀스는 입력 데이터를 읽을 때 InputFormat 자바 클래스를 이용합니다. 이 클래스는 HDFS에서 데이터를 바로 읽습니다. HBase나 카산드라 혹은 기타 데이터 소스로부터 데이터를 읽는 InputFormat 구현체도 있습니다.
    - 데이터를 섹션(section)으로 나누는 방법을 정의해 맵리듀스의 맵 태스크에 병렬 처리를 제공합니다. 두 번째는 RecordReader를 제공합니다. 이 클래스는 맵리듀스가 입력 소스로부터 레코드를 읽어 맵 태스크가 사용할 수 있는 형태인 키와 값 형태로 변환시키는 데 사용됩니다.
    - HCatalog는 맵리듀스 사용자가 하이브 데이터 웨어하우스에 저장되어 있는 데이터를 읽을 수 있게 HCatInputFormat을 제공합니다. 사용자는 필요한 테이블의 파티션과 컬럼을 읽을 수 있고 편한 리스트 형태로 레코드를 제공합니다. 사용자는 데이터를 파싱할 필요가 없습니다.
  - 데이터 쓰기
    - 한 번에 한 개 이상의 파티션을 쓸 수 있습니다. 이를 동적 파티셔닝(dynamic partitioning)이라 합니다. 레코드가 런타임에 동적으로 파티셔닝 되기 때문입니다.
