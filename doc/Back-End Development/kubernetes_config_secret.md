# 컨피그맵과 시크릿: 애플리케이션 설정 
- 쿠버네티스에서는 설정 옵션을 컨피그맵이라 부르는 별도 오브젝트로 분리할 수 있습니다. 컨피그맵은 짧은 문자열에서 전체 설정 파일에 이르는 값을 가지는 키/값 쌍으로 구성된 맵입니다.
- 컨테이너에서 필요한 환경설정 내용을 컨테이너와 분리해서 제공해 주기 위한 기능 : 클라우드 네이티브 아키텍처에서 컨테이너는 변하지 않는 자원
- 비기밀 데이터를 키-값 쌍으로 저장하기 위해 사용하는 API 객체
  - ![image](https://user-images.githubusercontent.com/47103479/230868764-80ceb9c7-2f2c-4544-8dd6-5a49964efa00.png)
- 컨테이너 이미지에서 설정 데이터를 분리(decouple)시키기 위한 것 
  - ![image](https://user-images.githubusercontent.com/47103479/230868890-2f3d9df7-07dc-465a-8d9c-4d659eef168b.png)
- 컨테이너 이미지에서 사용하는 환경변수와 같은 세부 정보를 분리하고, 그 환경변수에 대한 값을 외부로 노출 시키지 않고 내부에 존재하는 스토리지에 저장해서 사용하는 방법
- 컨피그맵을 사용하면 컨테이너 이미지에서 해당 환경에 국한된 설정을 분리 가능 : 애플리케이션을 어디로든 쉽게 이전 가능(portable)
- ConfigMap을 변경하더라도 Running 상태의 Pod에 곧바로 적용되지 않으며 Pod을 재기동 해야 합니다.
- 컨테이너화된 애플리케이션 설정
  - 설정 데이터를 저장하는 쿠버네티스 리소스를 컨피그맵(ConfigMap)이라 합니다. 컨피그맵을 사용해 설정 데이터를 저장할지 여부에 관계없이 다음 방법을 통해 애플리케이션을 구성할 수 있습니다.
    - 컨테이너에 명령줄 인수 전달
      - 명령줄인수 (command-line arguemnt) - 쿠버네티스는 configmap 값을 기반으로 컨테이너에 대한 명령줄을 동적으로 생성하도록 지원합니다. 
    - 각 컨테이너를 위한 사용자 정의 환경변수 지정
      - 환경변수 (environment variable) - configmap 을 사용해 환경변수의 값을 동적으로 설정합니다.
    - 특수한 유형의 볼륨을 통해 설정 파일을 컨테이너에 마운트 
      - 파일 시스템(filesystem) - pod에 configmap을 마운트할 수 있습니다. 키 이름에 따라 각 항목의 파일이 생성되며 해당 파일의 내용은 값으로 설정됩니다.
- 컨테이너에 명령줄 인자 전달 
  - 도커에서 명령어와 인자 정의 
    - Dockerfile에서 두 개의 지침
      - ENTRYPOINT는 컨테이너가 시작될 때 호출될 명령어를 정의함
      - CMD는 ENTRYPOINT에 전달되는 인자를 정의함 
      - ENTRYPOINT 명령어로 실행하고 기본 인자를 정의하려는 경우에만 CMD를 지정하는 것 
- 컨피그맵 생성
  - ```bash    
    $ kubectl create configmap my-config
       --from-file=foo.json
       --from-file=bar=foobar.conf
       --from-file=config-opts/
       --from-literal=some=thing
    ```
  - 개별 파일, 디렉터리 및 리터럴 값에서 ConfigMap 만들기
    - ![image](https://user-images.githubusercontent.com/47103479/231027144-bfd7e0a1-074a-4cba-9771-42b2be517b93.png)
- 시크릿으로 민감한 데이터를 컨테이너에 전달
  - 본질적으로 민감한 데이터는 시크릿을 사용해 키 아래에 보관하는 것이 필요합니다. 만약 설정 파일이 민감한 데이터와 그렇지 않은 데이터를 모두 가지고 있다면 해당 파일을 시크릿 안에 저장해야 합니다. 민감하지 않고, 일반 설정 데이터는 컨피그맵을 사용합니다.
  - ![image](https://user-images.githubusercontent.com/47103479/231027313-5b8bec74-51f6-4f15-ba96-e394b5a27df9.png)
  - 컨피그맵과 시크릿 비교
    - 시크릿 항목의 내용은 Base64 인코딩 문자열로 표시되고, 컨피그맵의 내용은 일반 텍스트로 표시됩니다. 처음에는 각 항목을 설정하고 읽을 때마다 인코딩과 디코딩을 해야 하기 때문 시크릿 안에 있는 YAML과 JSON 매니페스트를 다루는 것이 고통스럽습니다.
    - 바이너리 데이터 시크릿 사용
      - Base64 인코딩을 사용하는 까닭은 시크릿 항목에 일반 텍스트뿐만 아니라 바이너리 값도 담을 수 있기 때문입니다. Base64 인코딩은 바이너리 데이터를 일반 텍스트 형식인 YAML이나 JSON 안에 넣을 수 있습니다.

# Reference
- https://www.oreilly.com/library/view/kubernetes-in-action/9781617293726/
- https://www.oreilly.com/library/view/cloud-native-devops/9781492040750/
