# Docker
- <img width="535" alt="image" src="https://github.com/mjs1995/muse-data-engineer/assets/47103479/0e28f4e5-495f-4820-bd91-07646c430fd6">

  - https://www.docker.com/products/container-runtime/
- 리눅스의 응용 프로그램들을 프로세스 격리 기술들을 사용해 컨테이너로 실행하고 관리하는 오픈소스 프로젝트입니다.
- 도커 컨테이너는 일종의 소프트웨어를 소프트웨어의 실행에 필요한 모든 것을 포함하는 완전한 파일 시스템 안에 감쌉니다. 여기에는 코드, 런타임, 시스템 도구, 시스템 라이브러리 등 서버에 설치되는 무엇이든 아우릅니다. 이는 실행 중인 환경에 관계없이 언제나 동일하게 실행될 것을 보증합니다.
- 독립적인 컨테이너가 하나의 리눅스 인스턴스 안에서 실행할 수 있게 함으로써 가상 머신을 시작하여 유지보수해야 하는 부담을 없애줍니다.
- 도커 vs Virtual Machine(VM)
  - <img width="1174" alt="image" src="https://github.com/mjs1995/muse-data-engineer/assets/47103479/b9324ddb-7498-4422-a00e-e22e99016225">
    - https://www.docker.com/resources/what-container/
  - 기존의 VM은 새로운 가상환경을 추가할 때마다 새로운 Guest OS를 생성하여 독립된 가상환경을 만들었습니다.
  - 도커는 매번 Guest OS를 생성하지 않고 공유할 수 있는 자원은 공유한 뒤, 필요한 부분만 독립된 가상환경을 생성합니다.
  - 도커는 기존의 VM 방식 대비 훨씬 적은 리소스로 경량화된 형태로 독립된 가상환경을 생성할 수 있습니다.

# 도커의 기능
- <img width="661" alt="image" src="https://github.com/mjs1995/muse-data-engineer/assets/47103479/5b40c22e-d9f2-4467-80d5-47475aba0c24">

  - https://docs.docker.com/get-started/overview/#docker-architecture
- Build - 도커 이미지를 만드는 기술
  - 도커는 애플리케이션과 실행에 필요한 라이브러리, 미들웨어, OS, 네트워크 설정 등 필요한 모든 파일을 모아서 도커 이미지로 만듭니다.
  - 도커 이미지는 명령어를 이용해 수동으로 만들 수도 있지만 자동으로 빌드와 배포를 하는 CI/CD 환경에서는 도커 설정 파일(Dockerfile)을 이용해 자동으로 만들 수도 있습니다.
- Ship - 도커 이미지를 공유하는 기능
  - 도커 이미지는 도커 레지스트리에서 공유
    - 도커 레지스트리(Docker Registry) : 도커 이미지를 업로드해서 공유하는 저장소
    - 대표적으로는 도커의 공식 레지스트리인 Docker Hub
  - Ubuntu나 CentOS 같은 OS 이미지, MySQL, Redis, MongoDB, Nginx와 같은 미들웨어, OpenJDK, Golang, NodeJS와 같은 플랫폼 이미지도 제공합니다. 이런 베이스 이미지를 활용하면 환경을 빠르고 안전하게, 그리고 자동으로 구축할 수 있습니다.
  - Github와 같은 형상관리툴과 연동해서 Dockerfile을 관리하고 도커 이미지를 자동으로 빌드해서 도커 허브로 배포도 가능합니다.
- Run - 도커 컨테이너를 작동시키는 기능
  - 도커는 리눅스 상에서 컨테이너 단위로 서버 기능을 작동합니다. 또한 도커 이미지만 있으면 도커가 설치된 환경이라면 어디서든지 컨테이너를 작동시킬 수 있으며 도커 이미지를 가지고 여러개의 컨테이너를 가동시킬 수 있습니다.
  - 도커는 도커 이미지를 가지고 컨테이너를 생성해서 동작시키는데 하나의 이미지를 가지고 여러 개의 컨테이너를 만들어낼 수도 있습니다. 도커는 컨테이너를 생성하고 관리하기 위한 여러 명령을 제공합니다.
  - 실제 업무에서는 보통 한 대의 호스트에 모든 컨테이너를 동작시키는 것이 아니라 여러 호스트로 된 분산 환경인 경우가 많습니다. 이런 분산 환경에서 여러 노드의 컨테이너를 관리하기 위해 쿠버네티스(Kubernetes, k8s)와 같은 컨테이너 오케스트레이션 툴(Container Orchestration Tool)을 주로 사용
    - 오케스트레이션이란 컨테이너 배포, 장애 복구, 로드 밸런싱 등 여러 기능을 자동으로 처리해 주는 것을 말합니다.

# 도커 컴포넌트
- <img width="717" alt="image" src="https://github.com/mjs1995/muse-data-engineer/assets/47103479/1d2244f0-5613-4a05-b732-a7911102139c">

  - https://www.oreilly.com/library/view/native-docker-clustering/9781786469755/ch01.html
- Docker Engine(도커의 핵심기능) : 도커 이미지를 생성하고 컨테이너를 실행하는 핵심 기능.
- Docker Registry(이미지 공개 및 공유) : 컨테이너의 바탕이 되는 도커 이미지를 공개 및 공유하기 위한 레지스트리 기능. 도커 허브도 도커 레지스트리를 사용.
- Docker Compose(컨테이너 일원 관리) : 여러 개의 컨테이너 구성 정보를 코드로 정의하고, 명령을 실행함으로써 애플리케이션의 실행환경을 구성하는 컨테이너들을 일원 관리하기 위한 툴.
- Docker Machine(Docker 실행 환경 구축) : 로컬 호스트용인 VirtualBox를 비롯하여 Amazon Web Services EC2나 Microsoft Azure와 같은 클라우드 환경에 Docker의 실행 환경을 명령으로 자동 생성하기 위한 툴
- Docker Swarm(클러스터 관리) : 여러 Docker 호스트를 클러스터화하기 위한 툴로 클러스터를 관리하거나 API를 제공하는 역할은 Manager가, Docker 컨테이너를 실행하는 역할은 Node가 담당, 오픈소스인 Kubernetes도 이용할 수 있음

# 도커 컨테이너
- ![image](https://github.com/mjs1995/muse-data-engineer/assets/47103479/b1525035-d27c-41e5-b7cc-57d6232a47d1)
  - https://learn.microsoft.com/ko-kr/dotnet/architecture/microservices/container-docker-introduction/docker-containers-images-registries
- 도커 컨테이너
  - 이미지를 실행한 상태라고 볼 수 있고 추가되거나 변하는 값은 컨테이너에 저장됩니다.
  - 같은 이미지에서 여러 개의 컨테이너 생성 가능하며 컨테이너가 추가 및 변경되더라도 이미지는 변경되지 않고 그대로 남아있습니다.
  - 애플리케이션이 실행해야 하는 모든 것을 캡슐화하기 때문에 애플리케이션이 환경들 사이에 손쉽게 이동 가능합니다.
  - 마이크로서비스는 비즈니스 요구에 따라 별개의 팀에 의해 별개의 스케줄 상에서 LOB 앱(Line-of-Business App)의 부분들이 개별적으로 확장 수정 서비스되도록 할 수 있습니다.
  - 분리와 조절 기능 제공합니다.
  - 이식성을 제공하는 도커 컨테이너(컨테이너 런타임 환경을 지원하는 모든 장치에서 실행)
  - 결합성을 제공하는 도커 컨테이너(여러 컨테이너가 각 조각들을 제공하며 독립적으로 유지, 업데이트, 교체, 수정 가능)
  - 오케스트레이션과 스케일링이 쉬움(컨테이너는 가볍고, 오버헤드가 거의 없음)
- 도커 이미지
  - 이미지 : 컨테이너 실행에 필요한 파일과 설정값 등을 포함하고 있는 것으로 상태값을 가지지 않고 변하지 않습니다.(Immutable)
  - 컨테이너를 실행하기 위한 모든 정보를 가지고 있어서 더 이상 의존성 파일을 컴파일하고 설치할 필요 없습니다.
  - Doker hub에 등록하거나 Docker Registry 저장소를 직접 만들어 관리가 가능합니다.
  - Ubuntu 이미지 : ubuntu를 실행하기 위한 모든 파일을 가지고 있고 MySQL이미지는 debian을 기반으로 MySQL을 실행하는데 필요한 파일과 실행 명령어, 포트 정보 등을 가지고 있음
  - Gitlab 이미지 : centos를 기반으로 ruby, go, database, redis, gitlab source, nginx 드을 가지고 있음
- 도커 파일
  - ![image](https://github.com/mjs1995/muse-data-engineer/assets/47103479/825c5d9e-1244-4a12-ac43-0543ac0e9b26)
    - https://cto.ai/blog/docker-image-vs-container-vs-dockerfile/
  - 도커 이미지를 만들기 위한 설정 파일
  - 여러 가지 명령어를 토대로 Docker File을 작성하면 설정된 내용대로 도커 이미지를 만들 수 있습니다.
  - 도커는 이미지를 만들기 위해 Dockerfile이라는 이미지 빌드용 도메인 특화 언어(DSL - Domain Specific Language) 파일을 사용합니다.
  - 단순 텍스트 파일로 일반적으로 소스와 함께 관리합니다.
  - ![image](https://github.com/mjs1995/muse-data-engineer/assets/47103479/43abab06-46f6-4a4c-b31d-493b80812d7e)
    - https://extremeautomation.io/cheatsheets/docker-cheatsheet/
  - |명령어|설명|
    |:---:|:---:|
    |FROM|베이스 이미지 지정. 새로운 이미지를 만들 때, 이 이미지를 기반으로 생성
    |RUN|명령어 실행.
    |LABEL|라벨 설정. 주석과 비슷한 역할을 하며 이미지를 생성할 때, 정보 추가 가능
    |ENV|환경변수 설정
    |ADD|파일/디렉토리 추가
    |COPY|파일 복사. 파일/디렉토리를 이미지에 복사
    |VOLUME|볼륨 마운트. 호스트와 공유할 디렉토리를 지정. 컨테이너가 삭제되어도 데이터는 보존
    |EXPOSE|컨테이너에서 사용할 포트 지정
    |WORKDIR|명령어를 실행할 작업 디렉토리 설정
    |USER|명령어를 실행할 사용자 설정
    |ENTRYPOINT|데몬실행.ENTRYPOINT와 함께 사용할 경우에는 ENTRYPOINT가 CMD보다 우선시
    |CMD|데몬 실행. 컨테이너가 시작될 때 실행될 명령어를 설정

# 도커 명령어
```bash
docker run # 도커 이미지를 다운로드 받고, 컨테이너를 실행함

docker ps # 현재 실행되고 있는 컨테이너를 확인함

docker stop # 현재 실행되고 있는 컨테이너를 중지함

docker rm # 컨테이너를 제거함

docker logs # 컨테이너에 기록된 로그를 확인함

docker images # 현재 설치된 이미지 리스트를 출력함

docker pull # 이미지를 다운로드함

docker rmi # 현재 설치된 이미지를 제거함
```

# 도커를 이루는 기술
- ![image](https://github.com/mjs1995/muse-data-engineer/assets/47103479/56c68000-ecf1-4e56-b46d-39e418da88d4)
  - https://ko.wikipedia.org/wiki/%EB%8F%84%EC%BB%A4_%28%EC%86%8C%ED%94%84%ED%8A%B8%EC%9B%A8%EC%96%B4%29
- cgroup : 릴리스 관리 장치
  - 리눅스에서 프로그램은 프로세스로 실행되고, 프로세스는 하나 이상의 쓰레드로 이루어져 있습니다.
  - cgroups(Control Groups)는 프로세스와 쓰레드를 그룹화해서 관리하는 기술 
  - 호스트 OS의 자원을 그룹별로 할당하거나 제한을 둘 수 있습니다. 즉 컨테이너에서 사용하는 리소스를 제한함으로써 하나의 컨테이너가 자원을 모두 사용해 다른 컨테이너가 영향을 받지 않도록 할 수 있습니다.
  - 그룹에 계층 구조를 적용할 수 있어 체계적으로 리소스를 관리할 수 있습니다.
- namespace : 컨테이너를 구획화하는 장치
  - 컨테이너라는 가상의 독립된 환경을 만들기 위해 리눅스 커널의 namespace라는 기능을 사용합니다.
  - 리눅스 오브젝트에 이름표를 붙여 같은 이름표가 붙여진 것들만 묶어 관리합니다.
  - 도커는 컨테이너라는 독립된 환경을 만들고, 그 컨테이너를 구획화하여 애플리케이션의 실행 환경을 만듭니다.
  - |네임스페이스|설명|
    |:---:|:---:|
    |PID|리눅스에서 각 프로세스에 할당된 고유한 ID|
    |Network|네트워크 디바이스, IP주소, 포트 번호, 라우팅 테이블, 필터링 테이블 등과 같은 네트워크 리소스를 격리된 namespace마다 독립적으로 가질 수 있음|
    |UID|UID(사용자 ID), GID(그룹 ID)를 namespace별로 독립적으로 가질 수 있음|
    |Mount|리눅스에서 파일 시스템을 사용하기 위해서는 마운트가 필요|
    |UTS|namespace별로 호스트명이나 도메인명을 독자적으로 가질 수 있음|
    |IPC|프로세스 간의 통신(IPC) 오브젝트를 namespace별로 독립적으로 가질 수 있음

# Reference
- [도커 공홈 문서](https://docs.docker.com/get-started/overview/)
- [완벽한 IT인프라 구축을 위한 Docker](http://www.yes24.com/Product/Goods/64728692)
- [코드로 인프라 관리하기](https://www.oreilly.com/library/view/infrastructure-as-code/9781098114664/)
