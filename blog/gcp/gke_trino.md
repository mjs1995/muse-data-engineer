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