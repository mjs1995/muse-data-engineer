# Trino를 사용하여 MinIO에 저장된 데이터 쿼리하기(1) - Hive Metastore MySQL로 구성
- Hive Metastore를 MySQL에 설정하고, Trino를 사용하여 MinIO에 저장된 데이터에 대한 쿼리를 실행하는 방법에 대해 포스팅하려고 합니다.

## Hive Metastore와 MySQL 설정
- Hive Metastore는 메타데이터 저장소로 사용되며, 여기서는 MySQL을 이용해 이를 설정합니다.
- Helm 차트 리포지토리 추가 및 MySQL 설치
  - ```shell
    helm repo add bitnami https://charts.bitnami.com/bitnami
    kubectl create namespace mysql
    helm install mysql bitnami/mysql -n mysql
    ```
- MySQL 루트 비밀번호 얻기
  - ```shell
    MYSQL_ROOT_PASSWORD=$(kubectl get secret --namespace mysql mysql -o jsonpath="{.data.mysql-root-password}" | base64 -d)
    ```
- MySQL 클라이언트 Pod 실행 및 데이터베이스 초기화
  - ```shell
    kubectl run mysql-client --rm --tty -i --restart='Never' --image docker.io/bitnami/mysql:8.0.35-debian-11-r0 --namespace mysql --env MYSQL_ROOT_PASSWORD=$MYSQL_ROOT_PASSWORD --command -- bash
    kubectl exec -it mysql-client -n mysql -- /bin/bash
    mysql -h mysql.mysql.svc.cluster.local -uroot -p"$MYSQL_ROOT_PASSWORD"

    mysql> SELECT VERSION();
    +-----------+
    | VERSION() |
    +-----------+
    | 8.0.35    |
    +-----------+
    1 row in set (0.00 sec)
    ```
- MySQL에서 다음 명령어를 사용하여 데이터베이스와 사용자를 생성하고 권한을 부여합니다.
  - ```shell
    CREATE DATABASE test;
    CREATE USER 'mjs'@'%' IDENTIFIED BY '1234';
    GRANT ALL PRIVILEGES ON test.* TO 'mjs'@'%';
    FLUSH PRIVILEGES;
    ```

## Hive Metastore 설정
- Hive Metastore를 위한 Kubernetes 설정(hive_meta_mysql.yaml)을 만듭니다. 이는 MySQL 데이터베이스를 사용하여 메타데이터를 저장하는 방식입니다.
- Metastore의 MySQL 데이터베이스 설정을 위한 Kubernetes 리소스(hive_meta_mysql.yaml)를 정의합니다.
  - Secret 리소스를 통해 MySQL의 루트 비밀번호를 안전하게 저장합니다.
  - PersistentVolumeClaim은 데이터 저장을 위한 볼륨을 선언합니다.
  - Deployment는 MySQL 서버를 실행하는 데 사용되는 파드를 정의합니다.
  - Service는 MySQL 파드에 네트워크 액세스를 제공합니다.
- 위에서 만든 사용자 권한의 비밀번호인 1234를 base인코딩 해서 ROOT_PASSWORD에 넣어줍니다.
  - ```yaml
    apiVersion: v1
    kind: Secret
    metadata:
      name: mysql-secrets
      namespace: hive
    type: Opaque
    data:
      ROOT_PASSWORD: MTIzNA==
    ---
    apiVersion: v1
    kind: PersistentVolumeClaim
    metadata:
      name: mysql-data-disk
      namespace: hive
    spec:
      accessModes:
        - ReadWriteOnce
      resources:
        requests:
          storage: 2Gi
    ---
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: mysql-deployment
      namespace: hive
      labels:
        app: mysql
    spec:
      replicas: 1
      selector:
        matchLabels:
          app: mysql
      template:
        metadata:
          labels:
            app: mysql
        spec:
          containers:
            - name: mysql
              image: mysql:8.0.14
              ports:
                - containerPort: 3306
              volumeMounts:
                - mountPath: "/var/lib/mysql"
                  name: mysql-data
              env:
                - name: MYSQL_ROOT_PASSWORD
                  valueFrom:
                    secretKeyRef:
                      name: mysql-secrets
                      key: ROOT_PASSWORD
          volumes:
            - name: mysql-data
              persistentVolumeClaim:
                claimName: mysql-data-disk
    ---
    apiVersion: v1
    kind: Service
    metadata:
      name: mysql-service
      namespace: hive
    spec:
      selector:
        app: mysql
      ports:
        - protocol: TCP
          port: 3306
          targetPort: 3306
    ```
- 위에서 만든 설정을 적용해 줍니다.
  - ```shell
    kubectl apply -f hive_meta_mysql.yaml
    ```

# Trino를 사용하여 MinIO에 저장된 데이터 쿼리하기(2) - Hive Metastore 배포
## Dockerfile 생성
- Hive Metastore를 실행하기 위한 Docker 이미지를 만들기 위한 Dockerfile을 작성합니다.
  - [하둡](https://downloads.apache.org/hive/)과 [hive](https://hadoop.apache.org/releases.html), [mysql 커넥터](https://repo1.maven.org/maven2/com/mysql/mysql-connector-j/8.0.33/)의 경우 링크에서 확인할 수 있습니다.
  - ```Docker
    FROM openjdk:11-slim

    ARG HADOOP_VERSION=3.2.1

    RUN apt-get update && apt-get install -y curl --no-install-recommends && \
            rm -rf /var/lib/apt/lists/*

    RUN curl https://archive.apache.org/dist/hadoop/common/hadoop-$HADOOP_VERSION/hadoop-$HADOOP_VERSION.tar.gz \
            | tar xvz -C /opt/  \
            && ln -s /opt/hadoop-$HADOOP_VERSION /opt/hadoop \
            && rm -r /opt/hadoop/share/doc

    RUN ln -s /opt/hadoop/share/hadoop/tools/lib/hadoop-aws* /opt/hadoop/share/hadoop/common/lib/ && \
        ln -s /opt/hadoop/share/hadoop/tools/lib/aws-java-sdk* /opt/hadoop/share/hadoop/common/lib/

    ENV HADOOP_HOME="/opt/hadoop"
    ENV PATH="/opt/spark/bin:/opt/hadoop/bin:${PATH}"

    RUN curl https://repo1.maven.org/maven2/org/apache/hive/hive-standalone-metastore/3.1.2/hive-standalone-metastore-3.1.2-bin.tar.gz \
            | tar xvz -C /opt/ \
            && ln -s /opt/apache-hive-metastore-3.1.2-bin /opt/hive-metastore

    RUN curl -L https://repo1.maven.org/maven2/com/mysql/mysql-connector-j/8.0.33/mysql-connector-j-8.0.33.jar \
        -o /opt/mysql-connector-j-8.0.33.jar \
        && ln -s /opt/mysql-connector-j-8.0.33.jar /opt/hadoop/share/hadoop/common/lib/ \
        && ln -s /opt/mysql-connector-j-8.0.33.jar /opt/hive-metastore/lib/

    RUN rm /opt/hive-metastore/lib/guava-19.0.jar && \
            ls -lah /opt/hadoop/share/hadoop/common/lib/ && \
            cp /opt/hadoop/share/hadoop/common/lib/guava-27.0-jre.jar /opt/hive-metastore/lib/ && \
            cp /opt/hadoop/share/hadoop/tools/lib/hadoop-aws-3.2.1.jar /opt/hive-metastore/lib/ && \
            cp /opt/hadoop/share/hadoop/tools/lib/aws-java-sdk-bundle-1.11.375.jar /opt/hive-metastore/lib/

    CMD ["/opt/hive-metastore/bin/start-metastore"]
    ```

## Docker 배포
- Dockerfile을 사용하여 Hive Metastore 이미지를 빌드하고, 이를 Docker Hub 또는 Google Container Registry에 푸시합니다.
- 로컬 Docker 환경 또는 Kubernetes 클러스터에서 이미지를 실행합니다.
  - Docker 이미지 빌드
    - ```shell
      docker build -t hive-metastore .
      ```
  - Docker 이미지 푸시
    - ```shell
      docker push hive-metastore
      ```
  - Docker 컨테이너 실행
    - ```shell
      docker run -d -p 9083:9083 hive-metastore
      ```
- 로컬에서 빌드한 Docker 이미지를 GCR에 업로드할 수 있습니다.
  - Docker 이미지 빌드
    - ```shell
      docker build -t gcr.io/ggke-401900/hive-metastore .
      ```
  - Docker 이미지 푸시
    - ```shell
      docker push gcr.io/ggke-401900/hive-metastore
      ```
- metastore.xml은 Hive 메타스토어 서비스에 대한 중요한 설정을 포함하는 XML 파일입니다. 이 파일은 Hive 메타스토어가 어떻게 데이터베이스와 연결되고, 어떤 드라이버를 사용하는지 추가적인 설정 옵션들을 정의합니다.
  - ```shell
    kubectl create configmap metastore-cfg --from-file=metastore.xml -n hive
    ```
  - metastore.xml은 다음과 같습니다.
  - ```xml
    <configuration>
    <property>
      <name>metastore.thrift.uris</name>
      <value>thrift://localhost:9083</value>
      <description>Thrift URI for the remote metastore. Used by metastore client to connect to remote metastore.</description>
    </property>
    <property>
      <name>metastore.task.threads.always</name>
      <value>org.apache.hadoop.hive.metastore.events.EventCleanerTask</value>
    </property>
    <property>
      <name>metastore.expression.proxy</name>
      <value>org.apache.hadoop.hive.metastore.DefaultPartitionExpressionProxy</value>
    </property>
    <property>
      <name>javax.jdo.option.ConnectionURL</name>
      <value>jdbc:mysql://mysql.mysql.svc.cluster.local:3306/test?createdatabaseifnotexist=true&amp;useSSL=false</value>
    </property>
    <property>
      <name>javax.jdo.option.ConnectionDriverName</name>
      <value>com.mysql.cj.jdbc.Driver</value>
    </property>
    <property>
      <name>javax.jdo.option.ConnectionUserName</name>
      <value>mjs</value>
    </property>
    <property>
      <name>javax.jdo.option.ConnectionPassword</name>
      <value>1234</value>
    </property>
    <property>
      <name>fs.s3a.server-side-encryption-algorithm</name>
      <value>AES256</value>
    </property>
    <property>
      <name>fs.s3a.endpoint</name>
      <value>http://minio-service.minio-dev.svc.cluster.local:9000</value>
    </property>
    <property>
      <name>fs.s3a.access.key</name>
      <value>minioadmin</value>
    </property>
    <property>
      <name>fs.s3a.secret.key</name>
      <value>minioadmin</value>
    </property>
    <property>
      <name>fs.s3a.path.style.access</name>
      <value>true</value>
    </property>
    <property>
      <name>fs.s3a.impl</name>
      <value>org.apache.hadoop.fs.s3a.S3AFileSystem</value>
    </property>
    </configuration>
    ```
- Hive Metastore 초기화 (hive-initschema.yaml)
  - Hive Metastore 스키마 초기화를 위한 Kubernetes Job을 정의합니다.
  - MySQL 데이터베이스에 연결하여 필요한 스키마를 생성합니다.
  - ```yaml
    apiVersion: batch/v1
    kind: Job
    metadata:
      name: hive-initschema
      namespace: hive
    spec:
      template:
        spec:
          containers:
          - name: hivemeta
            image: gcr.io/ggke-401900/hive-metastore
            command: ["/opt/hive-metastore/bin/schematool"]
            args: ["--verbose", "-initSchema", "-dbType", "mysql", "-userName", "mjs",
              "-passWord", "1234", "-url", "jdbc:mysql://mysql.mysql.svc.cluster.local:3306/test?useSSL=false"]
          restartPolicy: Never
      backoffLimit: 4
    ```
  - yaml 파일을 적용합니다.
  - ```shell
    kubectl apply -f hive-initschema.yaml
    ```
- Metastore 배포 (meta.yaml)
  - Hive Metastore 서비스를 위한 Kubernetes Deployment와 Service를 정의합니다.
  - 이 설정은 Trino가 Hive Metastore와 통신할 수 있도록 합니다.
  - ```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: metastore
      namespace: hive
    spec:
      selector:
        matchLabels:
          app: metastore
      strategy:
        type: Recreate
      template:
        metadata:
          labels:
            app: metastore
        spec:
          containers:
          - name: metastore
            image: gcr.io/ggke-401900/hive-metastore
            ports:
            - containerPort: 9083
            volumeMounts:
            - name: metastore-cfg-vol
              mountPath: /opt/hive-metastore/conf/metastore-site.xml
              subPath: metastore.xml
            command: ["/opt/hive-metastore/bin/start-metastore"]
            args: ["-p", "9083"]
            resources:
              requests:
                memory: "2Gi"
                cpu: "1"
            imagePullPolicy: Always
          volumes:
          - name: metastore-cfg-vol
            configMap:
              name: metastore-cfg
    ---
    apiVersion: v1
    kind: Service
    metadata:
      name: metastore
      namespace: hive
    spec:
      ports:
      - port: 9083
        targetPort: 9083
      selector:
        app: metastore
    ```
  - yaml 파일을 적용합니다.
  - ```shell
    kubectl apply -f meta.yaml
    ```
- mysql에 접속하여 metastore와 관련된 테이블들을 확인해 봅니다.
  - ```shell
    mysql> SHOW databases;
    +--------------------+
    | Database           |
    +--------------------+
    | information_schema |
    | my_database        |
    | mysql              |
    | performance_schema |
    | sys                |
    | test               |
    +--------------------+
    6 rows in set (0.00 sec)

    mysql> USE test;
    Database changed

    mysql> SHOW TABLES;
    +-------------------------------+
    | Tables_in_test                |
    +-------------------------------+
    | AUX_TABLE                     |
    | BUCKETING_COLS                |
    | CDS                           |
    | COLUMNS_V2                    |
    | COMPACTION_QUEUE              |
    | COMPLETED_COMPACTIONS         |
    | COMPLETED_TXN_COMPONENTS      |
    | CTLGS                         |
    | DATABASE_PARAMS               |
    | DBS                           |
    | DB_PRIVS                      |
    | DELEGATION_TOKENS             |
    | FUNCS                         |
    | FUNC_RU                       |
    | GLOBAL_PRIVS                  |
    | HIVE_LOCKS                    |
    | IDXS                          |
    | INDEX_PARAMS                  |
    | I_SCHEMA                      |
    | KEY_CONSTRAINTS               |
    | MASTER_KEYS                   |
    | MATERIALIZATION_REBUILD_LOCKS |
    | METASTORE_DB_PROPERTIES       |
    | MIN_HISTORY_LEVEL             |
    | MV_CREATION_METADATA          |
    | MV_TABLES_USED                |
    | NEXT_COMPACTION_QUEUE_ID      |
    | NEXT_LOCK_ID                  |
    | NEXT_TXN_ID                   |
    | NEXT_WRITE_ID                 |
    | NOTIFICATION_LOG              |
    | NOTIFICATION_SEQUENCE         |
    | NUCLEUS_TABLES                |
    | PARTITIONS                    |
    | PARTITION_EVENTS              |
    | PARTITION_KEYS                |
    | PARTITION_KEY_VALS            |
    | PARTITION_PARAMS              |
    | PART_COL_PRIVS                |
    | PART_COL_STATS                |
    | PART_PRIVS                    |
    | REPL_TXN_MAP                  |
    | ROLES                         |
    | ROLE_MAP                      |
    | RUNTIME_STATS                 |
    | SCHEMA_VERSION                |
    | SDS                           |
    | SD_PARAMS                     |
    | SEQUENCE_TABLE                |
    | SERDES                        |
    | SERDE_PARAMS                  |
    | SKEWED_COL_NAMES              |
    | SKEWED_COL_VALUE_LOC_MAP      |
    | SKEWED_STRING_LIST            |
    | SKEWED_STRING_LIST_VALUES     |
    | SKEWED_VALUES                 |
    | SORT_COLS                     |
    | TABLE_PARAMS                  |
    | TAB_COL_STATS                 |
    | TBLS                          |
    | TBL_COL_PRIVS                 |
    | TBL_PRIVS                     |
    | TXNS                          |
    | TXN_COMPONENTS                |
    | TXN_TO_WRITE_ID               |
    | TYPES                         |
    | TYPE_FIELDS                   |
    | VERSION                       |
    | WM_MAPPING                    |
    | WM_POOL                       |
    | WM_POOL_TO_TRIGGER            |
    | WM_RESOURCEPLAN               |
    | WM_TRIGGER                    |
    | WRITE_SET                     |
    +-------------------------------+
    74 rows in set (0.00 sec)

- Hive 관련 리소스가 올바르게 실행되고 있는지 확인합니다.
  - ```shell
    mun_js@cloudshell:~ (ggke-401900)$ kubectl get all -n hive
    NAME                                     READY   STATUS      RESTARTS       AGE
    pod/hive-initschema-xd7ct                0/1     Completed   0              3d19h
    pod/metastore-78d4687f85-kqgm6           1/1     Running     0              3d20h
    pod/trino-coordinator-5bcbbcddf4-j2bml   1/1     Running     1 (3d1h ago)   3d2h
    pod/trino-worker-656f6c48cd-f5tfc        1/1     Running     0              89m
    pod/trino-worker-656f6c48cd-mkkd4        1/1     Running     0              12m
  
    NAME                TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE
    service/metastore   ClusterIP   10.52.7.186    <none>        9083/TCP   3d20h
    service/trino       ClusterIP   10.52.13.135   <none>        8080/TCP   3d19h
  
    NAME                                READY   UP-TO-DATE   AVAILABLE   AGE
    deployment.apps/metastore           1/1     1            1           3d20h
    deployment.apps/trino-coordinator   1/1     1            1           3d19h
    deployment.apps/trino-worker        2/2     2            2           3d19h
  
    NAME                                           DESIRED   CURRENT   READY   AGE
    replicaset.apps/metastore-78d4687f85           1         1         1       3d20h
    replicaset.apps/trino-coordinator-5bcbbcddf4   1         1         1       3d2h
    replicaset.apps/trino-coordinator-5f47dffc99   0         0         0       3d19h
    replicaset.apps/trino-worker-656f6c48cd        2         2         2       3d2h
    replicaset.apps/trino-worker-6f5fc48987        0         0         0       3d19h
  
    NAME                        COMPLETIONS   DURATION   AGE
    job.batch/hive-initschema   1/1           47s        3d19h
    ```

# Trino를 사용하여 MinIO에 저장된 데이터 쿼리하기(3) - Trino 설치 및 연동
- Kubernetes 클러스터에서 Trino를 설치하고, MinIO를 데이터 저장소로 활용하는 방법에 대해 포스팅하려고 합니다. 이를 통해 대규모 데이터셋에 대한 빠르고 유연한 쿼리를 수행할 수 있습니다.

## Trino와 MinIO 설정
### MinIO 설정
- MinIO 클라이언트 초기 설정
  - MinIO 클라이언트(mc)를 사용해 MinIO 서비스에 접속합니다.
  - ```shell
    ./mc alias set k8s-minio-dev http://127.0.0.1:9000 minioadmin minioadmin
    mun_js@cloudshell:~ (ggke-401900)$ ./mc ls k8s-minio-dev/trino
    [2023-11-12 13:51:31 UTC] 307MiB STANDARD nyc_data.csv
    ```
  - ![image](https://github.com/mjs1995/muse-data-engineer/assets/47103479/907b1454-5501-4345-8db9-07ac7dfe0acd)
  - trino 버킷에 있는 nyc_data.csv 파일을 trino에서 읽어드리려고 합니다.
  - ![image](https://github.com/mjs1995/muse-data-engineer/assets/47103479/af848d4b-ce52-4a81-a10c-1bf28d38364d)
- MinIO 네트워크 정책 설정
  - MinIO 서비스가 Trino 서비스에 접근할 수 있도록 네트워크 정책을 설정합니다. 즉 minio-dev 네임스페이스의 MinIO 서비스가 hive 네임스페이스의 Trino 서비스에 접근할 수 있도록 하는 네트워크 정책입니다.(minio-to-trino-networkpolicy.yaml)
  - ```shell
    apiVersion: networking.k8s.io/v1
    kind: NetworkPolicy
    metadata:
      name: allow-minio-to-trino
      namespace: minio-dev
    spec:
      podSelector:
        matchLabels:
          app: minio
      policyTypes:
      - Ingress
      - Egress
      ingress:
      - from:
        - namespaceSelector:
            matchLabels:
              name: hive
      egress:
      - to:
        - namespaceSelector:
            matchLabels:
              name: hive
    ```
- 네트워크 정책 적용
  - 위에서 작성한 네트워크 정책 파일을 적용합니다.
  - ```shell
    kubectl apply -f minio-to-trino-networkpolicy.yaml
    ```
- 네트워크 정책 확인
  - 새로운 네트워크 정책이 정상적으로 적용되었는지 확인합니다.
  - ```shell
    kubectl get networkpolicies --namespace minio-dev
    NAME                   POD-SELECTOR   AGE
    allow-minio-to-trino   app=minio      23s
    ```
  - minio-dev 네임스페이스의 MinIO 서비스와 hive 네임스페이스의 Trino 서비스 간의 네트워크 통신이 가능해집니다.

### Trino 설치 및 설정
- Trino Helm 차트 리포지토리를 추가합니다.
  - ```shell
    helm repo add trino https://trinodb.github.io/charts
    ```
- Trino 차트를 다운로드합니다.
  - ```shell
    mkdir trino-minio
    helm repo add trino https://trinodb.github.io/charts
    helm pull trino/trino
    tar xvf trino-0.13.0.tgz
    cd trino/
    ```
  - ![image](https://github.com/mjs1995/muse-data-engineer/assets/47103479/bc82b427-cc8b-4acc-823b-79fab225d818)
- values.yaml 파일을 수정하여 Trino의 설정을 조정합니다. MinIO와의 연동을 위해 additionalCatalogs 섹션에서 MinIO 카탈로그를 정의합니다.
  - ```yaml
    additionalCatalogs:
      minio: |-
        connector.name=hive
        hive.metastore.uri=thrift://metastore:9083
        hive.s3.endpoint=http://minio-service.minio-dev.svc.cluster.local:9000
        hive.s3.aws-access-key=minioadmin
        hive.s3.aws-secret-key=minioadmin
        hive.s3.path-style-access=true
    ```
- Trino 리소스 할당량을 조정합니다.
  - ```yaml
    coordinator:
      resources:
        requests:
          memory: "2Gi"
          cpu: "500m"
        limits:
          memory: "4Gi"
          cpu: "1000m"
    worker:
      resources:
        requests:
          memory: "2Gi"
          cpu: "500m"
        limits:
          memory: "4Gi"
          cpu: "1000m"
    ```
- Helm을 사용하여 Trino를 설치합니다.
  - ```shell
    helm install trino . -f values.yaml -n hive
    # helm upgrade trino trino/trino -f values.yaml -n hive
    # kubectl delete pods -l app=trino -n hive
    ```

## Trino로 MinIO 데이터 쿼리하기
- Trino CLI에 접속하여 MinIO 카탈로그를 확인합니다.
  - ```shell
    export POD_NAME=$(kubectl get pods --namespace hive -l "app=trino,release=trino,component=coordinator" -o jsonpath="{.items[0].metadata.name}")
    # kubectl port-forward $POD_NAME 8080:8080 -n hive
    kubectl exec -it $POD_NAME -n hive -- trino
    trino> SHOW CATALOGS;
    Catalog
    ---------
    minio
    system
    tpcds
    tpch
    (4 rows)

    Query 20231219_071108_00000_7zusv, FINISHED, 2 nodes
    Splits: 8 total, 8 done (100.00%)
    11.91 [0 rows, 0B] [0 rows/s, 0B/s]
    ```
- MinIO의 스키마를 확인하고 minio의 데이터를 불러와서 테이블로 만듭니다.
  - ```shell
    trino> SHOW SCHEMAS FROM minio;
          Schema
    --------------------
    default
    information_schema
    (2 rows)

    Query 20231219_071349_00001_7zusv, FINISHED, 3 nodes
    Splits: 8 total, 8 done (100.00%)
    4.16 [2 rows, 35B] [0 rows/s, 8B/s]

    trino> SHOW TABLES FROM minio.default;
    Table
    -------
    (0 rows)

    Query 20231219_071519_00002_7zusv, FINISHED, 3 nodes
    Splits: 8 total, 8 done (100.00%)
    2.76 [0 rows, 0B] [0 rows/s, 0B/s]

    trino> CREATE SCHEMA minio.l0
      -> WITH (
      ->     location = 's3a://trino/'
      -> );
      ->
    CREATE SCHEMA

    trino> CREATE TABLE IF NOT EXISTS minio.l0.nyc_data (
        ->     VendorID varchar,
        ->     passenger_count varchar,
        ->     trip_distance varchar
        -> )
        -> WITH (
        ->   external_location = 's3a://trino/',
        ->   format = 'CSV'
        -> );
    CREATE TABLE
    ```
  - ```shell
    trino> SELECT * FROM minio.l0.nyc_data;
    ```
  - ![image](https://github.com/mjs1995/muse-data-engineer/assets/47103479/a0ca43e3-c49d-40fa-8c67-80d57f481131)
- trino UI 실행되는 각 쿼리에 대한 세부 정보도 제공합니다. 리소스 활용도, 타임라인, 단계, 작업자가 수행한 작업 등의 세부 정보를 확인할 수 있습니다.
  - ![image](https://github.com/mjs1995/muse-data-engineer/assets/47103479/020c8f44-9820-4d44-8c50-10323f3815f2)
  - ![image](https://github.com/mjs1995/muse-data-engineer/assets/47103479/05966ed2-0e72-45f6-be0b-50441f8bfeba)
