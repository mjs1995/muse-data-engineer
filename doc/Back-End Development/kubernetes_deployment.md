# 디플로이먼트: 선언적 애플리케이션 업데이트 
- 쿠버네티스에서는 일반적으로 Replication Controller(RC)를 이용해서 배포하지 않고 Deployment라는 개념을 사용합니다. 복제된(replicated) 애플리케이션을 관리하는 API 객체입니다.
- ReplicatSet(+Pod)을 생성합니다. 롤릴 업데이트 등을 할 때 RC를 두 개를 만들어야 하고 하나씩 Pod의 수를 수동으로 조정해야 하기 때문에 이를 자동화 해서 추상화한 개념이 Deployment입니다.
- 기본적으로 Replication Controleer (RC)를 생성하고 이를 관리하는 역할입니다.
- 특징
  - 쿠버네티스가 애플리케이션의 인스턴스를 어떻게 생성하고 업데이트해야 하는지를 지시합니다.
  - 디플로이먼트가 만들어지면, 쿠버네티스 마스터가 해당 애플리케이션 인스턴스를 클러스터의 개별 노드에 스케줄합니다.
  - 애플리케이션 인스턴스가 생성되면, 쿠버네티스 디플로이먼트 컨트롤러는 지속적으로 이들 인스턴스를 모니터링합니다.
  - 머신의 장애나 정비에 대응할 수 있는 자동 복구(self-healing) 메커니즘을 제공합니다.
  - 디플로이먼트는 애플리케이션 인스턴스를 생성하고 업데이트 하는 역할을 담당합니다.
- Kubectl이라는 쿠버네티스 CLI를 통해 디플로이먼트를 생성하고 관리하며 Kubectl은 클러스터와 상호 작용하기 위해 쿠버네티스 API를 사용합니다.
- 디플로이먼트를 생성할 때, 애플리케이션에 대한 컨테이너 이미지와 구동시키고자 하는 복제 수를 지정합니다.
- 애플리케이션이 쿠버네티스 상에 배포되려면 지원되는 컨테이너 형식 중 하나로 패키지 되어야합니다.
- 파드에서 실행 중인 애플리케이션 업데이트 
  - 모든 파드를 업데이트 하는 방법
    - 기존 파드를 모두 삭제한 다음 새 파드를 시작합니다. (짧은 시간 동안 애플리케이션을 사용할 수 없습니다.)
    - 새로운 파드를 시작하고, 기동하면 기존 파드를 삭제합니다. 새 파드를 모두 추가한 다음 한꺼번에 기존 파드를 삭제하거나 순차적으로 새 파드를 추가하고 기존 파드를 점진적으로 제거해 이 작업을 수행할 수 있습니다.
      - ![image](https://user-images.githubusercontent.com/47103479/231029064-04bb32b1-691b-484f-bd28-86612acaac54.png)  
  - 새 파드 기동과 이전 파드 삭제
    - 한 번에 이전 버전에서 새 버전으로 전환 
      - ![image](https://user-images.githubusercontent.com/47103479/231029290-ce368f60-d5a2-4b87-a1e6-cf5aca40bd27.png)
      - 파드의 앞쪽에는 일반적으로 서비스를 배치하며 새 버전을 실행하는 파드를 불러오는 동안 서비스는 파드의 이전 버전에 연결됩니다. 
      - 블루 그린 디플로이먼트 : 새 파드가 모두 실행되면 서비스의 레이블 셀렉터를 변경하고 서비스를 새 파드로 전환할 수 있습니다. 전환한 후 새 버전이 올바르게 작동하면 이전 레플리케이션컨트롤러를 삭제해 이전 파드를 삭제할 수 있습니다.
    - 롤링 업데이트
      - ![image](https://user-images.githubusercontent.com/47103479/231030180-ad65e1cc-f362-444e-ac30-df8c20d7596e.png)
      - 새 파드가 모두 실행된 후 이전 파드를 한 번에 삭제하는 방법 대신 파드를 단계별로 교체하는 롤링 업데이트. 이전 레플리케이션컨트롤러를 천천히 스케일 다운하고 새 파드를 스케일 업해 이를 수행할 수 있습니다.

## 애플리케이션을 선언적으로 업데이트하기 위한 디플로이먼트 사용하기 
- 디플로이먼트는 낮은 수준(lower-level)의 개념으로 간주되는 레플리케이션컨트롤러 또는 레플리카셋을 통해 수행하는 대신 애플리케이션을 배포하고 선언적(declarative) 으로 업데이트하기 위한 높은 수준(high-level)의 리소스입니다. 
- 디플로이먼트를 생성하면 레플리카셋 리소스가 그 아래에 생성되어 결과적으로 더 많은 리소스가 생성됩니다.레플리카셋도 파드를 복제하고 관리하며 디플로이먼트를 사용하는 경우 실제 파드는 디플로이먼트가 아닌 디플로이먼트의 레플리카셋에 의해 생성되고 관리됩니다.
- 파드를 감시하는 레플리카셋이 디플로이먼트를 지원합니다.
  - ![image](https://user-images.githubusercontent.com/47103479/231030417-ff11b5b2-972d-4ce7-848d-a59d379dc8a2.png)
- 디플로이먼트 생성 
  - ```yaml
    apiVersion: apps/v1beta1
    kind: Deployment
    metadata:
      name: kubia
    spec:
      replicas: 3
      template:
        metadata:
          name: kubia
          labels:
            app: kubia
        spec:
          containers:
          - image: luksa/kubia:v1
            name: nodejs
    ```
  - 디플로이먼트는 레이블 셀렉터, 원하는 레플리카 수, 파드 템플릿으로 구성됩니다. 디플로이먼트 리소스가 수정될 때 업데이트 수행 방법을 정의하는 디플로이먼트 전략을 지정하는 필드도 있습니다. 
- 디플로이먼트 롤아웃 상태 출력
  - 디플로이먼트 상태를 확인하기 위해 특별히 만들어진 다른 명령어 
  - ```bash
    $ kubectl rollout status deployment kubia
    Waiting for rollout to finish: 1 out of 3 new replicas have been updated...
    Waiting for rollout to finish: 2 out of 3 new replicas have been updated...
    Waiting for rollout to finish: 1 old replicas are pending termination...
    deployment "kubia" successfully rolled out
    ```
- 디플로이먼트 업데이트 
  - 사용 가능한 디플로이먼트 전략 
    - Recreate 전략
      - 새 파드를 만들기 전에 이전 파드를 모두 삭제합니다. 애플리케이션이 여러 버전을 병렬로 실행하는 것을 지원하지 않고 새 버전을 시작하기 전에 이전 버전을 완전히 중지해야 하는 경우 이 전략을 사용합니다. 
      - 애플리케이션을 완전히 사용할 수 없는 짧은 서비스 다운타임이 발생합니다. 
    - RollingUpdate 전략
      - 기본 전략이며 이전 파드를 하나씩 제거하고 동시에 새 파드를 추가해 전체 프로세스에서 애플리케이션을 계속 사용할 수 있도록 하고 서비스 다운 타임이 없도록 합니다.
      - 의도하는 레플리카 수보다 많거나 적은 파드 수에 관한 상한과 하한을 설정할 수 있습니다. 애플리케이션에서 이전 버전과 새 버전을 동시에 실행할 수 있는 경우에만 이 전략을 사용해야 합니다. 
- 쿠버네티스의 기존 리소스 수정하기
  - kubectl edit : 기본 편집기로 오브젝트의 매니페스트를 오픈합니다. 변경 후 파일을 저장하고 편집기를 종료하면 오브젝트가 업데이트됨. kubectl edit deployment kubia
  - kubectl patch : 오브젝트의 개별 속성을 수정합니다. kubectl patch deployment kubia -p '{"spec": {"template": {"spec": {"containers": [{"name": "nodejs", "image": "luksa/kubia:v2" }]}}}}'
  - kubectl apply : 전체 YAML/JSON 파일의 속성 값을 적용해 오브젝트를 수정합니다. YAML/JSON에 지정된 오브젝트가 아직 없으면 생성되며 파일에는 리소스의 전체 정의가 포함돼야 합니다. kubectl apply -f kubia-deployment-v2.yaml 
  - kubectl replace : YAML/JSON 파일로 오브젝트를 새 것으로 교체합니다. apply 명령어와 달리 이 명령은 오브젝트가 있어야 하며 그렇지 않으면 오류를 출력합니다. kubectl replace -f kubia-deployment-v2.yaml 
  - kubectl set image : 파드, 레플리케이션컨트롤러 템플릿, 디플로이먼트, 데몬셋, 잡 또는 레플리카셋에 정의된 컨테이너 이미지를 변경합니다. kubectl set image deployment kubia nodejs=luksa/kubia:v2 
  - 이 명령들이 하는 역할은 디플로이먼트 스펙을 변경하는 것으로 이 변경으로 롤아웃 프로세스가 시작됩니다. 
- 디플로이먼트 롤백 
  - 디플로이먼트 롤아웃 이력 표시 
    - 디플로이먼트는 개정 이력(revision history)을 유지하므로 롤아웃의 롤백이 가능함. 롤아웃이 완료되면 이전 레플리카셋은 삭제되지 않으므로 이전 버전뿐만 아니라 모든 버전으로 롤백할 수 있음. kubectl rollout history 명령어로 개정 이력을 표시할 수 있습니다.
    - ```shell
      $ kubectl rollout history deployment kubia
      deployments "kubia":
      REVISION    CHANGE-CAUSE
      2           kubectl set image deployment kubia nodejs=luksa/kubia:v2
      3           kubectl set image deployment kubia nodejs=luksa/kubia:v3
      ```
  - 특정 디플로이먼트 개정으로 롤백
    - undo 명령어에서 개정(revison) 번호를 지정해 특정 개정으로 롤백할 수 있습니다.
    - ```shell
      # 첫 번째 버전으로 롤백
      $ kubectl rollout undo deployment kubia --to-revision=1 
      ```
  - 롤아웃 일시 정지
    - 카나리 릴리스는 잘못된 버전의 애플리케이션이 롤아웃돼 모든 사용자에게 영향을 주는 위험을 최소화하는 기술로 새 버전을 모든 사람에게 롤아웃하는 대신 하나 또는 적은 수의 이전 파드만 새 버전으로 바꿉니다. 이렇게 하면 소수의 사용자만 초기에 새 버전을 사용하게 됩니다.

# Reference
- https://www.oreilly.com/library/view/kubernetes-in-action/9781617293726/
- https://www.oreilly.com/library/view/cloud-native-devops/9781492040750/
