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
  </configuration>
    ```
