# 볼륨: 컨테이너에 디스크 스토리지 연결 
- 스토리지 볼륨은 파드와 같은 최상위 리소스는 아니지만 파드의 일부분으로 정의되며 파드와 동일한 라이프사이클을 가집니다. 파드가 시작되면 볼륨이 생성되고, 파드가 삭제되면 볼륨이 삭제된다는 것을 의미합니다.
- 볼륨
  - ![image](https://user-images.githubusercontent.com/47103479/230571873-eec9bc52-1758-46e4-aadc-bc4817b56a09.png)
  - 쿠버네티스 볼륨은 파드의 구성 요소로 컨테이너와 동일하게 파드 스펙에서 정의됩니다. 볼륨은 독립적인 쿠버네티스 오브젝트가 아니므로 자체적으로 생성, 삭제될 수 없습니다. 
  - 데이터를 담는 디렉터리로 Pod 내 컨테이너들이 접근가능합니다.
  - Pod에 소속되는 동안 유지됩니다. Pod 내에서 구동되는 컨테이너들보다 오래 유지되며, 그 데이터는 컨테이너가 재시작되더라도 계속 보존됩니다.
  - iSCSI나 NFS와 같은 Onpremiss기반의 외장 스토리지 이외에 클라우드 외장 스토리지인 AWS EBS, Google PD 그리고 github, 등의 오픈소스 기반 외장스토리 서비스를 지원합니다.
  - 사용 가능한 볼륨 유형 소개 
    - emptyDir : 일시적인 데이터를 저장하는 데 사용되는 간단한 빈 디렉터리 
    - hostPath : 워커 노드의 파일시스템을 파드의 디렉터리로 마운트 하는 데 사용함 
    - gitRepo : 깃 리포지터리의 콘텐츠를 체크아웃해 초기화환 볼륨 
      - 생성 시에 지정된 git 레파지토리의 특정 리비전의 내용을 clone을 이용해서 내려받은 후에 디스크 볼륨을 생성하는 방식
      - 물리적으로 emptyDir이 생성되고, git 레파지토리 내용을 clone으로 다운    
    - nfs :NFS 공유를 파드에 마운트함 
    - gcePersistentDisk(Google Compute Engine Persistent Disk), awsElasticBlock Store(Amazon Web Services Elastic Block Store Volume), azureDisk(Microsoft Azure Disk Volume): 클라우드 제공자의 전용 스토리지를 마운트하는 데 사용합니다.
    - cinder, cephfs, iscsi, flocker, glusterfs, quobyte, rdb, flexVolume, vsphere Volume, photonPersistentDisk, ScaleIO : 다른 유형의 네트워크 스토리지를 마운트 하는 데 사용합니다. 
    - configMap, secret, downwardAPI : 쿠버네티스 리소스나 클러스터 정보를 파드에 노출하는 데 사용되는 특별한 유형의 볼륨 
    - persistentVolumeClaim : 사전에 혹은 동적으로 프로비저닝 된 퍼시스턴트 스토리지를 사용하는 방법 
      - 클러스터 내 스토리지 일부 조각을 나타내는 API 객체로 개별 Pod의 수명주기를 넘어 보존되는 범용, 플러그가능 자원으로서 가용합니다.
      - 볼륨 자체를 의미하는 PV들은 스토리지를 사용하고자 할 때 그 제공방식의 세부사항들을 추상화하는 API를 제공합니다.
      - 스토리지가 사전에 생성되는 시나리오(정적 프로비저닝)에서는 PV들이 직접 사용됩니다. 반면 온디멘트 스토리지를 필요로 하는 시나리오(동적 프로비저닝)에서는 PV 대신 PVC가 사용됩니다.
      - PV는 파드하고는 별개로 관리되고 별도의 생명주기를 가지고 있으며, PVC는 사용자가 PV에 하는 요청
- 볼륨을 사용한 컨테이너 간 데이터 공유
  - emptyDir 볼륨 사용 
    - ```yaml
      apiVersion: v1
      kind: Pod
      metadata:
        name: fortune
      spec:
        containers:
        - image: luksa/fortune
          name: html-generator
          volumeMounts:
          - name: html
            mountPath: /var/htdocs
        - image: nginx:alpine
          name: web-server
          volumeMounts:
          - name: html
            mountPath: /usr/share/nginx/html
            readOnly: true
          ports:
          - containerPort: 80
            protocol: TCP
        volumes:
        - name: html
          emptyDir: {}
      ```
    - Pod가 생성될 때 생성되고, Pod가 삭제될 때 같이 삭제되는 임시 볼륨으로 생성 당시에는 디스크에 아무 내용이 없기 때문에 emptyDir, 단 Pod 내의 컨테이너 크래쉬 되어 삭제되거나 재시작되더라도 emptyDir의 생명주기는 컨테이너 단위가 아니라 Pod 단위이기 때문에 emptyDir은 삭제되지 않고 계속해서 사용이 가능합니다.  
    - 볼륨이 빈 디렉터리로 시작됩니다. 파드에 실행중인 애플리케이션은 어떤 파일이든 볼륨에 쓸 수 있고 볼륨의 라이플사이클이 파드에 묶여 있으므로 파드가 삭제되면 볼륨의 콘텐츠는 사라집니다. 
    - emptyDir 볼륨은 동일 파드에서 실행 중인 컨테이너 간 파일을 공유할 때 유용합니다.
    - 사이드카 컨테이너 소개
      - ![image](https://user-images.githubusercontent.com/47103479/233387859-2d399077-618f-4435-9770-de7d794305c7.png)
        - https://matthewpalmer.net/kubernetes-app-developer/articles/multi-container-pod-design-patterns.html#sidecar-pattern-example
      - 사이드카 컨테이너는 파드의 주 컨테이너의 동작을 보완합니다.
      - 새로운 로직을 메인 애플리케이션 코드에 밀어 넣어 복잡성을 더하고 재사용성을 떨어뜨리는 대신에 파드에 사이드카를 추가하면 기존 컨테이너 이미지를 사용할 수 있습니다.   
- 워커 노드 파일 시스템의 파일 접근
  - hostPath 볼륨
    - ![image](https://user-images.githubusercontent.com/47103479/230862512-19ec4eed-2570-4c0c-bc15-dacdd333c63d.png)
    - hostPath 볼륨은 노드 파일시스템의 특정 파일이나 디렉터리를 가리킵니다. 동일 노드에 실행 중인 파드가 hostPath 볼륨의 동일 경로를 사용 중이며 동일한 파일이 표시됩니다.
    - gitRepo나 emptyDir 볼륨의 콘텐츠는 파드가 종료되면 삭제되는 반면, hostPath 볼륨의 콘텐츠는 삭제되지 않습니다. 파드가 삭제되면 다음 파드가 호스트의 동일 경로를 가리키는 hostPath 볼륨을 사용하고, 이전 파드와 동일한 노드에 스케줄링된다는 조건에서 새로운 파드는 이전 파드가 남긴 모든 항목을 볼 수 있습니다. 
    - 노드의 로컬 디스크의 경로를 Pod에서 마운트 해서 사용합니다. 같은 hostPath에 있는 볼륨은 여러 Pod 사이에서 공유되어 사용
- 기반 스토리지 기술과 파드 분리 - 퍼시스턴트볼륨과 퍼시스턴트볼륨클레임
  - 인프라스트럭처의 세부 사항을 처리하지 않고 애플리케이션이 쿠버네티스 클러스터에 스토리지를 요청할 수 있도록 하기 위해 새로운 리소스 두 개가 도입됍니다. 퍼시스턴트볼륨(PV, PersistentVolume)과 퍼시스턴트볼륨클레임(PVC, PersistentVolumeClaim)
  - 퍼시스턴트볼륨 생성 
    - ![image](https://user-images.githubusercontent.com/47103479/230865633-037d0977-4848-4d83-8c40-43c2b79b5901.png)
      - PersistentVolume은 클러스터 관리자가 프로비저닝 하고 PersistentVolumeClaim을 통해 포드에서 사용합니다.
    - ```yaml
      apiVersion: v1
      kind: PersistentVolume
      metadata:
        name: mongodb-pv
      spec:
        capacity:
          storage: 1Gi
        accessModes:
        - ReadWriteOnce
        - ReadOnlyMany
        persistentVolumeReclaimPolicy: Retain
        gcePersistentDisk:
          pdName: mongodb
          fsType: ext4
      ```
    - 퍼시스턴트볼륨을 생성할 때 관리자는 쿠버네티스에게 용량이 얼마가 되는지 단일 노드나 동시에 다수 노드에 읽기나 쓰기가 가능한지 여부를 알려야 합니다. 또한 쿠버네티스에게 퍼시스턴트볼륨이 해제되면 어떤 동작을 해야 할지 알려야 합니다.(바인딩된 퍼시스턴트볼륨클레임이 삭제되는 경우)
    - 퍼시스턴트볼륨을 지원하는 실제 스토리지의 유형, 위치, 그 밖의 속성 정보를 지정해야 합니다.
    - ![image](https://user-images.githubusercontent.com/47103479/230865922-9392a6cc-0098-4708-9577-bc38cd0d6226.png)
      - PersistentVolume은 클러스터 노드와 마찬가지로 포드 및 PersistentVolumeClaim과 달리 네임스페이스에 속하지 않습니다
  - 퍼시스턴트볼륨클레임 생성을 통한 퍼시스턴트볼륨 요청 
    - ```yaml
      apiVersion: v1
      kind: Pod
      metadata:
        name: mongodb
      spec:
        containers:
        - image: mongo
          name: mongodb
          volumeMounts:
          - name: mongodb-data
            mountPath: /data/db
          ports:
          - containerPort: 27017
            protocol: TCP
        volumes:
        - name: mongodb-data
          persistentVolumeClaim:
            claimName: mongodb-pvc
      ```
    - 퍼시스턴트볼륨클레임이 생성되자마자 쿠버네티스는 적절한 퍼시스턴트볼륨을 찾고 클레임에 바인딩합니다. 퍼시스턴트볼륨의 용량은 퍼시스턴트볼륨클레임의 요청을 수용할 만큼 충분히 커야 합니다.
      - RWO(ReadWriteOnce) : 단일 노드만이 읽기/쓰기용으로 볼륨을 마운트 할 수 있습니다.
      - ROX(ReadOnlyMany) : 다수 노드가 읽기용으로 볼륨을 마운트 할 수 있습니다.
      - RWX(ReadWriteMany) : 다수 노드가 읽기/쓰기용으로 볼륨을 마운트 할 수 있습니다. 

# Reference
- https://www.oreilly.com/library/view/kubernetes-in-action/9781617293726/
- https://www.oreilly.com/library/view/cloud-native-devops/9781492040750/
