# GKE의 Apache Airflow에 minio 연동하기
- MinIO는 고성능, 오픈소스 객체 스토리지 서비스로, 클라우드 네이티브 환경에서 매우 유용합니다. 이번 포스팅에서는 Kubernetes 클러스터 상에서 Airflow와 MinIO를 연동하는 과정을 단계별로 살펴보겠습니다.

## MinIO 설치 및 설정
- 먼저, Kubernetes 클러스터에 MinIO를 설치해야 합니다. MinIO는 쿠버네티스 Pod로 실행되며, 이를 위해 YAML 파일을 사용하여 필요한 리소스를 정의합니다.
- MinIO 설치를 위한 minio-dev.yaml 파일 예시입니다
  - ```yaml
    apiVersion: v1
    kind: Namespace
    metadata:
      name: minio-dev
    ---
    apiVersion: v1
    kind: Pod
    metadata:
      labels:
        app: minio
      name: minio
      namespace: minio-dev
    spec:
      containers:
      - name: minio
        image: quay.io/minio/minio:latest
        args: 
        - minio server /data --console-address :9090
      volumes:
      - name: localvolume
        emptyDir: {}
    ---
    apiVersion: v1
    kind: Service
    metadata:
      name: minio-service
      namespace: minio-dev
    spec:
      type: ClusterIP
      ports:
        - port: 9000
          targetPort: 9000
      selector:
        app: minio
    ```
  - 이 파일은 MinIO를 단일 Pod로 실행하며, 클러스터 내부에서만 접근 가능한 서비스를 생성합니다.
- minio-dev.yaml 파일에 정의된 설정을 Kubernetes 클러스터에 적용합니다. 이 파일에는 MinIO를 실행하는 데 필요한 Namespace, Pod 및 Service 설정이 포함되어 있습니다.
  - ```shell
    kubectl apply -f minio-dev.yaml
    ```
- MinIO가 배포된 minio-dev 네임스페이스 내의 모든 파드의 상태를 확인합니다. 여기서 minio 파드가 성공적으로 실행 중인지 확인할 수 있습니다
  - ```shell
    kubectl get pods -n minio-dev
    NAME    READY   STATUS    RESTARTS   AGE
    minio   1/1     Running   0          2d7h
    ```
- MinIO 서비스의 상태를 확인합니다. 서비스는 파드와 외부 또는 다른 파드 간의 네트워크 통신을 관리합니다
  - ```shell
    NAME            TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)    AGE
    minio-service   ClusterIP   10.52.6.113   <none>        9000/TCP   59m    
    ```
## MinIO CLI (mc) 설정
- MinIO를 관리하기 위해 mc (MinIO Client)를 사용합니다. 이 클라이언트를 통해 버킷 생성, 파일 업로드 등 다양한 작업을 수행할 수 있습니다.
- mc 설치 및 설정 과정입니다.
  - ```shell
    wget https://dl.min.io/client/mc/release/linux-amd64/mc
    chmod +x mc
    ./mc --version
    
    ./mc alias set k8s-minio-dev http://127.0.0.1:9000 minioadmin minioadmin
    ```
  - ![image](https://github.com/mjs1995/muse-data-engineer/assets/47103479/a3c08954-37b4-4d38-a0a5-62d0ed58de71)
  - ![image](https://github.com/mjs1995/muse-data-engineer/assets/47103479/411fde5b-11e1-4758-9235-2451b29cb694)
  - MinIO 클라이언트 Alias 설정
    - MinIO 클라이언트(mc)에 k8s-minio-dev라는 이름의 alias(별칭)를 설정합니다. http://127.0.0.1:9000는 MinIO 서버의 주소입니다. 이 주소는 로컬 환경에서 포트 포워딩을 통해 MinIO Pod에 접근하기 위한 것이며 minioadmin은 MinIO의 Access Key와 Secret Key입니다.
    
## Airflow와 MinIO 연동
- Airflow에서 MinIO를 사용하기 위해, Airflow의 PythonOperator를 사용하여 데이터 처리 및 저장 로직을 구현합니다.
- minio 접속 정보에 대해서 Connection을 추가해 줍니다.
- ![image](https://github.com/mjs1995/muse-data-engineer/assets/47103479/f6f44b7e-5032-4bbf-ac80-f44a1ea369f6)
- 다음은 Airflow DAG의 예시입니다.
- ```python
  from airflow import DAG
  from airflow.hooks.base_hook import BaseHook
  from airflow.operators.python_operator import PythonOperator
  from datetime import datetime
  import requests
  import pandas as pd
  import boto3
  import json
  from io import BytesIO
  
  
  def download_and_upload_to_minio():
      url = "https://d37ci6vzurychx.cloudfront.net/trip-data/yellow_tripdata_2023-01.parquet"
      response = requests.get(url)
      data = response.content
      df = pd.read_parquet(BytesIO(data))
      csv_data = df.to_csv(index=False).encode("utf-8")
      conn = BaseHook.get_connection("minio-test")
      extra_config = json.loads(conn.extra)
  
      s3_client = boto3.client(
          "s3",
          endpoint_url=extra_config["host"],
          aws_access_key_id=conn.login,
          aws_secret_access_key=conn.password,
      )
  
      s3_client.put_object(
          Bucket="airflow-minio", Key="nyc_data.csv", Body=BytesIO(csv_data)
      )
  
  
  default_args = {
      "owner": "airflow",
      "start_date": datetime(2023, 1, 1),
      "retries": 1,
  }
  
  with DAG(
      "download_and_upload_to_minio", default_args=default_args, schedule_interval=None
  ) as dag:
  
      upload_task = PythonOperator(
          task_id="download_upload_nyc_data", python_callable=download_and_upload_to_minio
      )
  
  upload_task
  ```
- 이 예시에서는 외부 URL에서 Parquet 파일을 다운로드하고, 이를 CSV 형식으로 변환하여 MinIO에 업로드합니다. Airflow의 PythonOperator를 사용하여 이러한 데이터 처리 및 저장 작업을 자동화합니다.
- ![image](https://github.com/mjs1995/muse-data-engineer/assets/47103479/be3dce0c-d65f-43a8-b957-582df9ec6798)
- minio에 잘 적재가 되었는지 확인해 봅니다.
  - ```shell
    ./mc ls k8s-minio-dev/airflow-minio
    ```
  - ![image](https://github.com/mjs1995/muse-data-engineer/assets/47103479/e79a8ec1-2742-4ab1-94c4-f97610617f32)
