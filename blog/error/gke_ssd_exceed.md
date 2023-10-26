# Quota 'SSD_TOTAL_GB' exceeded. Limit: 500.0 in region
- Airflow의 작업자(worker) 및 트리거러(triggerer) 파드가 Pending 상태에 머무는 에러가 발생하여 이를 해결하고자 합니다.
- Airflow 환경을 GKE 클러스터에 구축한 후, 특정 Airflow 컴포넌트들이 실행되지 않는 문제가 발생했습니다. 
  - ```shell
    gcloud container clusters create gke-airflow \
    --machine-type e2-medium \
    --num-nodes 1 \
    --region "asia-northeast3" \
    --min-nodes 1 \
    --max-nodes 3
    ```
- 파드 상태를 확인했을 때, airflow-worker-0 및 airflow-triggerer-0 파드들이 Pending 상태에 머물고 있었습니다.
  - ```shell
    mun_js@cloudshell:~ (ggke-401900)$ kubectl get pods -n airflow
    NAME                                 READY   STATUS    RESTARTS   AGE
    airflow-postgresql-0                 1/1     Running   0          19m
    airflow-redis-0                      1/1     Running   0          19m
    airflow-scheduler-849bbd5757-lkt6d   3/3     Running   0          19m
    airflow-statsd-7d985bcb6f-g76j8      1/1     Running   0          19m
    airflow-triggerer-0                  0/3     Pending   0          19m
    airflow-webserver-55f8b57964-nn6xq   1/1     Running   0          19m
    airflow-worker-0                     0/3     Pending   0          19m
    ```
- 스토리지 클래스 확인
  - 해당 PVC가 요청하는 StorageClass (standard-rwo이 사용 가능한지, 즉 이 StorageClass에 해당하는 동적 프로비저닝이 작동하고 있는지 확인합니다. GKE에서는 일반적으로 여러 StorageClass가 사전에 구성되어 있으며, 필요에 따라 적절한 StorageClass를 선택해야 합니다.
  - ```shell
    mun_js@cloudshell:~ (ggke-401900)$ kubectl get storageclass
    NAME                     PROVISIONER             RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
    premium-rwo              pd.csi.storage.gke.io   Delete          WaitForFirstConsumer   true                   23m
    standard                 kubernetes.io/gce-pd    Delete          Immediate              true                   23m
    standard-rwo (default)   pd.csi.storage.gke.io   Delete          WaitForFirstConsumer   true                   23m
    ```
- PVC 상태 확인
  - logs-airflow-triggerer-0, logs-airflow-worker-0 pvc가 Bound 상태가 아니라 Pending 상태이므로 파드가 필요한 스토리지를 할당받지 못하고 있습니다.
  - ```shell
    mun_js@cloudshell:~ (ggke-401900)$ kubectl get pvc -n airflow
    NAME                        STATUS    VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
    data-airflow-postgresql-0   Bound     pvc-40faa475-1412-4260-b8fa-aecd9808092f   8Gi        RWO            standard-rwo   7m55s
    logs-airflow-triggerer-0    Pending                                                                        standard-rwo   7m54s
    logs-airflow-worker-0       Pending                                                                        standard-rwo   7m54s
    redis-db-airflow-redis-0    Bound     pvc-8f15e758-608b-4f0f-9f55-b2b871dd8a3d   1Gi        RWO            standard-rwo   7m54s
    ```
  - PVC가 Pending 상태인 원인을 파악하기 위해 해당 PVC의 이벤트를 확인합니다.
  - ```shell
    mun_js@cloudshell:~ (ggke-401900)$ kubectl describe pvc logs-airflow-worker-0 -n airflow
    Access Modes:
    VolumeMode:    Filesystem
    Used By:       airflow-worker-0
    Events:
      Type     Reason                Age                   From                                                                                              Message
      ----     ------                ----                  ----                                                                                              -------
      Normal   WaitForFirstConsumer  19m                   persistentvolume-controller                                                                       waiting for first consumer to be created before binding
      Warning  ProvisioningFailed    19m                   pd.csi.storage.gke.io_gke-643502a388d341438f81-a7b2-6933-vm_125cf7b4-6600-43e0-8c95-43de8ac74897  failed to provision volume with StorageClass "standard-rwo": rpc error: code = Internal desc = CreateVolume failed to create single zonal disk pvc-08dbf8af-4b18-4dd0-a3bc-2ae2291d34a0: failed to insert zonal disk: unknown Insert disk operation error: rpc error: code = ResourceExhausted desc = operation operation-1698211795430-60883bed1eac7-9b4c88a7-1dba47b4 failed (QUOTA_EXCEEDED): Quota 'SSD_TOTAL_GB' exceeded.  Limit: 500.0 in region asia-northeast3.
    ```
- 해결
  - Kubernetes에서 Persistent Volume Claim (PVC) logs-airflow-worker-0을 생성하려고 하는데, 이 PVC가 "standard-rwo" StorageClass를 사용하고 있으며, 이 StorageClass는 Google Kubernetes Engine(GKE)의 SSD를 사용하고 있습니다. 여기 "ProvisioningFailed" 이벤트 로그에서 Quota 'SSD_TOTAL_GB' exceeded. Limit: 500.0 in region asia-northeast3.라는 메시지가 나타나고 있어, 해당 지역(asia-northeast3)에서 설정된 SSD 총 사용량 한도(500GB)를 초과했습니다.
  - 현재 사용 중인 지역(asia-northeast3) 외에 다른 지역에서 클러스터를 생성하고 클러스터에 파드가 필요로 하는 리소스 또한 업그레이드 하였습니다.
    - ```shell
      gcloud container clusters create gke-airflow \
        --machine-type n1-standard-2 \
        --region us-west1 \
        --node-locations us-west1-c \
        --num-nodes 2
      ```
