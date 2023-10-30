# Quota 'SSD_TOTAL_GB' exceeded. Limit: 500.0 in region
- Airflow의 작업자(worker) 및 트리거(triggerer) 파드가 Pending 상태에 머무는 에러가 발생하여 이를 해결하고자 합니다.
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
  - 현재 사용 중인 지역(asia-northeast3) 외에 다른 지역에서 클러스터를 생성하고 클러스터에 파드가 필요로 하는 리소스 또한 업그레이드하였습니다.
    - ```shell
      gcloud container clusters create gke-airflow \
        --machine-type n1-standard-2 \
        --region us-west1 \
        --node-locations us-west1-c \
        --num-nodes 2
      ```
  - 다른 지역에 리소스가 여유가 있어서 해당 클러스터가 잘 생성된 것을 확인하였습니다.
    - ```shell
      mun_js@cloudshell:~ (ggke-401900)$ kubectl get pods -n airflow
      NAME                                 READY   STATUS    RESTARTS      AGE
      airflow-postgresql-0                 1/1     Running   0             4m22s
      airflow-redis-0                      1/1     Running   0             4m22s
      airflow-scheduler-77b484b6d5-5nvh5   2/2     Running   0             4m23s
      airflow-statsd-7d985bcb6f-6wghh      1/1     Running   0             4m23s
      airflow-triggerer-0                  2/2     Running   0             4m22s
      airflow-webserver-544dc4bfcc-sztwb   1/1     Running   2 (51s ago)   4m23s
      airflow-worker-0                     2/2     Running   0             4m22s
      ```
  - 이 에러에 대해서 조금 더 자세히 파악해 봤습니다. gcp상에서는 [할당량 및 한도 정책](https://cloud.google.com/vpc/docs/quota?hl=ko)이 있습니다. 공정성 보장 및 사용량 급증 방지 등 여러 이유로 리소스 소비를 제한하는데 할당량이 초과되면 대부분의 경우 시스템에서 관련 Google 리소스에 대한 액세스를 즉시 차단하고 수행하려는 작업이 실패합니다.
    - ```shell
      ERROR: (gcloud.container.clusters.create) ResponseError: code=403, message=Insufficient regional quota to satisfy request: resource "SSD_TOTAL_GB": request requires '300.0' and is short '290.0'. project has a quota of '500.0' with '10.0' available
      ```
    - 현재 리전에서 SSD 스토리지의 양이 초과되면 다음과 같은 에러를 확인할 수 있습니다. 전체 할당량: 500.0 GB 중 요청된 SSD 스토리지 양: 300.0 GB이고 현재 사용 가능한 할당량: 10.0 GB로 리소스가 부족하다고 나옵니다. 이를 해결하기 위한 다른 방법을 알려드리겠습니다.
      - 할당량 증가 요청
        - GCP 콘솔에서 해당 리소스의 할당량을 늘리기 위해 요청할 수 있습니다. 이는 몇 시간 내지 며칠이 걸릴 수 있으며, GCP의 승인을 받아야 합니다.
        - GCP 할당량 페이지로 이동한 뒤 SSD_TOTAL_GB 리소스에 대한 할당량 증가를 요청합니다. 대부분의 후기를 보니 무료계정으로는 할당량 증가 요청이 어렵고 서비스가 어느 정도인지 구글이 모니터링하여 증가 여부 메일을 주는 거 같습니다.
      - 다른 지역 선택
        - 필요한 리소스가 더 많이 남아 있는 다른 지역에서 클러스터를 생성합니다. 현재 이 방법으로 해결을 했지만 이 경우, 네트워크 지연시간, 기타 요인들을 고려해야 할 거 같습니다.
      - 기존 리소스 정리
        - 현재 사용 중인 SSD 리소스를 점검하여, 더 이상 사용하지 않거나 필요하지 않은 리소스를 해제함으로써 필요한 SSD 공간을 확보할 수 있습니다.
        - 사용 안 하는 SSD 리소스를 정리하여 SSD 스토리지 용량을 확보하였습니다.
        - ![image](https://github.com/mjs1995/muse-data-engineer/assets/47103479/d4df3afd-f982-445f-9631-a054ae1f8402)
        - ![image](https://github.com/mjs1995/muse-data-engineer/assets/47103479/b3a58753-7977-4349-8c9b-bf826743dc98)
        - 리소스 할당량을 보려면 다음과 같이 명령어를 입력하시면 됩니다.
          - ```shell
            mun_js@cloudshell:~ (ggke-401900)$ kubectl get resourcequota gke-resource-quotas -o yaml -n airflow
            apiVersion: v1
            kind: ResourceQuota
            metadata:
              creationTimestamp: "2023-10-28T04:16:50Z"
              name: gke-resource-quotas
              namespace: airflow
              resourceVersion: "5448"
              uid: c433bc8c-1cf3-4943-bba6-517645e139e2
            spec:
              hard:
                count/ingresses.extensions: "100"
                count/ingresses.networking.k8s.io: "100"
                count/jobs.batch: 5k
                pods: "1500"
                services: "500"
            status:
              hard:
                count/ingresses.extensions: "100"
                count/ingresses.networking.k8s.io: "100"
                count/jobs.batch: 5k
                pods: "1500"
                services: "500"
              used:
                count/ingresses.networking.k8s.io: "0"
                count/jobs.batch: "0"
                pods: "7"
                services: "7"
            ```
- [stackoverflow](https://stackoverflow.com/questions/76848658/airflow-worker-pods-arent-starting-on-gke-cluster-when-just-deploying-airflow-f)를 보니 다른 사용자들도 비슷한 에러를 경험하고 있는 거 같습니다. 근본적인 해결책은 아니지만 트러블슈팅을 하는데 도움이 되었으면 좋겠습니다.
