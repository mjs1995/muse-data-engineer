# Error: Table 'CTLGS' already exists (state=42S01,code=1050)
- Hive 메타스토어 데이터베이스 초기화 중에 발생한 'Table 'CTLGS' already exists' 오류는, Hive 메타스토어 스키마 초기화 과정 중 이미 존재하는 테이블에 대한 처리를 시도할 때 발생할 수 있습니다.
- ```shell
  Error: Table 'CTLGS' already exists (state=42S01,code=1050)
  com.mysql.jdbc.exceptions.jdbc4.MySQLSyntaxErrorException: Table 'CTLGS' already exists
  ```
- 에러를 확인해 보니 메타스토어 데이터베이스가 이미 일부 스키마 구조를 가지고 있었습니다.
  - ```shell
    mysql> select * from CTLGS;
    +---------+------+--------------------------+--------------+
    | CTLG_ID | NAME | DESC                     | LOCATION_URI |
    +---------+------+--------------------------+--------------+
    |       1 | hive | Default catalog for Hive | TBD          |
    +---------+------+--------------------------+--------------+
    1 row in set (0.00 sec)
- 데이터베이스를 삭제하고 다시 생성함으로써 이전 스키마와 관련된 모든 문제를 제거하고 새로운 스키마를 생성합니다.
  - ```shell
    mysql> DROP DATABASE test;
    Query OK, 38 rows affected (0.64 sec)
    
    mysql> CREATE DATABASE test;
    Query OK, 1 row affected (0.01 sec)
    ```
- hive metastore를 사용하기 위해 만든 CTLGS를 지우고 그다음에 metastore.xml 파일을 수정해 줍니다.
  - ```shell
    <property>
      <name>javax.jdo.option.ConnectionURL</name>
      <value>jdbc:mysql://mysql.mysql.svc.cluster.local:3306/test?createdatabaseifnotexist=true&useSSL=false</value>
    </property>
    ```
  - javax.jdo.option.ConnectionURL 속성의 변경은 MySQL JDBC 연결 URL을 지정합니다. 여기서 createdatabaseifnotexist=true 옵션은 지정된 데이터베이스가 존재하지 않을 경우 자동으로 생성하도록 합니다. 이 옵션은 데이터베이스 초기화 과정에서 유용할 수 있습니다.
- 그다음 metastore.xml 파일에서 설정을 불러와 ConfigMap을 만들어줍니다.
  - ```shell
    kubectl create configmap metastore-cfg --from-file=metastore.xml -n hive
    ```
- 마지막으로 Hive 메타스토어의 스키마를 MySQL 데이터베이스에 초기화하기 위해 schematool -initSchema -dbType mysql 명령실행하면 됩니다.
  - ```shell
    kubectl get pods -n hive
    NAME                    READY   STATUS      RESTARTS   AGE
    hive-initschema-c2wtg   0/1     Completed   0          3h43m
    ```
