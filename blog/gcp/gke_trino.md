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