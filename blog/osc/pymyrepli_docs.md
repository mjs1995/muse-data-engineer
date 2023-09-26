# [OSSCA](https://www.contribution.ac/2023-ossca) 오픈소스 컨트리뷰션 아카데미
- ![image](https://github.com/mjs1995/muse-data-engineer/assets/47103479/064d2377-fe5d-4e31-8893-24dbfbb8ded7)
- 2023 오픈소스 컨트리뷰션 아카데미를 참가하면서 python-mysql-repllication프로젝트의 멘티로 참가하게 되었습니다. 약 3달간의 여정으로 온/오프라인 모임을 통해서 오픈소스 기여를 하였습니다.
- 참여 계기는 데이터 엔지니어링에 관심이 있는 다양한 분들과 협업을 하면서 개발 문화와 오픈소스에 기여하는 방식을 보고자 지원을 하였습니다. 또한 프로젝트 운영을 하면서 binlog 파일에 대해 이슈가 있었고 CDC에 관심이 있어서 세미나를 다니던 중 python-mysql-repllication의 분야가 핏이 맞아 신청을 하였습니다.

# [python-mysql-replication](https://github.com/julien-duponchelle/python-mysql-replication) 프로젝트 소개
## MySQL 데이터베이스에서 데이터 추출하는 두 가지 방법
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
## python-mysql-replication
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

## 복제 데이터 포맷 
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

## 예제
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
    
# Reference 
- [MySQL 8.0 docs](https://dev.mysql.com/doc/refman/8.0/en/)
- [데이터 파이프라인 핵심 가이드](https://product.kyobobook.co.kr/detail/S000001766501)
- [MySQL 8.0 2권](https://product.kyobobook.co.kr/detail/S000001766483)
