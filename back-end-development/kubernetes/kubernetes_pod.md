# 쿠버네티스 파드: 쿠버네티스에서 컨테이너 실행 
- 파드
  - ![httpsdataninjago com20220111spark-sql-query-engine-deep-dive-11-join-strategies (19)](https://user-images.githubusercontent.com/47103479/230708134-934802fe-e09b-44cb-ad6c-1a03e9e237ce.png)
    - https://kubernetes.io/de/docs/tutorials/kubernetes-basics/explore/explore-intro/
  - Kubernetes에서 생성하고 관리할 수 있는 배포가능한 가장 작은 단위
  - 파드는 함께 배치된 컨테이너 그룹이며 쿠버네티스의 기본 빌딩 블록이며 파드 안에 있는 모든 컨테이너는 같은 노드에서 실행됩니다.
  - 하나의 컨테이너를 개별적으로 배포하는 것이 아닌 Pod 단위로 배포
  - 가장 기본적인 배포 단위로 하나 이상의 컨테이너를 포함하는 단위로 일반적으로 1 Pod 1 Container
  - Pod 내의 컨테이너들은 IP, Port를 공유합니다. Pod가 재시작되면 IP가 변경되며 Pod내의 컨테이너들의 로컬디스크의 내용이 사라집니다.
- YAML 또는 JSON 디스크크립터로 파드 생성 
  - 파드를 포함한 다른 쿠버네티스 리소스는 일반적으로 쿠버네티스 REST API 엔드포인트에 JSON 혹은 YAML 매니페스트를 전송해 생성합니다.
  - 배포된 파드의 전체 YAML
    - ```yaml
      apiVersion: v1
      kind: Pod
      metadata:
        name: kubia-manual
      spec:
        containers:
        - image: luksa/kubia
          name: kubia
          ports:
          - containerPort: 8080
            protocol: TCP
      ```
      - YAML이 디스크립터에서 사용한 쿠버네티스 API버전
      - 쿠버네티스 오브젝트/리소스 유형 
      - 파드 메타데이터(이름, 레이블, 어노테이션 등) 
      - 파드 정의/내용(파드 컨테이너 목록, 볼륨 등) 
      - 파드와 그 안의 여러 컨테이너의 상세한 상태 
  - 파드를 정의하는 주요 부분 소개 
    - Metadata: 이름, 네임스페이스, 레이블 및 파드에 관한 기타 정보를 포함합니다.
    - Spec : 파드 컨테이너, 볼륨, 기타 데이터 등 파드 자체에 관한 실제 명세를 가집니다.
    - Status: 파드 상태, 각 컨테이너 설명과 상태, 파드 내부 IP, 기타 기본 정보 등 현재 실행 중인 파드에 관한 현재 정보를 포함합니다. 
- 레이블을 이용한 파드 구성 
  - 마이크로서비스 아키텍처의 경우 배포된 마이크로서비스의 수는 매우 쉽게 20개를 초과합니다. 이러한 구성 요소는 복제돼(동일한 구성 요소의 여러 복사본이 배포됨) 여러 버전 혹은 릴리스(안정, 베타, 카나리 등)가 동시에 실행됩니다.
  - 레이블을 통해 파드와 기타 다른 쿠버네티스 오브젝트의 조직화가 이뤄집니다.
  - 레이블 소개
    - ![image](https://user-images.githubusercontent.com/47103479/230552968-13ed6333-1bcb-401e-83f3-1a4b19a1f399.png) 
      - https://github.com/luksa/kubernetes-in-action
    - 레이블은 파드와 모든 다른 쿠버네티스 리소스를 조직화할 수 있는 단순하면서 강력한 쿠버네티스 기능으로 레이블은 리소스에 첨부하는 키-값 쌍입니다. 이 쌍은 레이블 셀럭터를 사용해 리소스를 선택할 때 활용됩니다.
      - app : 파드가 속한 애플리케이션, 구성 요소 혹은 마이크로서비스를 지정합니다. 
      - rel : 파드에서 실행 중인 애플리케이션이 안정, 베타 혹은 카나리 릴리스인지 보여줌(카나리 릴리스는 안정 버전 옆에 새 버전을 배포하고, 모든 사용자에게 배포하기 전에 소수의 사용자만이 새로운 버전을 사용할 수 있도록 해서 어떻게 동작하는지 볼 수 있게 하는 것)
- 파드에 어노테이션 달기 
  - ```yaml
    apiVersion: v1
    kind: Pod
    metadata:
      name: annotations-demo
      annotations:
        imageregistry: "https://hub.docker.com/"
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
    ```
  - 파드 및 다른 오브젝트는 레이블 외에 어노테이션(annotations)을 가질 수 있습니다. 어노테이션은 키-값 쌍으로 레이블과 거의 비슷하지만 식별 정보를 갖지 않습니다.
  - 어노테이션이 유용하게 사용되는 경우는 파드나 다른 API 오브젝트에 설명을 추가해 두는 것으로 클러스터를 사용하는 모든 사람이 개별 오브젝트에 관한 정보를 신속하게 찾아볼 수 있습니다.
- 네임스페이스를 사용한 리소스 그룹화 
  - ```yaml
    apiVersion: v1
    kind: Namespace
    metadata:
      name: custom-namespace
    ```
  - 오브젝트를 겹치지 않는 그룹으로 분할하고자 할 때 한 번에 하나의 그룹 안에서만 작업하고 싶을 것입니다. 쿠버네티스는 오브젝트를 네임스페이스로 그룹화합니다.
  - 쿠버네티스 네임스페이스는 오브젝트 이름의 범위를 제공합니다. 모든 리소스를 하나의 단일 네임스페이스에 두는 대신에 여러 네임스페이스로 분할할 수 있으며, 분리된 네임스페이스는 같은 리소스 이름을 다른 네임스페이스에 걸쳐 여러 번 사용할 수 있게 해 줍니다.
  - 네임스페이스의 필요성 
    - 여러 네임스페이스를 사용하면 많은 구성 요소를 가진 복잡한 시스템을 좀 더 작은 개별 그룹으로 분리할 수 있습니다. 
    - 멀티테넌트(multi-tenant) 환경처럼 리소스를 분리하는 데 사용됩니다.

# Reference 
- https://www.oreilly.com/library/view/kubernetes-in-action/9781617293726/
- https://www.oreilly.com/library/view/cloud-native-devops/9781492040750/
