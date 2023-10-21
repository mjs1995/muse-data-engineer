# airflow 배포 
- Google Kubernetes Engine (GKE)에 helm 차트를 활용해서 airflow 배포를 하는법에 대해서 포스팅하려고 합니다.
- gcloud는 Google Cloud Platform(GCP) 리소스를 관리하고 조작하는데 사용되는 커맨드 라인 인터페이스(CLI) 도구입니다. gcloud 명령어를 사용하면 GCP의 다양한 서비스와 리소스를 생성, 수정, 삭제 및 관리할 수 있습니다.
- GCP의 기본 작업 프로젝트를 ggke-401900로 설정합니다
  - ```shell
    mun_js@cloudshell:~ (ggke-401900)$ gcloud config set project ggke-401900
    Updated property [core/project].
    ```
- "asia-northeast3" (서울) 지역에 "airflow"라는 이름의 GKE 클러스터를 생성하며, 클러스터에는 n1-standard-4 머신 유형의 하나의 노드가 포함합니다.
  - ```shell
    mun_js@cloudshell:~ (ggke-401900)$ gcloud container clusters create gke-airflow \
    --machine-type n1-standard-4 \
    --num-nodes 1 \
    --region "asia-northeast3"
    ```
- gke 클러스터의 인증 정보를 가져와 로컬 시스템의 kubeconfig 파일에 저장합니다. 이때 kubeconfig 파일에 클러스터의 인증 정보가 추가되기 때문에 kubectl을 사용하여 클러스터 리소스를 생성, 수정, 삭제하는 등의 작업을 수행할 수 있게 됩니다.
  - ```shell
    mun_js@cloudshell:~ (ggke-401900)$ gcloud container clusters get-credentials gke-airflow --region "asia-northeast3"
    mun_js@cloudshell:~ (ggke-401900)$ helm repo add apache-airflow https://airflow.apache.org
    "apache-airflow" has been added to your repositories
    mun_js@cloudshell:~ (ggke-401900)$ helm repo list
    NAME            URL
    apache-airflow  https://airflow.apache.org
    ```
- Airflow를 GKE에 배포합니다.
  - ```shell
    helm upgrade --install airflow apache-airflow/airflow -n airflow --debug

    You can get Fernet Key value by running the following:

    echo Fernet Key: $(kubectl get secret --namespace airflow airflow-fernet-key -o jsonpath="{.data.fernet-key}" | base64 --decode)

    ###########################################################
    #  WARNING: You should set a static webserver secret key  #
    ###########################################################
    
    You are using a dynamically generated webserver secret key, which can lead to
    unnecessary restarts of your Airflow components.
    
    Information on how to set a static webserver secret key can be found here:
    https://airflow.apache.org/docs/helm-chart/stable/production-guide.html#webserver-secret-key
    ```
- 웹서버의 시크릿 키를 동적으로 생성하는 것에 대해 에러 메시지를 반환하고 있어 정적인 웹서버 시크릿 키를 설정합니다.
- Secret을 airflow 네임스페이스로 이동하기 위해서 먼저 Secret을 YAML 파일로 저장합니다.
  - ```shell
    mun_js@cloudshell:~ (ggke-401900)$ kubectl get secret my-webserver-secret --namespace=default -o yaml > my-webserver-secret.yaml
    mun_js@cloudshell:~ (ggke-401900)$ ls
    my-webserver-secret.yaml  README-cloudshell.txt  values.yaml
    ```
- YAML 파일에서 네임스페이스 항목을 변경합니다.
  - ```shell
    mun_js@cloudshell:~ (ggke-401900)$ sed -i 's/namespace: default/namespace: airflow/' my-webserver-secret.yaml
    ```
- 변경된 YAML 파일을 새로운 네임스페이스에 적용합니다.
  - ```shell
    mun_js@cloudshell:~ (ggke-401900)$ kubectl apply -f my-webserver-secret.yaml
    secret/my-webserver-secret created
    ```
- Secret이 올바른 네임스페이스에 있는지 확인한 후, Helm 업그레이드를 다시 시도합니다.
  - ```shell
    helm upgrade --install airflow apache-airflow/airflow -n airflow --set webserverSecretKeySecretName=my-webserver-secret --debug
    NOTES:
    Thank you for installing Apache Airflow 2.7.1!
    
    Your release is named airflow.
    You can now access your dashboard(s) by executing the following command(s) and visiting the corresponding port at localhost in your browser:
    
    Airflow Webserver:     kubectl port-forward svc/airflow-webserver 8080:8080 --namespace airflow
    Default Webserver (Airflow UI) Login credentials:
        username: admin
        password: admin
    Default Postgres connection credentials:
        username: postgres
        password: postgres
        port: 5432
  
    You can get Fernet Key value by running the following:
  
      echo Fernet Key: $(kubectl get secret --namespace airflow airflow-fernet-key -o jsonpath="{.data.fernet-key}" | base64 --decode)
    ```
- 이제 Airflow의 웹 UI에 접근할 수 있게 됩니다.
  - ![image](https://github.com/mjs1995/muse-data-engineer/assets/47103479/9db8dd3f-8286-4ae8-bbd6-0be233f29097)

# Terraform을 활용한 Airflow 배포 
- 테라폼을 활용해서 gke 클러스터를 생성하고 airflow helm차트를 배포해보려고 합니다.
## 서비스 계정 생성 및 역할 부여
- Service account 란 사용자를 대신하여 작업을 수행하는, 프로젝트에 연결된 Google 계정이며 이러한 Service account 에는 사용자와 동일한 방식으로 역할과 권한을 할당 할 수 있습니다.
- 테라폼 명령을 실행하기 전에 아래 역할(Role)을 가지고 있는 Service account를 생성합니다.
  - [서비스 계정 관리자 (roles/iam.serviceAccountAdmin)](https://cloud.google.com/iam/docs/understanding-roles#service-accounts-roles) : 서비스 계정과 관련된 모든 작업에 대한 권한을 제공합니다. 이 역할은 서비스 계정의 생성, 삭제, 업데이트, 키 관리 등의 작업을 허용합니다.
  - [서비스 계정 사용자 (roles/iam.serviceAccountUser)](https://cloud.google.com/iam/docs/service-account-permissions?hl=ko#roles) : 이 역할은 서비스 계정의 자격 증명을 사용하여 애플리케이션을 인증하는 데 필요합니다. 특히, GKE 클러스터는 해당 서비스 계정의 자격 증명을 사용하여 GCP 서비스와 상호 작용해야 하므로 이 권한이 필요합니다.
  - [Kubernetes Engine 관리자 (roles/container.admin)](https://cloud.google.com/kubernetes-engine/docs/how-to/iam?hl=ko#predefined) : GKE 리소스에 대한 모든 관리 작업에 대한 권한을 제공합니다. 클러스터 생성, 삭제, 업데이트 및 관련된 모든 작업을 수행할 수 있습니다.
  - [Kubernetes Engine 클러스터 관리자(roles/container.clusterAdmin)](https://cloud.google.com/kubernetes-engine/docs/concepts/access-control?hl=ko#recommendations) : GKE 클러스터의 생성, 삭제, 업데이트 및 관련된 모든 작업을 수행할 수 있는 권한을 제공합니다
  - [Compute 인스턴스 관리자(v1)](https://cloud.google.com/compute/docs/access/iam?hl=ko#compute.instanceAdmin.v1) : 인스턴스 그룹을 관리하는 도구로, 특정 수의 인스턴스를 유지하도록 설정하거나 특정 상황에서 자동으로 인스턴스를 추가/제거하도록 구성할 수 있습니다.
  - [저장소 개체 관리자(roles/storage.objectAdmin)](https://cloud.google.com/storage/docs/access-control/iam-roles?hl=ko) : Google Cloud Storage의 개체와 버킷에 대한 액세스 권한을 제공합니다. 이 역할은 버킷 내의 개체를 생성, 볼, 수정, 삭제하는 데 필요한 권한을 부여합니다.
 
## terraform 코드 구조
- cloudshell에서 github 저장소의 코드를 clone 합니다.
- ```shell
  git clone https://github.com/mjs1995/gcp-de-pipeline.git
  ```
- 프로젝트의 구조는 다음과 같습니다.
- ```shell
  terraform-airflow-gke/
  │
  ├── main.tf
  ├── airflow.tf
  ├── variables.tf
  └── terraform.tfvars
  ```
  - main.tf
    - 주요 Terraform 구성을 포함하며, GKE 클러스터 및 필요한 Google Cloud 리소스를 정의합니다.
    - Providers : Google Cloud, Helm, Kubernetes의 세 가지 provider가 정의되어 있습니다. 각각의 provider는 특정 서비스나 플랫폼과 Terraform 간의 연동을 담당합니다.
    - Resources : 여러 리소스들이 정의되어 있습니다. GKE 클러스터, 서비스 어카운트, GKE 노드 풀, Kubernetes 네임스페이스 및 Helm 차트 등이 포함됩니다.
  - airflow.tf
    - Airflow에 필요한 Kubernetes 및 Helm 리소스를 정의합니다. 예를 들어, Airflow의 Kubernetes 네임스페이스, Helm 차트 등이 포함됩니다.
  - variables.tf
    - 이 파일에는 여러 변수들이 정의되어 있습니다. 이 변수들은 사용자에게 입력을 받거나 기본값을 가질 수 있습니다. 예를 들어, 프로젝트 ID, 클러스터 이름, 노드 풀 설정 등에 사용됩니다.
  - terraform.tfvars
    - variables.tf에서 정의한 변수의 실제 값들을 이 파일에서 설정합니다. 예를 들어, 프로젝트 ID나 노드 풀의 크기 등을 지정할 수 있습니다.

## terraform 코드 실행
- 이제 terraform plan을 실행하여 예상되는 변경사항을 확인하는 것입니다. 변경사항이 예상대로라면, terraform apply를 실행하여 인프라를 실제로 생성하거나 변경합니다.
- 테라폼 명령어에 관해서는 [아래 포스팅]()을 참고하시면 좋을거 같습니다.
  - terraform plan: 이 명령은 Terraform이 실제로 어떤 작업을 할지를 보여줍니다. 이는 적용 전에 어떤 리소스가 생성, 수정, 삭제될 것인지 확인하기 위한 미리보기와 같습니다. 명령을 실행하면, Terraform은 계획한 변경사항을 출력합니다.
  - terraform apply: 이 명령은 실제로 인프라 변경을 수행합니다. 실행 시 변경사항을 확인하고 승인해야 합니다. 변경사항을 승인하면 Terraform은 리소스를 생성, 수정, 삭제하기 시작합니다.
  - terraform destroy: 테스트나 실습 후에 인프라 리소스를 모두 삭제하고 싶을 때 사용합니다. 이 명령을 실행하면 terraform apply로 생성한 모든 리소스를 삭제합니다.
- Terraform 코드는 Terraform이 Kubernetes 리소스를 관리하기 위한 설정을 제공하며, 이와 별개로 kubectl이 올바른 kubeconfig 정보를 가지고 있어야 클러스터와 통신할 수 있습니다.
- ```shell
  mun_js@cloudshell:~ (ggke-401900)$ gcloud container clusters get-credentials my-gke-cluster --region=asia-northeast3
  ```
- Airflow가 airflow 네임스페이스에 베포되었고 아래의 명령어로 파드의 상태를 확인합니다.
- ```shell
  mun_js@cloudshell:~ (ggke-401900)$ kubectl get pods -n airflow
  NAME                                     READY   STATUS    RESTARTS   AGE
  airflow-db-migrations-7c7976b4fb-hxqfb   1/1     Running   0          11m
  airflow-flower-dd9c57f94-vsbrg           1/1     Running   0          11m
  airflow-pgbouncer-7467bc948b-sbllf       1/1     Running   0          11m
  airflow-postgresql-0                     1/1     Running   0          11m
  airflow-redis-master-0                   1/1     Running   0          11m
  airflow-scheduler-974b4d64f-z7sx2        2/2     Running   0          11m
  airflow-sync-users-64b6585f95-qrlds      1/1     Running   0          11m
  airflow-triggerer-5dbc7495fb-rlbhb       1/1     Running   0          11m
  airflow-web-6d6759b8cf-5bcf7             1/1     Running   0          11m
  airflow-worker-0                         2/2     Running   0          11m
  ```
- 이제 airflow-web 서비스를 통해 Airflow Web UI에 접근합니다.
- ```shell
  mun_js@cloudshell:~ (ggke-401900)$ kubectl port-forward svc/airflow-web -n airflow 8080:8080
  ````
- 패키지 검색
  - Airflow 웹 서버나 스케줄러의 파드에서 apache-airflow-providers-google 라이브러리가 설치되었는지 확인하려면 파드에 직접 접속해서 Python 패키지 목록에서 apache-airflow-providers-google를 검색합니다.
  - ```shell
    mun_js@cloudshell:~ (ggke-401900)$ kubectl exec -it airflow-web-6d6759b8cf-5bcf7 -n airflow -- /bin/bash
    Defaulted container "airflow-web" out of: airflow-web, check-db (init), wait-for-db-migrations (init)
    airflow@airflow-web-6d6759b8cf-6h8rd:/opt/airflow$ pip list | grep apache-airflow-providers-google
    apache-airflow-providers-google          10.2.0
    
    [notice] A new release of pip is available: 23.1.2 -> 23.3
    [notice] To update, run: python -m pip install --upgrade pip
    ```
- helm 활용
  - helm list 명령을 사용하여 현재 클러스터에 설치된 모든 Helm release를 나열할 수 있습니다.
  - ```shell
    mun_js@cloudshell:~/terraform-airflow-gke (ggke-401900)$ helm list -n airflow
    NAME    NAMESPACE       REVISION        UPDATED                                STATUS   CHART           APP VERSION
    airflow airflow         1               2023-10-18 15:02:06.51287727 +0000 UTC failed   airflow-8.8.0   2.6.3
    ```
  - 특정 Helm release의 값을 확인할 수 있습니다.
  - ```shell
    mun_js@cloudshell:~/terraform-airflow-gke (ggke-401900)$ kubectl get configmap -n airflow
    NAME                   DATA   AGE
    airflow-redis          3      43m
    airflow-redis-health   6      43m
    kube-root-ca.crt       1      4h39m
    ```

## 클러스터 삭제 시
- gke 클러스터를 삭제할 때 현재 Terraform 상태 파일(terraform.tfstate)에 저장된 리소스의 목록을 표시합니다.
  - ```shell
    mun_js@cloudshell:~/terraform-airflow-gke (ggke-401900)$ terraform state list
    data.google_client_config.default
    google_container_cluster.primary
    helm_release.airflow
    kubernetes_namespace.airflow
  
  
    mun_js@cloudshell:~/terraform-airflow-gke (ggke-401900)$ terraform state rm data.google_client_config.default
    Removed data.google_client_config.default
    Successfully removed 1 resource instance(s).
    ```
  - ```shell
    terraform state list
    ```
    - 이 명령어는 현재 Terraform 상태 파일(terraform.tfstate)에 저장된 리소스의 목록을 표시합니다.

  - ```shell
    terraform state rm data.google_client_config.default
    ```
    - terraform state rm 명령어는 Terraform의 상태 파일에서 특정 리소스를 제거하는데 사용됩니다. 여기서는 data.google_client_config.default 리소스를 상태 파일에서 제거하고 있습니다.

  - ```shell
    terraform refresh
    ```
    - 현재 Terraform 코드와 실제 클라우드 환경의 리소스 상태를 비교하여 상태 파일을 최신 상태로 업데이트하려고 합니다.
