# spark 클러스터 매니저 
- ![image](https://user-images.githubusercontent.com/47103479/214573774-8e1c36bf-af5e-4118-a1b7-d26808c7cd9e.png)
-  클라우드에서 standalone cluster mode, 하둡에서 YARN, Apache Mesos, Kubernetes에서 스파크를 사용할 수 있습니다.
  - https://medium.com/@rachit1arora/why-run-spark-on-kubernetes-51c0ccb39c9b
## Standalone 
- ![image](https://user-images.githubusercontent.com/47103479/214574048-6c3d0257-76cc-4a2f-aefd-f46cc5cf684d.png)
  - https://www.researchgate.net/figure/Spark-Standalone-clustering-system_fig2_336020411
- 아파치 스파크 워크로드용으로 특별히 제작된 경량화 플랫폼으로 하나의 클러스터에서 다수의 스파크 애플리케이션을 실행할 수 있습니다.
- 실행을 위한 간단한 인터페이스를 제공하며 대형 스파크 워크로드로 확장할 수 있습니다.
- 스파크 애플리케이션만 실행할 수 있다는 큰 단점, 클러스터 환경을 빠르게 구축해 스파크 애플리케이션을 실행해야 하거나 YARN이나 메소스를 사용해본 경험이 없다면 가장 좋은 선택지입니다.
## YARN
- ![image](https://user-images.githubusercontent.com/47103479/214574233-bb141c8c-2fde-4bda-93a1-c115dd81c0ac.png)
  - https://medium.com/@goyalsaurabh66/running-spark-jobs-on-yarn-809163fc57e2
- 하둡 YARN은 잡 스케줄링과 클러스터 자원 관리용 프레임워크입니다.
- 스파크는 기본적으로 하둡 YARN 클러스터 매니저를 지원하지만 하둡 자체가 필요한 것은 아닙니다.
- 하둡 YARN은 다양한 실행 프레임워크를 지원하는 통합 스케줄러입니다.
- cluster 모드는 YARN 클러스터에서 스파크 드라이버 프로세스를 관리하며 클라이언트는 애플리케이션을 생성한 다음 즉시 종료됩니다. client 모드는 드라이버가 클라이언트 프로세스에서 실행됩니다.
- 하둡 설정
  - 스파크를 이용해 HDFS의 파일을 읽고 쓰려면 스파크 클래스패스에 두 개의 하둡 설정 파일을 포함시켜야합니다.
  - HDFS 클라이언트의 동작 방식을 결정하는 hdfs-site.xml 파일, 기본 파일 시스템의 이름을 설정하는 core-site.xml
  - /etc/hadoop/conf 하위에 설정 파일이 존재합니다.
## Mesos
- ![image](https://user-images.githubusercontent.com/47103479/214574374-9263db0a-c4e2-4fca-a4d7-733b834c44c9.png)
  - https://datastrophic.io/spark-jobserver-from-spark-standalone-to-mesos-marathon-and-docker-part-i/
- 아파치 메소스는 CPU, 메모리, 저장소 그리고 다른 연산 자원을 머신에서 추상화합니다.
- 내고장성(fault-tolerant) 및 탄력적 분산 시스템(elastic distributed system)을 쉽게 구성하고 효과적으로 실행합니다.
- 메소스는 스파크처럼 짧게 실행되는 애플리케이션을 관리합니다. 웹 애플리케이션이나 다른 자원 인터페이스 등 오래 실행되는 애플리케이션까지 관리할 수 있는 데이터센터 규모의 클러스터 매니저를 지향합니다.
- 메소스는 스파크에서 지원하는 클러스터 매니저 중 가장 무겁고 대규모의 메소스 배포 환경이 있는 경우에만 사용하는 것이 좋습니다.
## Kubernetes
- ![image](https://user-images.githubusercontent.com/47103479/214574523-fff5ab1e-2aef-4a65-b2a8-6acb3d7604eb.png)
  - https://spark.apache.org/docs/3.3.1/running-on-kubernetes.html#cluster-mode
- spark-submitKubernetes 클러스터에 Spark 애플리케이션을 제출하는 데 직접 사용할 수 있습니다. 
- 제출 메커니즘 작동 
  - Spark는 Kubernetes 포드 내에서 실행되는 Spark 드라이버를 생성합니다.
  - 드라이버는 Kubernetes 포드 내에서도 실행되는 실행기를 생성하고 연결하여 애플리케이션 코드를 실행합니다.
  - 애플리케이션이 완료되면 실행기 포드가 종료되고 정리되지만 드라이버 포드는 로그를 유지하고 결국 가비지 수집되거나 수동으로 정리될 때까지 Kubernetes API에서 "완료" 상태를 유지합니다.
- 완료된 상태에서 드라이버 포드는 계산 또는 메모리 리소스를 사용 하지 않습니다.

# Reference
- https://www.amazon.com/Spark-Definitive-Guide-Processing-Simple/dp/1491912219 
- https://www.oreilly.com/library/view/learning-spark/9781449359034/ch01.html
- https://spark.apache.org/docs/3.3.1/running-on-kubernetes.html#cluster-mode
