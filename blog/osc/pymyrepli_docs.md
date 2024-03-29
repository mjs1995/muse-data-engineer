# [OSSCA](https://www.contribution.ac/2023-ossca) 오픈소스 컨트리뷰션 아카데미
- ![image](https://github.com/mjs1995/muse-data-engineer/assets/47103479/064d2377-fe5d-4e31-8893-24dbfbb8ded7)
- 오픈소스 컨트리뷰션 아카데미는 언어, 개발문화, 시작의 두려움으로 인해 높게만 느껴지던 오픈소스에 대한 진입장벽을 허물고 선배 개발자와 함께 서로의 컨트리뷰톤을 응원하며 참여,오픈,공유,협업하는 오픈소스 문화를 직접 경험할 수 있는 멘토링 프로그램입니다.
- <img width="716" alt="image" src="https://github.com/mjs1995/muse-data-engineer/assets/47103479/f8d0934c-3820-4f81-87e4-ca4dfe34fb0e">
  
  - https://www.oss.kr/ossca_23_projects/show/2d3fc657-7d79-404c-8fa7-84db9c5d4d4d
- 2023 오픈소스 컨트리뷰션 아카데미를 참가하면서 python-mysql-repllication프로젝트의 멘티로 참가하게 되었습니다. 약 3달간의 여정으로 온/오프라인 모임을 통해서 오픈소스 기여를 하였습니다.
- 참여 계기는 데이터 엔지니어링에 관심이 있는 다양한 분들과 협업을 하면서 개발 문화와 오픈소스에 기여하는 방식을 보고자 지원을 하였습니다. 또한 프로젝트 운영을 하면서 binlog 파일에 대해 이슈가 있었고 CDC에 관심이 있어서 세미나를 다니던 중 python-mysql-repllication의 분야가 핏이 맞아 신청을 하였습니다.

## [python-mysql-replication](https://github.com/julien-duponchelle/python-mysql-replication) 프로젝트 소개
### MySQL 데이터베이스에서 데이터 추출하는 두 가지 방법
- SQL을 사용한 전체 또는 증분 추출
  - 구현하기가 훨씬 간단하지만 자주 변경되는 대규모 데이터세트에서는 확장성이 떨어지며 전체 추출과 증분 추출 사이에도 트레이드오프가 있습니다.
  - 증분 추출이 최적의 성능에 이상적이지만 어떤 테이블에 대해서는 가능하지 않을 수 있는 몇 가지 단점과 이유가 있습니다.
    - 삭제된 행은 캡처되지 않음
    - 원본 테이블에는 마지막으로 업데이트된 시간에 대한 신뢰할 수 있는 타임스탬프가 있어야 함
- 이진 로그(binlog) 복제 (스트리밍 데이터 수집을 수행하는 하나의 경로)
  - 이진 로그 복제는 구현이 더 복잡하지만 원본 테이블의 변경되는 데이터 볼륨이 크거나 MySQL 소스에서 데이터를 더 자주 수집해야 하는 경우에 더 적합합니다.
  - MySQL 이진 로그는 데이터베이스에서 수행된 모든 작업에 대한 기록을 보관하는 로그입니다.
    - 구성된 방식에 따라 모든 테이블의 생성 또는 수정 사항뿐만 아니라 모든 INSERT, UPDATE 및 DELETE 작업도 기록됩니다.
    - 이진 로그의 원래 목적은 다른 MySQL 인스턴스로 데이터를 복제하기 위한 것이지만, 이진 로그의 내용은 데이터 웨어하우스로 데이터를 수집하려는 데이터 엔지니어에게 매우 매력적입니다.
### python-mysql-replication
- 이진 로그 복제에 해당되며 MySQL 복제 프로토콜의 순수한 Python 구현입니다. 이것은 PyMYSQL 위에 구축되어 있으며, 이를 통해 데이터와 원시 SQL 쿼리를 가진 이벤트(예: 삽입, 업데이트, 삭제)를 수신할 수 있습니다.
- ![diagrams_image_graph](https://github.com/mjs1995/muse-data-engineer/assets/47103479/a6a514ed-9780-460c-9e16-00090bb9947d)
- master-slave 구조에서 마스터가 슬레이브에게 전송하는 데이터를 파싱하여 Python 객체를 생성합니다. 데이터의 종류에 따라 각기 다른 방법으로 파싱하고, 다른 종류의 Event 객체로 저장합니다.
- 주요 특징
  - MySQL 복제 프로토콜의 Python 구현: MySQL의 복제 프로토콜을 사용하여 데이터베이스의 변경 사항을 실시간으로 추적할 수 있습니다.
  - 이벤트 수신: 라이브러리를 사용하면 데이터베이스에서 발생하는 삽입, 업데이트, 삭제와 같은 이벤트를 실시간으로 수신할 수 있습니다.
  - 원시 SQL 쿼리: 라이브러리를 통해 데이터베이스에서 실행되는 원시 SQL 쿼리를 수신할 수도 있습니다.
- 사용 사례
  - MySQL to NoSQL 데이터베이스 복제: MySQL 데이터베이스의 변경 사항을 NoSQL 데이터베이스(예: MongoDB, Cassandra)로 복제하는 데 사용됩니다.
  - MySQL to 검색 엔진 복제: MySQL 데이터베이스의 변경 사항을 검색 엔진(예: Elasticsearch)으로 복제하는 데 사용됩니다.
  - 캐시 무효화: 데이터베이스에서 변경 사항이 발생할 때 캐시(예: Redis, Memcached)를 무효화하는 데 사용됩니다.
  - 감사: 데이터베이스의 모든 변경 사항을 추적하고 기록하여 데이터베이스 활동을 감사하는 데 사용됩니다.
  - 실시간 분석: 데이터베이스의 실시간 변경 사항을 분석하는 데 사용됩니다.
- 환경 설정 
  - binlog 구성과 관련하여 MySQL 데이터베이스에서 확인해야 할 두 가지 주요 설정이 있습니다. 먼저 바이너리 로깅이 활성화되어 있는지 확인해야 합니다.
  - ```shell
    mysql> SELECT variable_value as bin_log_status
        -> FROM performance_schema.global_variables
        -> WHERE variable_name='log_bin';
    +----------------+
    | bin_log_status |
    +----------------+
    | ON             |
    +----------------+
    1 row in set (0.01 sec)
    ```
  - 그다음은 바이너리 로깅 형식이 적절하게 설정되었는지 확인해야 합니다.
    - ```shell
      mysql> SELECT variable_value as bin_log_format
          -> FROM performance_schema.global_variables
          -> WHERE variable_name = 'binlog_format';
      +----------------+
      | bin_log_format |
      +----------------+
      | ROW            |
      +----------------+
      1 row in set (0.00 sec)
      ```
    - 이진 로깅 형식
      - STATEMENT : 이진 로그에 행을 삽입하거나 수정하는 행동들에 대해 SQL 문 자체를 기록함, 하나의 MySQL 데이터베이스에서 다른 MySQL 데이터베이스로 데이터를 복제하려는 경우 이 형식이 유용합니다
      - ROW : 테이블의 행에 대한 모든 변경 사항이 SQL문이 아니라 행 자체의 데이터로 이진 로그 행에 표시됨, 이것이 주로 사용하는 기본 형식
      - MIXED : 이진 로그에 STATE 형식 레코드와 ROW 형식 레코드를 모두 기록하며 나중에 ROW 데이터만 걸러낼 수도 있지만, 이진 로그를 다른 용도로 사용하지 않는 한 결국 디스크 공간을 추가로 사용하게 되기 때문에 MIXED를 활성화할 필요는 없습니다.
  - binlog 형식과 기타 구성 설정은 일반적으로 MySQL 데이터베이스 인스턴스와 관련된 my.cnf 파일에 설정됩니다. 파일을 열면 다음과 같은 행이 포함된 것을 볼 수 있습니다.
    - ```shell
      [mysqld]
      binlog_format=row
      ........
      ```
  - 환경 설정이 끝나면 python-mysql-replication 라이브러리를 활용해서 MySQL 서버의 기본 binlog 파일을 읽습니다. 기본 binlog 파일 이름과 경로는 MySQL 데이터베이스의 my.cnf 파일에 저장된 log_bin 변수에 설정됩니다. 

### 복제 데이터 포맷 
- MySQL에서는 실행된 SQL문을 바이너리 로그에 기록하는 Statement 방식과 변경된 데이터 자체를 기록하는 Row 방식
- `Row` 기반 바이너리 로그 포맷
  - MySQL 서버에서 데이터 변경이 발생했을 때 변경된 값 자체가 바이너리 로그에 기록되는 방식
  - 어떤 형태의 쿼리가 실행됐든 간에 복제 시 소스 서버와 레플리카 서버의 데이터를 일관되게 하는 가장 안전한 방식
  - Row 포맷에서는 소스 서버에서 실행된 쿼리가 UUID(), USER() 등과 같은 비확정적 함수를 사용했다 하더라도 레플리카 서버에서 똑같이 이 함수가 다시 실행되는 것이 아니라 함수의 결괏값을 전달받아 처리되므로 이 같은 경우에 있어서도 안전하게 복제가 가능합니다.
  - 변경된 데이터가 그대로 바이너리 로그에 기록된다는 것은 가장 큰 장점이자 단점, MySQL 서버에서 실행된 쿼리가 굉장히 많은 데이터를 변경하는 경우에는 변경된 데이터가 전부 기록되므로 바이너리 로그 파일 크기가 단시간에 매우 커질 수 있습니다. 변경된 데이터 수가 적더라도 BLOB 형태의 큰 값이 새로 저장되거나 변경되는 경우에는 마찬가지로 파일 크기가 많이 커질 수 있음을 유의해야 합니다.
  - Row 포맷을 사용하는 경우 바이너리 로그에 쿼리로 인해 변경된 데이터들이 전부 저장되기 때문에 Statement 포맷보다 더 많은 저장 공간과 네트워크 트래픽을 유발할 가능성이 있습니다.
- `Statement` 기반 바이너리 로그 포맷
  - 변경 이벤트에 대해 이벤트를 발생시킨 SQL문을 바이너리 로그에 기록하는 방식
  - Statement 포맷을 사용하면 손쉽게 SQL문들을 확인할 수 있으므로 감사에 더 용이합니다.
  - 대표적인 단점은 비확정적(Non-Deterministic)으로 처리될 수 있는 쿼리가 실행된 경우 Statement 포맷에서는 복제 시 소스 서버와 레플리카 서버 간에 데이터가 달라질 수 있다는 점, 다른 단점은 Row 포맷으로 복제될 때보다 데이터에 락을 더 많이 건다는 점
  - Statement 포맷 기반 복제에서 소스 서버와 레플리카 서버 간의 데이터 일관성을 해칠 수 있는 비확정적 쿼리 유형의 몇 가지 예
    - DELETE/UPDATE 쿼리에서 OREDER BY 절 없이 LIMIT 사용
    - SELECT ... FOR UPDATE 및 SELECT ... FOR SHARE 쿼리에서 NOWAIT이나 SKIP LOCKED 옵션을 사용
    - LOAD_FILE(), UUID(), UUID_SHORD(), USER(), FOUND_ROWS(), RAND(), VERSION() 등과 같은 함수를 사용하는 쿼리
    - 동일한 파라미터 값을 입력하더라도 결괏값이 달라질 수 있는 사용자 정의 함수(User-defined Function)나 스토어드 프로시저(Stored Procedure)를 사용하는 쿼리
  - Statement 기반 바이너리 로그 포맷은 사용할 때 한 가지 제한사항이 있는데, 바로 트랜잭션 격리 수준이 반드시 REPEATABLE-READ 이상이어야 한다는 점
- `Mixed` 포맷
  - MySQL 서버는 MIXED 포맷으로 설정되면 기본적으로는 Statement 포맷을 사용하며, 실행된 쿼리와 스토리지 엔진의 종류에 따라 필요시 자동으로 Row 포맷으로 전환해서 로그에 기록합니다.

### 예제
- [show master status](https://mariadb.com/kb/en/show-binlog-status/) : 원본 서버의 바이너리 로그 파일에 대한 상태 정보를 확인할 수 있습니다. 
  - ```shell
    mysql> show master status;
    +---------------+----------+--------------+------------------+-------------------+
    | File          | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
    +---------------+----------+--------------+------------------+-------------------+
    | binlog.000001 |      157 |              |                  |                   |
    +---------------+----------+--------------+------------------+-------------------+
    1 row in set (0.00 sec)
    ```
- 예제 쿼리를 실행합니다.
  - ```sql
    CREATE DATABASE test;
    use test;
    CREATE TABLE test4 (id int NOT NULL AUTO_INCREMENT, data VARCHAR(255), data2 VARCHAR(255), PRIMARY KEY(id));
    INSERT INTO test4 (data,data2) VALUES ("Hello", "World");
    UPDATE test4 SET data = "World", data2="Hello" WHERE id = 1;
    DELETE FROM test4 WHERE id = 1;
    ```
- [show binlog events](https://mariadb.com/kb/en/show-binlog-events/) : 바이너리 로그의 이벤트를 확인할 수 있습니다.
  - ```shell
    mysql> show binlog events in 'binlog.000001';
    +---------------+------+----------------+-----------+-------------+--------------------------------------------------------------------------------------------------------------------------------------+
    | Log_name      | Pos  | Event_type     | Server_id | End_log_pos | Info                                                                                                                                 |
    +---------------+------+----------------+-----------+-------------+--------------------------------------------------------------------------------------------------------------------------------------+
    | binlog.000001 |    4 | Format_desc    |         1 |         126 | Server ver: 8.0.33, Binlog ver: 4                                                                                                    |
    | binlog.000001 |  126 | Previous_gtids |         1 |         157 |                                                                                                                                      |
    | binlog.000001 |  157 | Anonymous_Gtid |         1 |         234 | SET @@SESSION.GTID_NEXT= 'ANONYMOUS'                                                                                                 |
    | binlog.000001 |  234 | Query          |         1 |         342 | CREATE DATABASE test /* xid=12 */                                                                                                    |
    | binlog.000001 |  342 | Anonymous_Gtid |         1 |         421 | SET @@SESSION.GTID_NEXT= 'ANONYMOUS'                                                                                                 |
    | binlog.000001 |  421 | Query          |         1 |         616 | use `test`; CREATE TABLE test4 (id int NOT NULL AUTO_INCREMENT, data VARCHAR(255), data2 VARCHAR(255), PRIMARY KEY(id)) /* xid=17 */ |
    | binlog.000001 |  616 | Anonymous_Gtid |         1 |         695 | SET @@SESSION.GTID_NEXT= 'ANONYMOUS'                                                                                                 |
    | binlog.000001 |  695 | Query          |         1 |         770 | BEGIN                                                                                                                                |
    | binlog.000001 |  770 | Table_map      |         1 |         832 | table_id: 131 (test.test4)                                                                                                           |
    | binlog.000001 |  832 | Write_rows     |         1 |         886 | table_id: 131 flags: STMT_END_F                                                                                                      |
    | binlog.000001 |  886 | Xid            |         1 |         917 | COMMIT /* xid=18 */                                                                                                                  |
    | binlog.000001 |  917 | Anonymous_Gtid |         1 |         996 | SET @@SESSION.GTID_NEXT= 'ANONYMOUS'                                                                                                 |
    | binlog.000001 |  996 | Query          |         1 |        1080 | BEGIN                                                                                                                                |
    | binlog.000001 | 1080 | Table_map      |         1 |        1142 | table_id: 131 (test.test4)                                                                                                           |
    | binlog.000001 | 1142 | Update_rows    |         1 |        1216 | table_id: 131 flags: STMT_END_F                                                                                                      |
    | binlog.000001 | 1216 | Xid            |         1 |        1247 | COMMIT /* xid=19 */                                                                                                                  |
    | binlog.000001 | 1247 | Anonymous_Gtid |         1 |        1326 | SET @@SESSION.GTID_NEXT= 'ANONYMOUS'                                                                                                 |
    | binlog.000001 | 1326 | Query          |         1 |        1401 | BEGIN                                                                                                                                |
    | binlog.000001 | 1401 | Table_map      |         1 |        1463 | table_id: 131 (test.test4)                                                                                                           |
    | binlog.000001 | 1463 | Delete_rows    |         1 |        1517 | table_id: 131 flags: STMT_END_F                                                                                                      |
    | binlog.000001 | 1517 | Xid            |         1 |        1548 | COMMIT /* xid=20 */                                                                                                                  |
    +---------------+------+----------------+-----------+-------------+--------------------------------------------------------------------------------------------------------------------------------------+
    21 rows in set (0.00 sec)
    ```
- python-mysql-replication의 BinLogStreamReader
  - binlog 파일의 이벤트들을 python-mysql-replication 라이브러리를 이용해서 파싱합니다. 
  - ```shell
    WARNING:root:
                       Setting The Variable Value BINLOG_ROW_METADATA = FULL, BINLOG_ROW_IMAGE = FULL. 
                       By Applying this, provide properly mapped column information on UPDATE,DELETE,INSERT. 
                        
    === RotateEvent ===
    Position: 4
    Next binlog file: binlog.000001
    
    === FormatDescriptionEvent ===
    Date: 2023-09-26T13:18:10
    Log position: 126
    Event size: 99
    Read bytes: 52
    Binlog version: 4
    MySQL version: 8.0.33
    
    === PreviousGtidsEvent ===
    Date: 2023-09-26T13:18:10
    Log position: 157
    Event size: 8
    Read bytes: 8
    previous_gtids: 
    
    === QueryEvent ===
    Date: 2023-09-26T13:18:17
    Log position: 342
    Event size: 85
    Read bytes: 85
    Schema: b'test'
    Execution time: 0
    Query: CREATE DATABASE test
    
    === QueryEvent ===
    Date: 2023-09-26T13:18:17
    Log position: 616
    Event size: 172
    Read bytes: 172
    Schema: b'test'
    Execution time: 0
    Query: CREATE TABLE test4 (id int NOT NULL AUTO_INCREMENT, data VARCHAR(255), data2 VARCHAR(255), PRIMARY KEY(id))
    
    === QueryEvent ===
    Date: 2023-09-26T13:18:17
    Log position: 770
    Event size: 52
    Read bytes: 52
    Schema: b'test'
    Execution time: 0
    Query: BEGIN
    
    === TableMapEvent ===
    Date: 2023-09-26T13:18:17
    Log position: 832
    Event size: 39
    Read bytes: 39
    Table id: 131
    Schema: test
    Table: test4
    Columns: 3
    
    === WriteRowsEvent ===
    Date: 2023-09-26T13:18:17
    Log position: 886
    Event size: 31
    Read bytes: 12
    Table: test.test4
    Affected columns: 3
    Changed rows: 1
    Column Name Information Flag: False
    Values:
    --
    * None : World
    
    === XidEvent ===
    Date: 2023-09-26T13:18:17
    Log position: 917
    Event size: 8
    Read bytes: 8
    Transaction ID: 18
    
    === QueryEvent ===
    Date: 2023-09-26T13:18:17
    Log position: 1080
    Event size: 61
    Read bytes: 61
    Schema: b'test'
    Execution time: 0
    Query: BEGIN
    
    === TableMapEvent ===
    Date: 2023-09-26T13:18:17
    Log position: 1142
    Event size: 39
    Read bytes: 39
    Table id: 131
    Schema: test
    Table: test4
    Columns: 3
    
    === UpdateRowsEvent ===
    Date: 2023-09-26T13:18:17
    Log position: 1216
    Event size: 51
    Read bytes: 13
    Table: test.test4
    Affected columns: 3
    Changed rows: 1
    Column Name Information Flag: False
    Values:
    --
    *None:World=>Hello
    
    === XidEvent ===
    Date: 2023-09-26T13:18:17
    Log position: 1247
    Event size: 8
    Read bytes: 8
    Transaction ID: 19
    
    === QueryEvent ===
    Date: 2023-09-26T13:18:17
    Log position: 1401
    Event size: 52
    Read bytes: 52
    Schema: b'test'
    Execution time: 0
    Query: BEGIN
    
    === TableMapEvent ===
    Date: 2023-09-26T13:18:17
    Log position: 1463
    Event size: 39
    Read bytes: 39
    Table id: 131
    Schema: test
    Table: test4
    Columns: 3
    
    === DeleteRowsEvent ===
    Date: 2023-09-26T13:18:17
    Log position: 1517
    Event size: 31
    Read bytes: 12
    Table: test.test4
    Affected columns: 3
    Changed rows: 1
    Column Name Information Flag: False
    Values:
    --
    * None : Hello
    
    === XidEvent ===
    Date: 2023-09-26T13:18:17
    Log position: 1548
    Event size: 8
    Read bytes: 8
    Transaction ID: 20
    ```
- [mysqlbinlog](https://dev.mysql.com/doc/refman/8.0/en/mysqlbinlog.html) : MySQL 서버의 바이너리 로그 파일의 내용을 텍스트 형식으로 표시합니다.
  - homebrew로 mysql을 설치할 경우 binlog 파일의 위치는 `/opt/homebrew/var/mysql`에 있습니다.
  - ```shell
    mysqlbinlog binlog.000001
    # The proper term is pseudo_replica_mode, but we use this compatibility alias
    # to make the statement usable on server versions 8.0.24 and older.
    /*!50530 SET @@SESSION.PSEUDO_SLAVE_MODE=1*/;
    /*!50003 SET @OLD_COMPLETION_TYPE=@@COMPLETION_TYPE,COMPLETION_TYPE=0*/;
    DELIMITER /*!*/;
    # at 4
    #230926 22:24:51 server id 1  end_log_pos 126 CRC32 0x47639f18 	Start: binlog v 4, server v 8.0.33 created 230926 22:24:51 at startup
    # Warning: this binlog is either in use or was not closed properly.
    ROLLBACK/*!*/;
    BINLOG '
    o9sSZQ8BAAAAegAAAH4AAAABAAQAOC4wLjMzAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
    AAAAAAAAAAAAAAAAAACj2xJlEwANAAgAAAAABAAEAAAAYgAEGggAAAAICAgCAAAACgoKKioAEjQA
    CigAARifY0c=
    '/*!*/;
    # at 126
    #230926 22:24:51 server id 1  end_log_pos 157 CRC32 0xe790b0ec 	Previous-GTIDs
    # [empty]
    SET @@SESSION.GTID_NEXT= 'AUTOMATIC' /* added by mysqlbinlog */ /*!*/;
    DELIMITER ;
    # End of log file
    /*!50003 SET COMPLETION_TYPE=@OLD_COMPLETION_TYPE*/;
    /*!50530 SET @@SESSION.PSEUDO_SLAVE_MODE=0*/;    
    ```
    
## Reference 
- [MySQL 8.0 docs](https://dev.mysql.com/doc/refman/8.0/en/)
- [데이터 파이프라인 핵심 가이드](https://product.kyobobook.co.kr/detail/S000001766501)
- [MySQL 8.0 2권](https://product.kyobobook.co.kr/detail/S000001766483)

# PR 내용 
- <img width="16" alt="git-merged" src="https://github.com/mjs1995/muse-data-engineer/assets/47103479/1199bcfb-b94f-4ff3-89cc-9d271da95dac"> [docs: Update README to add Featured Books](https://github.com/julien-duponchelle/python-mysql-replication/pull/413)
- <img width="16" alt="git-merged" src="https://github.com/mjs1995/muse-data-engineer/assets/47103479/1199bcfb-b94f-4ff3-89cc-9d271da95dac"> [Add Featured Section in README](https://github.com/julien-duponchelle/python-mysql-replication/pull/457)
- <img width="16" alt="git-merged" src="https://github.com/mjs1995/muse-data-engineer/assets/47103479/1199bcfb-b94f-4ff3-89cc-9d271da95dac"> [Update README Featured Section with AWS Blog on RDS, XA Transactions](https://github.com/julien-duponchelle/python-mysql-replication/pull/485)
- <img width="16" alt="git-merged" src="https://github.com/mjs1995/muse-data-engineer/assets/47103479/1199bcfb-b94f-4ff3-89cc-9d271da95dac"> [Remove duplicated Affected columns output in UpdateRowsEvent](https://github.com/julien-duponchelle/python-mysql-replication/pull/478)
- <img width="16" alt="git-merged" src="https://github.com/mjs1995/muse-data-engineer/assets/47103479/1199bcfb-b94f-4ff3-89cc-9d271da95dac"> [Developed UserVarEvent and Added Statement-Based Logging Test](https://github.com/julien-duponchelle/python-mysql-replication/pull/466)
- <img width="16" alt="git-merged" src="https://github.com/mjs1995/muse-data-engineer/assets/47103479/1199bcfb-b94f-4ff3-89cc-9d271da95dac"> [Enhance Testing with MySQL8 & Update GitHub Actions](https://github.com/julien-duponchelle/python-mysql-replication/pull/484)
- <img width="16" alt="git-merged" src="https://github.com/mjs1995/muse-data-engineer/assets/47103479/1199bcfb-b94f-4ff3-89cc-9d271da95dac"> [Modify test structure](https://github.com/julien-duponchelle/python-mysql-replication/pull/502)
- 문서기여, 버그수정, 기능개발 등 기여를 하였고 개인적인 기여 난이도를 기준으로 말씀드리려고 합니다.

## 문서기여 
- [데이터 파이프라인 핵심 가이드](https://product.kyobobook.co.kr/detail/S000001766501) 책을 읽었을 때 CDC 부분과 binlog에 대해서 관심이 있었던 부분이 있어서 읽을 당시에는 몰랐지만 이번에 OSSCA를 진행하면서 python-mysql-replication이 사용되었다는 것을 알게 되었습니다. 관련하여 멘토님과 얘기를 하다가 프로젝트 주인장분이 프로젝트의 쓰임새에 대해서 관심이 많아서 알려드리면서 README를 업데이트를 하는 건 어떠냐고 물어보았습니다.
- ![image](https://github.com/mjs1995/muse-data-engineer/assets/47103479/82ab183b-0242-4e6c-9c48-6f02ecab4f45)
- O'REILLY에 소개되었던 프로젝트에 대해서 긍정적으로 반응을 하셔서 첫 프로젝트 PR은 [docs: Update README to add Featured Books](https://github.com/julien-duponchelle/python-mysql-replication/pull/413)로 시작을 하게 되었습니다.
- 프로젝트를 진행하면서 책에서 소개된 내용뿐만 아니라 공신력 있는 기술블로그에 대한 소개 내용도 관심이 있어서 AWS의 사례에 대해서 추가를 하게 되었습니다. 최근에 [AWS Summit 2023](https://aws.amazon.com/ko/events/summits/seoul/)과 [AWS Data Roadshow 2023](https://pages.awscloud.com/aws-data-roadshow-2023-reg.html?trk=a063bb01-f7a1-443f-ab0d-b541b793a721&sc_channel=em) 세미나를 다녀오면서 기술블로그와 aws-sample 깃허브가 생각이 났었고 저희 프로젝트에 대해 소개를 하고 있는 내용이 있어서 위 내용 모두 PR을 올렸습니다.
  - [Add Featured Section in README](https://github.com/julien-duponchelle/python-mysql-replication/pull/457)
  - [Update README Featured Section with AWS Blog on RDS, XA Transactions](https://github.com/julien-duponchelle/python-mysql-replication/pull/485)
- ![diagrams_image](https://github.com/mjs1995/muse-data-engineer/assets/47103479/89422be7-d410-4ffb-a74f-b4d902adda54)
- 이를 바탕으로 python-mysql-replicatino은 여러 기업에서 사용하고 있지만 snowflake 기술 블로그나 AWS에 소개된 내용으로 아키텍처를 그려보았습니다.

## 버그 수정 
- python-mysql-replication에서 이벤트 구현에 대한 기능개발을 하고 있을 때 UpdateRowsEvent에 대해서 Affected columns이 중복되어서 나타나고 있었습니다. 커밋이력을 확인한 뒤에 소스코드를 분석하였습니다.
- ```markdown
  BinLogEvent
  |
  └── RowsEvent
      |
      └── UpdateRowsEvent
  ```
- RowsEvent는 BinLogEvent를 상속받아 행 기반 이벤트를 처리하고 UpdateRowsEvent는 RowsEvent를 상속받아 업데이트 이벤트를 처리하는데 [_dump 메소드](https://github.com/23-OSSCA-python-mysql-replication/python-mysql-replication/blob/08219f740edda3923ec866ebeb92fa580e3cc571/pymysqlreplication/row_event.py#L468C9-L468C14)에서 "Affected columns" 상속되는 게 중복으로 출력이 되고 있어서 PR을 올렸습니다.
  - [Remove duplicated Affected columns output in UpdateRowsEvent](https://github.com/julien-duponchelle/python-mysql-replication/pull/478)
 
## 기능개발
### MySQL8 버전 테스트 추가 및 github action 버전 업그레이드 
- 프로젝트 진행 당시 MySQL8에 대한 테스트를 할 수 없었고 MySQL5.7 버전과 MariaDB에 대해서만 진행을 할 수 있었으며 [MySQL 8.0.14](https://dev.mysql.com/doc/relnotes/mysql/8.0/en/news-8-0-14.html#:~:text=Replication%3A%20Character) 버전 이후로 binlog_row_metadata=FULL이 나오면서 [OptionalMetaData에 대해서 이벤트 개발](https://github.com/julien-duponchelle/python-mysql-replication/pull/471)을 하였습니다. 관련해서는 [같은 조의 멘티님](https://seanical.tistory.com/5)의 블로그를 읽어보시면 도움 되실 거 같습니다. 따라서 MySQL8 버전대에 대해서 도커파일을 추가하고 테스트 코드를 수정해야 했습니다.
- ```yaml
  version: '3.4'

  x-mysql: &mysql
    environment:
      MYSQL_ALLOW_EMPTY_PASSWORD: true
    command: >
      mysqld
      --log-bin=mysql-bin.log
      --server-id 1
      --binlog-format=row
      --gtid_mode=on
      --enforce-gtid-consistency=on
  
  x-mariadb: &mariadb
    environment:
      MARIADB_ALLOW_EMPTY_ROOT_PASSWORD: 1
    command: >
      --log-bin=master-bin
      --server-id=1
      --default-authentication-plugin=mysql_native_password
      --binlog-format=row
  
  services:
    percona-5.7:
      <<: *mysql
      image: percona:5.7
      ports:
        - "3306:3306"
  
    percona-5.7-ctl:
      <<: *mysql
      image: percona:5.7
      ports:
        - "3307:3306"
  
    percona-8.0:
      <<: *mysql
      image: percona:8.0
      ports:
        - "3309:3306"
  
    mariadb-10.6:
      <<: *mariadb
      image: mariadb:10.6
      ports:
        - "3308:3306"
      volumes:
        - type: bind
          source: ./.mariadb
          target: /opt/key_file
        - type: bind
          source: ./.mariadb/my.cnf
          target: /etc/mysql/my.cnf
  ```
  - [도커 컴포스 파일](https://docs.docker.com/compose/reference/#command-options-overview-and-help)은 여러 개의 컨테이너 구성 정보를 코드로 정의하고, 명령을 실행함으로써 애플리케이션의 실행환경을 구성하는 컨테이너들을 일원 관리하기 위한 툴로 docker-compose.yml 코드를 보면 [percona:8.0 이미지](https://hub.docker.com/_/percona)를 추가하였고 [percona 8](https://github.com/percona/percona-docker/blob/3f666ccdf6a9eed0e0505723fbe8b4954a105c99/percona-server-8.0/Dockerfile)은 gituhb에서 확인해 보면 최신 버전을 가져오고 있는 거 같습니다. (Percona Server 8.0.33 버전과 MySQL Shell 8.0.33 버전을 포함하고 있음)
  - 해당 코드는 [YAML Anchors와 Aliases](https://yaml.org/spec/1.2.2/#3222-anchors-and-aliases)를 사용하였는데 이는 중복된 데이터 구조나 값을 재사용하기 위한 메커니즘이며, 이를 통해 파일의 크기를 줄이고 데이터의 중복을 최소화하며 파일의 가독성을 향상했습니다. [도커 앵커](https://docs.docker.com/compose/compose-file/10-fragments/)에서 사례를 확인할 수 있습니다.
    - Anchors (&): 특정 노드에 이름을 붙여 재사용할 수 있게 합니다.
    - Aliases (*): 이전에 정의한 앵커의 값을 재사용할 수 있습니다.
    - <<: 위에서 정의된 앵커를 참조합니다.
  - 테스트 케이스 추가
    - docker-compose 파일에서 설정한 환경 변수와 포트 값을 사용하여 데이터베이스에 연결할 수 있게끔 신규 클래스를 만들었고 복제 기능 테스트케이스를 추가하였습니다.
- [깃허브 액션 버전 업그레이드](https://github.com/julien-duponchelle/python-mysql-replication/pull/484)
  - ```yaml
    name: PyTest
    on: [push, pull_request]
    
    jobs:
      test:
        strategy:
          fail-fast: false
          matrix:
            include:
              - {name: 'CPython 3.7', python: '3.7'}
              - {name: 'CPython 3.11', python: '3.11'}
              - {name: 'Pypy 3.7', python: 'pypy-3.7'}
              - {name: 'Pypy 3.9', python: 'pypy-3.9'}
        name: ${{ matrix.name }}
        runs-on: ubuntu-latest
        timeout-minutes: 2
    
        steps:
          - name: Check out code
            uses: actions/checkout@v2
    
          - name: Setup Python
            uses: actions/setup-python@v2
            with:
              python-version: ${{ matrix.python }}
      ```
  - 현재 pytest github action에서 사용되고 있는 workflow 액션은 actions/checkout@v2과 actions/setup-python@v2 입니다. 여기서 Node 16의 기본 런타임은 2023년 9월 11일에 지원이 종료될 예정으로 액션을 실행할 때마다 경고 문구가 나왔습니다. 따라서 GitHub Actions 버전의 기능과 최적화를 활용하기 위해 최신버전으로 업데이트를 진행하였습니다.
    - <img width="1023" alt="image" src="https://github.com/mjs1995/muse-data-engineer/assets/47103479/67ce401f-8f41-4bc8-b802-e3c234fad7c2">
    - [actions/checkout@v4](https://github.com/actions/checkout/releases) : GitHub 저장소의 코드를 체크아웃하여 GitHub Actions 실행 환경의 작업 디렉토리에 배치합니다.
    - [actions/setup-python@v4](https://github.com/actions/setup-python/releases) : 작업 환경에 특정 버전의 Python을 설치하고 설정합니다.

### github action pytest 개선
#### 단위 테스트
- 단위 테스트 프레임워크와 라이브러리
  - unittest는 파이썬의 표준 라이브러리이고 두 번째 pytest는 pip를 통해 설치해야 하는 라이브러리
  - 테스트 시나리오를 다루는 것은 unittest만으로도 충분할 것입니다. 왜냐하면 다양한 헬퍼 기능을 제공하기 때문입니다. 그러나 외부 시스템에 연결하는 등의 의존성이 많은 경우 테스트 케이스를 파라미터화할 수 있는 픽스처(fixture)라는 패치 객체가 필요하며 이렇게 보다 복잡한 옵션이 필요한 경우는 pytest가 적합합니다.
    - unittest
      - unittest 모듈은 자바의 JUnit을 기반으로 하며 JUnit은 Smaltalk의 아이디어를 기반으로 만들어졌으므로 객체 지향적입니다. 이러한 이유로 테스트는 객체를 사용해 작성되며 클래스의 시나리오별로 테스트를 그룹화하는 것이 일반적입니다.
      - 가장 일반적인 메서드는 실제 실행 값과 예상 값을 비교하는 assertEquals(<actual>, <expected>[, message])
      - 예외적인 상황이 발생하면 잘못된 가정 아래 실행을 계속하는 것보다는 예외를 발생시키고 호출자에게 바로 알려주는 것이 좋습니다. 이것이 assertRaises 메서드가 확인하려는 것입니다.
    - pytest
      - 차이점은 unittest 처럼 테스트 시나리오를 클래스로 만들고 객체 지향 모델을 생성하는 것이 가능하지만 필수 사항이 아니며, 단순히 assert 비교만으로 단위 테스트를 식별하고 결과를 보고하는 것이 가능합니다.
      - 픽스처(Fixture)
        - pytest의 가장 큰 장점 중 하나는 재사용 가능한 기능을 쉽게 만들 수 있다는 점입니다.
        - 테스트에서 사용될 때는 테스트 사전/사후에 사용 가능한 리소스 또는 모듈을 뜻합니다.
  - 더욱더 자세한 설명을 원하면 [파이썬 클린코드 2nd Edition](https://github.com/mjs1995/Book_review/blob/main/%ED%8C%8C%EC%9D%B4%EC%8D%AC%20%ED%81%B4%EB%A6%B0%EC%BD%94%EB%93%9C%202nd%20Edition.md) 책을 읽어보시는것을 추천드립니다.
#### [pytest Fixtures](https://docs.pytest.org/en/latest/reference/fixtures.html#autouse-order)
- pytest의 fixture는 테스트 코드의 재사용성을 높이기 위해 사용되며, 테스트 코드 실행 전 특정 설정이나 값을 설정할 수 있게 합니다.
- fixture는 여러 테스트에서 공용으로 사용될 수 있으며, 특정 scope 내에서만 사용가능합니다.
- pytest가 테스트를 실행하려고 할 때 3가지 요소의 픽스처 실행 순서를 고려합니다.
  - scope
  - dependencies
  - autouse
    - autouse fixture는 정의된 scope 안에 있는 테스트에만 영향을 주며 scope 내에서 먼저 실행됩니다.
  - ```python
    class PyMySQLReplicationTestCase(base):
      def ignoredEvents(self):
          return []

      @pytest.fixture(autouse=True)
      def setUpDatabase(self, get_db):
          databases = get_databases()
          # For local testing, set the get_dbms parameter to one of the following values: 'mysql-5', 'mysql-8', mariadb-10'.
          # This value should correspond to the desired database configuration specified in the 'config.json' file.
          self.database = databases[get_db]
    ```
#### [conftest.py](https://docs.pytest.org/en/6.2.x/writing_plugins.html?highlight=conftest%20py#conftest-py-local-per-directory-plugins)
- conftest.py 파일은 한 디렉토리에 있는 모든 테스트에 fixture를 제공하는 수단입니다.
- conftest.py에 정의된 fixture는 해당 패키지의 어떤 테스트에서도 import 없이 사용될 수 있습니다.(pytest가 자동으로 검색합니다.)
- 여러 conftest.py가 중첩된 디렉토리 구조를 가질 수 있으며, 각 디렉토리의 conftest.py는 상위 디렉토리의 conftest.py에서 제공하는 fixture에 추가하여 자체 fixture를 제공할 수 있습니다.
- ```python
  import pytest


  def pytest_addoption(parser):
      parser.addoption("--db", action="store", default="mysql-5")


  @pytest.fixture
  def get_db(request):
      return request.config.getoption("--db")
  ```
- [pytest_addoption](https://docs.pytest.org/en/latest/reference/reference.html#pytest.hookspec.pytest_addoption)
  - pytest_addoption은 pytest 초기화 단계에서 커맨드라인 옵션 및 설정 값을 등록하는 데 사용됩니다
  - parser.addoption(...): 커맨드라인 옵션을 등록하며 [argparse](https://docs.python.org/ko/3/library/argparse.html)의 add_argument() 함수가 받는 속성과 동일합니다
  - 플러그인 또는 테스트의 루트 디렉토리에 위치한 conftest.py 파일에서만 구현되어야 합니다.
#### pytest keyword expressions
- [pytest -k](https://docs.pytest.org/en/latest/how-to/usage.html#specifying-which-tests-to-run)
- 이 표현식은 대소문자를 구분하지 않으며, Python 연산자를 포함할 수 있습니다.
- EXPRESSION 부분에는 파일명, 클래스명, 함수명 등의 문자열을 포함하여 조건을 지정할 수 있습니다.
```shell
env:
  PYTEST_SKIP_OPTION: "not test_no_trailing_rotate_event and not test_end_log_pos and not test_query_event_latin1"

pytest -k "$PYTEST_SKIP_OPTION"
```
- test_no_trailing_rotate_event, test_end_log_pos, test_query_event_latin1라는 문자열이 포함되지 않은 테스트를 선택하여 실행합니다.
#### pytest markers
- [pytest -m](https://docs.pytest.org/en/latest/how-to/mark.html#mark)
- pytest -m 옵션은 마커 표현식을 사용하여 테스트를 선택합니다.
- MARKER 부분에는 미리 정의된 마커의 이름을 지정합니다.
- ```shell
  pytest -k "$PYTEST_SKIP_OPTION" -m mariadb
  ```
- @pytest.mark.mariadb 데코레이터가 지정된 테스트만 선택하여 실행합니다.
- ```python
  @pytest.mark.mariadb
  class TestMariadbBinlogStreamReader(base.PyMySQLReplicationTestCase):
      def setUp(self):
          super().setUp()
          if not self.isMariaDB():
              self.skipTest("Skipping the entire class for MariaDB")

  @pytest.mark.mariadb
  class TestOptionalMetaData(base.PyMySQLReplicationTestCase):
      def setUp(self):
          super(TestOptionalMetaData, self).setUp()
          self.stream.close()
          self.stream = BinLogStreamReader(
              self.database,
              server_id=1024,
              only_events=(TableMapEvent,),
          )
          if not self.isMySQL8014AndMore():
              self.skipTest("Mysql version is under 8.0.14 - pass TestOptionalMetaData")
          self.execute("SET GLOBAL binlog_row_metadata='FULL';")
  ```
- 위의 코드에서는 TestMariadbBinlogStreamReader와 TestOptionalMetaData 클래스에 대해서만 pytest가 실행됩니다.
- mysql5.7, mysql5.7-ctl, mysql8, mariadb에 대해서 [테스트 코드 구조 개선 및 github action에 대해서 pr](https://github.com/julien-duponchelle/python-mysql-replication/pull/502)을 올렸습니다.

### UserVarEvent 신규 이벤트 구현 
- UserVarEvent 클래스는 사용자 변수 이벤트의 세부 사항을 추출하기 위해 버퍼를 파싱하고 그 타입에 따라 값을 읽는 메서드를 제공합니다
  - [User-var event](https://mariadb.com/kb/en/user_var_event/)
  - [User-var event의 레이아웃](https://dev.mysql.com/doc/dev/mysql-server/latest/classbinary__log_1_1User__var__event.html#:~:text=the%20server%20code.-,%E2%97%86%C2%A0,User_var_event()%20%5B2/2%5D,-binary_log%3A%3AUser_var_event%3A%3AUser_var_event)
- 이벤트와 관련해서 버퍼 레이아웃과 클래스 속성을 공식 문서에서 확인할 수 있습니다.
  - UserVarEvent 버퍼 레이아웃
    - ```shell
      +-------------------------------------------------------------------+
      | name_len | name | is_null | type | charset_number | val_len | val |
      +-------------------------------------------------------------------+
      ```
  - UserVarEvent 클래스의 속성
    - name_len: 사용자 변수 이름의 길이
    - name: 사용자 변수의 이름
    - is_null: 변수 값이 null인지 여부를 나타냅니다.
    - type: 사용자 변수의 타입
    - charset: 사용자 변수의 문자 집합 번호
    - value_len: 사용자 변수의 값의 길이
    - value: 사용자 변수의 값
#### 생성자에서의 버퍼 파싱
- UserVarEvent 클래스의 [__init__ 메서드](https://github.com/julien-duponchelle/python-mysql-replication/blob/5f39cb773590522b37470272fcadc7c9b2efd985/pymysqlreplication/event.py#L707-L744)는 이 버퍼 레이아웃을 파싱하는 역할을 합니다. 레이아웃에서 설명한 순서대로 값을 읽고 클래스의 해당 속성에 할당합니다.
- 타입 처리 및 값 추출
  - 여기서 사용자 변수의 타입별로 버퍼를 파싱하는 값이 달라집니다.
  - |Value|Type|Example|
    |:---:|:---:|:---:|
    |0x00|STRING_RESULT|set @a:="foo"
    |0x01|REAL_RESULT|set @a:= @@timestamp
    |0x02|INT_RESULT|set @a:= 4
    |0x03|ROW_RESULT|(not in use)
    |0x04|DECIMAL_RESULT|set @a:=1.2345|
  - type_to_codes_and_method 딕셔너리는 타입 코드와 해당 데이터 추출 메서드 (_read_string, _read_real, _read_int, _read_decimal, _read_default)를 매핑하는 데 사용됩니다. 타입 코드에 따라 특정 메서드가 호출되어 버퍼에서 값을 파싱합니다.
- 타입별로 [매핑하는 방법](https://docs.python.org/ko/3/library/struct.html#struct-format-strings)에 대해 말씀드리겠습니다.
  - ```python
    struct.unpack("<B", ...)
    ```
  - struct 모듈의 unpack 함수는 주어진 형식에 따라 바이트 데이터를 언팩(해석)합니다.
  - "<B"는 리틀 엔디안 형식의 부호 없는 1바이트 정수(unsigned char)를 나타냅니다. <는 리틀 엔디안을 나타냅니다. (리틀 엔디안은 바이트 순서가 오른쪽에서 왼쪽으로 간다는 것을 의미합니다.)
  - B는 부호 없는 1바이트 정수(unsigned char)를 나타냅니다.
  - [엔디언](https://ko.wikipedia.org/wiki/%EC%97%94%EB%94%94%EC%96%B8)에서 빅 엔디언은 사람이 숫자를 쓰는 방법과 같이 큰 단위의 바이트가 앞에 오는 방법이고, 리틀 엔디언은 반대로 작은 단위의 바이트가 앞에 오는 방법입니다.
- 버퍼 레이아웃을 파싱할 때 int type과 decimal type에 대해서 이슈가 있었습니다.
  - int
    - [MySQL에서 BIGINT UNSIGNED와 BIGINT의 범위](https://dev.mysql.com/doc/refman/8.0/en/integer-types.html)는 다음과 같습니다.
      - BIGINT UNSIGNED: 0 to 18,446,744,073,709,551,615
      - BIGINT: -9,223,372,036,854,775,808 to 9,223,372,036,854,775,807
    - 현재 mysql에서 UNSIGNED일때 18446744073709551615이 -1로 반환되는 이슈가 있었습니다. 관련해서는 [mysql 리포트](https://bugs.mysql.com/bug.php?id=79295)와 [mysql 공식 문서](https://dev.mysql.com/doc/refman/8.0/en/cast-functions.html) 문서를 참고하시면 될 거 같습니다. 
    - ```bash
      mysql> SELECT CAST(1-2 AS UNSIGNED)
      -> 18446744073709551615
      mysql> SELECT CAST(CAST(1-2 AS UNSIGNED) AS SIGNED);
      -> -1
      mysql> select cast(-1 as unsigned);
      +----------------------+
      | cast(-1 as unsigned) |
      +----------------------+
      | 18446744073709551615 |
      +----------------------+
      1 row in set (0.00 sec)
      ```
    - -1로 나오는 이슈를 어떻게 처리할지 고민하며 바이트 덤프를 기반으로 데이터를 파싱해 봤습니다.
    - ```shell
      00 26 11 EE 64 0E 01 00 00 00 2F 00 00 00 15 0A .&..d...../.....
      00 00 00 00|01 00 00 00|61|00|02|FF 00 00 00|08 ........a.......
      00 00 00|FF FF FF FF FF FF FF FF|01|16 25 04 29 .............%.)
      ```
      - name_len: 01 00 00 00 → 리틀 엔디안 형식으로 1입니다.
      - name: 61 → ASCII로 'a'입니다.
      - is_null: 00 → False입니다.
      - type: 02 → INT_RESULT입니다.
      - charset: FF 00 00 00 → 생략됩니다.
      - val_len: 08 00 00 00 → 값의 길이는 8바이트입니다.
      - val: FF FF FF FF FF FF FF FF → 18446744073709551615입니다.
      - flag: 01 → UNSIGNED_F입니다.
    - 바이트 덤프를 기반으로 데이터를 파싱을 했을 때, flag가 01일 때 UNSIGNED로 이를 기반으로 int 타입을 파싱하였습니다. 따라서 flags를 활용하기 위해 임시버퍼를 사용하였고 [flag가 1일 때와 아닐 때를 리틀 엔디안을 활용해 unsinged와 singed를 구분해서 언패킹](https://github.com/julien-duponchelle/python-mysql-replication/blob/5f39cb773590522b37470272fcadc7c9b2efd985/pymysqlreplication/event.py#L771-L776)하였습니다.
  - deciaml
    - [내부 패킷 객체를 사용](https://github.com/julien-duponchelle/python-mysql-replication/blob/5f39cb773590522b37470272fcadc7c9b2efd985/pymysqlreplication/row_event.py#L411C4-L458)하여 데이터를 읽으려고 했지만 위에 int 타입의 경우 flags를 활용해야 하므로, 임시 버퍼를 사용해서 바이트를 받아서 decimal로 파싱을 하는 메소드를 개발해야 했습니다. deciaml은 음수일때와 양수일때를 구분해서 파싱을 했고 음수일때 기존 코드에서 -4444.2343243245 의 경우 XOR 연산사 byte 범위 벖어나서 [0-255 범위로 제한](https://github.com/julien-duponchelle/python-mysql-replication/blob/5f39cb773590522b37470272fcadc7c9b2efd985/pymysqlreplication/util/bytes.py#L65C34-L66C25) 하였습니다. 
- charset에 대해서도 mysql5.7, mysql8, mariadb에서 매번 바뀌는 이슈가 있었습니다.
  - 이는 환경설정에서 charset과 collation에 영향을 받는 것을 확인했습니다.
  - MySQL은 다양한 문자셋을 지원하며, 그중 여러 유니코드 문자셋을 포함하고 있습니다. 콜레이션은 특정 문자셋의 문자들을 어떻게 비교하고 정렬할지를 결정하는 규칙의 집합입니다.
  - 사용 가능한 문자셋을 확인하려면 INFORMATION_SCHEMA CHARACTER_SETS 테이블 또는 SHOW CHARACTER SET 문장을 사용할 수 있으며 사용 가능한 콜레이션을 확인하려면 INFORMATION_SCHEMA COLLATIONS 테이블 또는 SHOW COLLATION 문장을 사용할 수 있습니다. 관련해서는 [공식 문서](https://dev.mysql.com/doc/refman/8.0/en/charset-mysql.html
)를 확인해 주시면 될 거 같습니다.
  - Database를 생성할 때 문자셋과 콜레이션을 지정하면 같은 값을 반환하는 것을 확인했습니다. 
  - ```shell
    mysql> CREATE DATABASE test CHARACTER SET utf8mb3 COLLATE utf8mb3_general_ci;
    Query OK, 1 row affected (0.02 sec)
    
    mysql> SHOW VARIABLES LIKE 'character_set%';
    +--------------------------+-------------------------------------+
    | Variable_name            | Value                               |
    +--------------------------+-------------------------------------+
    | character_set_client     | latin1                              |
    | character_set_connection | latin1                              |
    | character_set_database   | utf8mb4                             |
    | character_set_filesystem | binary                              |
    | character_set_results    | latin1                              |
    | character_set_server     | latin1                              |
    | character_set_system     | utf8                                |
    | character_sets_dir       | /usr/share/percona-server/charsets/ |
    +--------------------------+-------------------------------------+
    8 rows in set (0.03 sec)
    
    | mysql-bin.000003 | 5082 | User var       |         1 |        5135 | @`test_user_var`=_latin1 0x666F6F COLLATE latin1_swedish_ci                                         |
    
    mysql> SET NAMES 'utf8mb3';
    Query OK, 0 rows affected (0.01 sec)
    
    mysql> SHOW VARIABLES LIKE 'character_set%';
    +--------------------------+-------------------------------------+
    | Variable_name            | Value                               |
    +--------------------------+-------------------------------------+
    | character_set_client     | utf8                                |
    | character_set_connection | utf8                                |
    | character_set_database   | utf8mb4                             |
    | character_set_filesystem | binary                              |
    | character_set_results    | utf8                                |
    | character_set_server     | latin1                              |
    | character_set_system     | utf8                                |
    | character_sets_dir       | /usr/share/percona-server/charsets/ |
    +--------------------------+-------------------------------------+
    8 rows in set (0.03 sec)
    ```
- squash merge를 통해서 압축해서 커밋을 한 뒤에 [PR](https://github.com/julien-duponchelle/python-mysql-replication/pull/466)을 올렸습니다.
- python-mysql-replication 프로젝트에 기여하면서 다양한 개발 경험을 얻었습니다. 이 과정에서는 기술적 지식을 획득하는 것뿐만 아니라, 오픈소스 커뮤니티와의 협업에 대한 인사이트 또한 얻을 수 있었습니다. 오픈소스에 기여하는 것은 코드 작성뿐만 아니라 지식 공유, 협업, 함께 성장하는 커뮤니티의 일원이 되어가는 것 같습니다. OSSCA 프로그램을 통해 멘토 및 동료 멘티들과의 교류 과정에서 많은 것을 배우고, 개인적으로 큰 성장을 이룰 수 있었던 것이 무척 값진 경험으로 남았습니다.
