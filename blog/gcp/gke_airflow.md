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
