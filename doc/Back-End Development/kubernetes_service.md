# 서비스: 클라이언트가 파드를 검색하고 통신을 가능하게 함 
- 서비스 
  - ![image](https://user-images.githubusercontent.com/47103479/230567221-3af29b4c-2a27-40cf-9ab9-c414304f697c.png)
  - Pod 집합과 같은 애플리케이션들에 접근하는 방법을 기술하는 API 객체
  - 쿠버네티스의 서비스는 동일한 서비스를 제공하는 파드 그룹에 지속적인 단일 접점을 만들려고 할 때 생성하는 리소스로 각 서비스는 서비스가 존재하는 동안 절대 바뀌지 않는 IP 주소와 포트가 있습니다.
  - 클라이언트는 해당 IP와 포트로 접속한 다음 해당 서비스를 지원하는 파드 중 하나로 연결됩니다.
  - Service의 Cluster IP를 통해 유동적으로 생성되고 사라지는 Pod에 접근하기 위한 방법으로 사용합니다.
  - ClusterIP , NodePort , LoadBalancer 타입 제공하며 기본 옵션은 클러스터 내부에서만 사용할 수 있는 Cluster IP입니다.
  - 여러 Pod를 묶어 Healthy한 Pod로 Traffic 라우팅하는 로드 밸런싱 기능 제공합니다. 
  - 클러스터의 Service CIDR 중에서 지정된 IP로 생성 가능합니다.
  - ```yaml
    apiVersion: v1
    kind: Service
    metadata:
      name: kubia
    spec:
      ports:
      - port: 80
        targetPort: 8080
      selector:
        app: kubia
    ```
  - 서비스 생성
    - kubectl expose로 서비스 생성 
      - expose 명령어는 레플리케이션컨트롤러에서 사용된 것과 동일한 파드 셀렉터를 사용해 서비스 리소스를 생성하고 모든 파드를 단일 IP 주소와 포트로 노출합니다.
    - 실행 중인 컨테이너에 원격으로 명령어 실행 
      - kubectl exec 명령어를 사용하면 기존 파드의 컨테이너 내에서 원격으로 임의의 명령어를 실행할 수 있습니다. 컨테이너의 내용, 상태, 환경을 검사할 때 유용합니다.
      - kubectl exec {k get pods의 파드} -- curl -s {http://10.111.249.153 - k get svc의 클러스터 IP}
        - 명령어의 더블 대시(--)는 kubectl 명령줄 옵션의 끝을 의미하며 더블 대시 뒤의 모든 것은 파드 내에서 실행돼야 하는 명령어입니다.
- 클러스터 외부에 있는 서비스 연결 
  - ```yaml
    apiVersion: v1
    kind: Endpoints
    metadata:
      name: external-service
    subsets:
      - addresses:
        - ip: 11.11.11.11
        - ip: 22.22.22.22
        ports:
        - port: 80
    ```
  - ![image](https://user-images.githubusercontent.com/47103479/230568664-82738a2c-a7b9-494f-9da3-c862053329b3.png)
  - 서비스 엔드포인트 
    - 서비스는 파드에 직접 연결(link)되지 않습니다. 대신 엔드포인트 리소스가 그 사이에 있습니다.
    - 엔드포인트 리소스는 서비스로 노출되는 파드의 IP 주소와 포트 목록입니다.
  - 외부 서비스를 위한 별칭 생성  
    - ExternalName 서비스 생성 
      - 외부 서비스의 별칭으로 사용되는 서비스를 만들려면 유형(type) 필드를 ExternalName으로 설정해 서비스 리소스를 만듬 
- 외부 클라이언트에 서비스 노출 
  - 외부에서 서비스를 액세스할 수 있는 방법
    - ![image](https://user-images.githubusercontent.com/47103479/230567904-8c0ab938-e16f-4ef0-a63c-e035474829a0.png)
    - 노드포트로 서비스 유형 설정 : 노드포트 서비스의 경우 각 클러스터 노드는 노드 자체에서 포트를 결고 해당 포트로 수신된 트래픽을 서비스로 전달합니다. 이 서비스는 내부 클러스터 IP와 포트로 액세스할 수 있을 뿐만 아니라 모든 노드의 전용 포트로도 액세스할 수 있습니다.
    - 서비스 유형을 노드포트 유형의 확장인 로드밸런서로 설정 : 쿠버네티스가 실행 중인 클라우드 인프라에서 프로비저닝된 전용 로드밸런서로 서비스에 액세스할 수 있습니다. 로드밸런서는 트래픽을 모든 노도의 노드포트로 전달하며 클라이언트는 로드밸런서의 IP로 서비스에 액세스합니다. 
    - 단일 IP 주소로 여러 서비스를 노출하는 인그레스 리소스 만들기 : HTTP 레벨에서 작동하므로 4계층 서비스보다 더 많은 기능을 제공할 수 있습니다 
  - 노드포트 서비스 사용 
    - ![image](https://user-images.githubusercontent.com/47103479/230569400-5c620eea-7626-4c6f-8b46-b5b502114108.png)
    - ```yaml
      apiVersion: v1
      kind: Service
      metadata:
        name: kubia-nodeport
      spec:
        type: NodePort
        ports:
        - port: 80
          targetPort: 8080
          nodePort: 30123
        selector:
          app: kubia
      ```
    - 노드포트 서비스를 만들면 쿠버네티스는 모든 노드에 특정 포트를 할당하고(모든 노드에서 동일한 포트 번호가 사용됨) 서비스를 구성하는 파드로 들어오는 연결을 전달합니다.
    - 일반 서비스(실제 유형은 ClusterIP)와 유사하지만 서비스의 내부 클러스터 IP뿐만 아니라 모든 노드의 IP와 할당된 노드포트로 서비스에 액세스할 수 있습니다.
    - 노드포트 서비스 확인
      - ```shell
        $ kubectl get svc kubia-nodeport
        NAME             CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
        kubia-nodeport   10.111.254.223   <nodes>       80:30123/TCP   2m
        ``` 
      - EXTERNAL-IP에 <nodes>라고 표시돼 있고 클러스터 노드의 IP 주소로 서비스에 액세스할 수 있음을 나타냄 
  - 외부 로드밸런서로 서비스 노출
    - ![image](https://user-images.githubusercontent.com/47103479/230569723-8695bd4a-9d37-4ac9-89c1-4c25451c0c1f.png)
    - 클라우드 공급자에서 실행되는 쿠버네티스 클러스터는 일반적으로 클라우드 인프라에서 로드밸런서를 자동으로 프로비저닝하는 기능을 제공합니다.
    - 로드밸런서는 공개적으로 액세스 가능한 고유한 IP주소를 가지며 모든 연결을 서비스로 전달합니다. 로드밸런서의 IP 주소로 서비스에 액세스할 수 있습니다.
- 인그레스 리소스로 서비스 외부 노출 
  - ![image](https://user-images.githubusercontent.com/47103479/230569898-4c8bb97c-1184-498a-b173-52966c7e6b08.png)
  - 인그레스가 필요한 이유
    - 로드밸런서 서비스는 자신의 공용 IP 주소를 가진 로드밸런서가 필요하지만, 인그레스는 한 IP 주소로 수십 개의 서비스에 접근이 가능하도록 지원해줍니다.
    - 클라이언트가 HTTP 요청을 인그레스에 보낼 때, 요청한 호스트와 경로에 따라 요청을 전달할 서비스가 결정됩니다.
    - 인그레스는 네트워크 스택의 애플리케이션 계층(HTTP)에서 작동하며 서비스가 할 수 없는 쿠키 기반 세션 어피니티 등과 같은 기능을 제공할 수 있습니다.
- 서비스 문제 해결
  - 먼저 외부가 아닌 클러스터 내에서 서비스의 클러스터 IP에 연결되는지 확인합니다. 
  - 서비스에 액세스할 수 있는지 확인하려고 서비스 IP로 핑을 할 필요 없습니다.(서비스의 클러스터 IP는 가상 IP이므로 평되지 않습니다.) 
  - 레디니스 프로브를 정의했다면 성공했는지 확인하라. 그렇지 않으면 파드는 서비스에 포함되지 않습니다.
  - 파드가 서비스의 일부인지 확인하려면 kubectl get endpoints를 사용해 해당 엔드포인트 오브젝트를 확인합니다. 
  - FQDN이나 그 일부(myservice.mynamespace.svc.cluster.local 또는 myservice.mynamespace)로 서비스에 액세스하려고 하는데 작동하지 않는 경우, FQDN 대신 클러스터 IP를 사용해 액세스할 수 있는지 확인합니다. 
  - 대상 포트가 아닌 서비스로 노출된 포트에 연결하고 있는지 확인합니다. 
  - 파드 IP에 직접 연결해 파드가 올바른 포트에 연결돼 있는지 확인합니다.
  - 파드 IP로 애플리케이션에 액세스할 수 없는 경우 애플리케이션이 로컬호스트에만 바인딩하고 있는지 확인합니다.

# Reference
- https://www.oreilly.com/library/view/kubernetes-in-action/9781617293726/
- https://www.oreilly.com/library/view/cloud-native-devops/9781492040750/
