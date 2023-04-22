# Embulk
- [Embulk](https://github.com/embulk/embulk)에 대한 자세한 내용은 링크를 참고해주세요
  - https://github.com/mjs1995/muse-data-engineer/blob/main/doc/Data%20Ingestion/embulk.md 
- embulk를 설치합니다. 
- ```shell 
  # JRE 설치
  sudo apt install default-jre

  # embulk 최신버전 다운로드
  curl --create-dirs -o ~/.embulk/bin/embulk -L "https://dl.embulk.org/embulk-latest.jar"

  # embulk 실행 권한 추가 
  chmod +x ~/.embulk/bin/embulk

  # PATH 에 embulk 등록 
  echo 'export PATH="$HOME/.embulk/bin:$PATH"' >> ~/.bashrc

  # 변경된 PATH에 적용
  source ~/.bashrc

  # 확인
  embulk -version
  ```
  - ![image](https://user-images.githubusercontent.com/47103479/233781666-b98933ea-83b0-4742-907b-9785e24aa3c3.png)
  - [WARN] Unrecognized Java version: openjdk full version "17.0.6+10-Debian-1deb11u1", Unrecognized VM option 'AggressiveOpts', Error: Could not create the Java Virtual Machine. 해당 에러가 발생하였습니다. 
  - Embulk가 Java 버전을 인식하지 못하여 발생하는 문제로 Java 가상 머신(VM) 옵션 중 인식하지 못하는 것으로, Java 9 이후 버전에서는 삭제되었습니다. 따라서 Java 9 이후 버전을 사용하면 해당 오류가 발생할 수 있습니다.
    - ```shell
      # 기본 JRE 설치
      sudo apt install default-jre

      # Embulk가 인식할 수 있는 Java 버전 선택
      sudo update-alternatives --config java

      # 3. Embulk 실행 스크립트 파일에 JAVA_OPTS 환경 변수 설정하여 AggressiveOpts 옵션 비활성화
      echo 'export JAVA_OPTS="-XX:-UseAggressiveOpts"' >> ~/.bashrc
      source ~/.bashrc
      ```
    - ![image](https://user-images.githubusercontent.com/47103479/233782370-199e0e1a-facd-4d0c-9d07-1bd1bd57de56.png)
    - Embulk가 잘 설치된것을 확인할 수 있습니다.

## Embulk plug-in 설치 
- https://plugins.embulk.org/
- 기본적인 embulk에 plugin을 설치해서 postgres, bigquery, mysql, hdfs, oracle, redshift, s3, dynamodb, elasticsearch등등 다양한 저장소에서 데이터를 가져오고나 넣거나 할 수 있습니다
- ```shell 
  embulk gem list # 설치된 플러그인 리스트 
  embulk gem install embulk-input-mysql # MySQL
  embulk gem install embulk-input-postgresql # PostgreSQL
  embulk gem install embulk-output-bigquery # Bigquery
  ``` 
  - embulk gem install embulk-output-gcs를 실행할때 ERROR:  Error installing embulk-output-bigquery:jwt requires Ruby version >= 2.5. 에러가 발생하여서 해결했습니다.
    - ```shell
      # Java 8 설치하기
      sudo apt-get update
      sudo apt-get install openjdk-8-jre-headless -y
      
      # Embulk 설치하기
      curl --create-dirs -o ~/.embulk/bin/embulk -L "https://dl.embulk.org/embulk-latest.jar"
      chmod +x ~/.embulk/bin/embulk
      echo 'export PATH="$HOME/.embulk/bin:$PATH"' >> ~/.bashrc
      source ~/.bashrc
      
      # 필요한 플러그인 설치하기
      embulk gem install embulk-input-gcs
      embulk gem install embulk-output-gcs
      ```
    - ![image](https://user-images.githubusercontent.com/47103479/233783807-e29bc781-afb8-4fde-8fc0-e1fa218750e2.png)

## Embulk Example
- ```shell  
  embulk example ./test # 샘플데이터 생성 
  embulk guess ~/test/seed.yml -o config.yml # 입력 플러그인, 출력 플러그인 및 필드 매핑 구성을 포함하는 config.yml 파일을 추측하며 생성
  embulk preview config.yml # Embulk를 사용하여 config.yml 처리 결과를 미리보기
  embulk run config.yml # # 실행
  ```
  - embulk guess로 yaml 파일을 확인합니다.
    - ![image](https://user-images.githubusercontent.com/47103479/233784142-5b3c7f77-1880-4f72-9ecf-baf48b730c06.png)
  - ![image](https://user-images.githubusercontent.com/47103479/233784203-367675c9-36e6-424b-bf6c-34a33296e050.png)
  - ![image](https://user-images.githubusercontent.com/47103479/233784411-c717c959-d9f2-405e-b58b-0a499dc57522.png)

# Bigquery 설정 
- BigQuery Admin : 기존에 없는 테이블을 생성하기위한 권한으로 이하의 BigQuery Admin 권한을 갖게 되면, 기존의 테이블을 수정, 읽기, 쓰기는 가능하지만, 없는 테이블 생성에 대한 권한은 없습니다.
- Embulk 설정시 service_account_email을 넣어줘야합니다.
- 서비스 계정의 키를 넣어줍니다. 
  - ![image](https://user-images.githubusercontent.com/47103479/233784681-d4b9a5ba-ba2f-4df6-becc-e399d539f99a.png)
- gcs로 test할 yaml파일을 만든 뒤에 embulk run을 실행해줍니다.
  - ![image](https://user-images.githubusercontent.com/47103479/233785465-e4575295-2b45-4502-b682-d9f6400e1e5c.png)
- ```shell
  in:
    type: file
    path_prefix: /home/{유저}/./test/csv/sample_
    decoders:
    - {type: gzip}
    parser:
      charset: UTF-8
      newline: LF
      type: csv
      delimiter: ','
      quote: '"'
      escape: '"'
      null_string: 'NULL'
      trim_if_not_quoted: false
      skip_header_lines: 1
      allow_extra_columns: false
      allow_optional_columns: false
      columns:
      - {name: id, type: long}
      - {name: account, type: long}
      - {name: time, type: timestamp, format: '%Y-%m-%d %H:%M:%S'}
      - {name: purchase, type: timestamp, format: '%Y%m%d'}
      - {name: comment, type: string}
  out:
    type: bigquery
    mode: append
    auth_method: json_key
    json_keyfile: /home/{유저}/test/{프로젝트 키}.json
    service_account_email: zoom-de-service-acct@{프로젝트}.iam.gserviceaccount.com
    dataset: test_dataset
    auto_create_table: true
    table: embulk_table_%Y_%m
  ```
