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
- <img width="16" alt="git-merged" src="https://github.com/mjs1995/muse-data-engineer/assets/47103479/1199bcfb-b94f-4ff3-89cc-9d271da95dac"> [Enhance Testing with MySQL8 & Update GitHub Actions](https://github.com/julien-duponchelle/python-mysql-replication/pull/484)
- <img width="16" alt="git-merged" src="https://github.com/mjs1995/muse-data-engineer/assets/47103479/1199bcfb-b94f-4ff3-89cc-9d271da95dac"> [Developed UserVarEvent and Added Statement-Based Logging Test](https://github.com/julien-duponchelle/python-mysql-replication/pull/466)
- 문서기여, 버그수정, 기능개발 등 기여를 하였고 개인적인 기여 난이도를 기준으로 말씀드리려고 합니다.

## 문서기여 
- [데이터 파이프라인 핵심 가이드](https://product.kyobobook.co.kr/detail/S000001766501) 책을 읽었을 때 CDC 부분과 binlog에 대해서 관심이 있었던 부분이 있어서 읽을 당시에는 몰랐지만 이번에 OSSCA를 진행하면서 python-mysql-replication이 사용되었다는 것을 알게되었습니다. 관련하여 멘토님과 얘기를 하다가 프로젝트 주인장분이 프로젝트의 쓰임새에 대해서 관심이 많아서 알려드리면서 README를 업데이트를 하는건 어떠냐고 물어보았습니다.
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
- RowsEvent는 BinLogEvent를 상속받아 행 기반 이벤트를 처리하고 UpdateRowsEvent는 RowsEvent를 상속받아 업데이트 이벤트를 처리하는데 [_dump 메소드](https://github.com/23-OSSCA-python-mysql-replication/python-mysql-replication/blob/08219f740edda3923ec866ebeb92fa580e3cc571/pymysqlreplication/row_event.py#L468C9-L468C14)에서 "Affected columns" 상속되는게 중복으로 출력이 되고 있어서 PR을 올렸습니다.
  - Remove duplicated Affected columns output in UpdateRowsEvent](https://github.com/julien-duponchelle/python-mysql-replication/pull/478)
 
## 기능개발
### MySQL8 버전 테스트 추가 및 github action 버전 업그레이드 
- 프로젝트 진행 당시 MySQL8에 대한 테스트를 할 수 없었고 MySQL5.7버전과 MariaDB에 대해서만 진행을 할 수 있었으며 [MySQL 8.0.14](https://dev.mysql.com/doc/relnotes/mysql/8.0/en/news-8-0-14.html#:~:text=Replication%3A%20Character)버전 이후로 binlog_row_metadata=FULL이 나오면서 [OptionalMetaData에 대해서 이벤트 개발](https://github.com/julien-duponchelle/python-mysql-replication/pull/471)을 하였습니다. 관련해서는 [같은 조의 멘티님](https://seanical.tistory.com/5)의 블로그를 읽어보시면 도움되실거 같습니다. 따라서 MySQL8 버전대에 대해서 도커파일을 추가하고 테스트 코드를 수정해야 했습니다.
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
  - [도커 컴포스 파일](https://docs.docker.com/compose/reference/#command-options-overview-and-help)은 여러 개의 컨테이너 구성 정보를 코드로 정의하고, 명령을 실행함으로써 애플리케이션의 실행환경을 구성하는 컨테이너들을 일원 관리하기 위한 툴로 docker-compose.yml 코드를 보면 [percona:8.0 이미지](https://hub.docker.com/_/percona)를 추가하였고 [percona 8](https://github.com/percona/percona-docker/blob/3f666ccdf6a9eed0e0505723fbe8b4954a105c99/percona-server-8.0/Dockerfile)은 gituhb에서 확인해보면 최신 버전을 가져오고 있는거 같습니다. (Percona Server 8.0.33 버전과 MySQL Shell 8.0.33 버전을 포함하고 있음)
  - 해당 코드는 [YAML Anchors와 Aliases](https://yaml.org/spec/1.2.2/#3222-anchors-and-aliases)를 사용하였는데 이는 중복된 데이터 구조나 값을 재사용하기 위한 메커니즘이며, 이를 통해 파일의 크기를 줄이고 데이터의 중복을 최소화하며 파일의 가독성을 향상시켰습니다. [도커 앵커](https://docs.docker.com/compose/compose-file/10-fragments/)에서 사례를 확인할 수 있습니다.
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
  - 현재 pytest github action에서 사용되고 있는 workflow 액션은 actions/checkout@v2과 actions/setup-python@v2 입니다. 여기서 Node 16의 기본 런타임은 2023년 9월 11일에 지원이 종료될 예정으로 액션을 실행할 때 마다 경고 문구가 나왔습니다. 따라서 GitHub Actions 버전의 기능과 최적화를 활용하기 위해 최신버전으로 업데이트를 진행하였습니다.
    - <img width="1023" alt="image" src="https://github.com/mjs1995/muse-data-engineer/assets/47103479/67ce401f-8f41-4bc8-b802-e3c234fad7c2">
    - [actions/checkout@v4](https://github.com/actions/checkout/releases) : GitHub 저장소의 코드를 체크아웃하여 GitHub Actions 실행 환경의 작업 디렉토리에 배치합니다.
    - [actions/setup-python@v4](https://github.com/actions/setup-python/releases) : 작업 환경에 특정 버전의 Python을 설치하고 설정합니다.

### UserVarEvent 신규 이벤트 구현 
- 
