# [Prefect](https://www.prefect.io/)
- ![image](https://user-images.githubusercontent.com/47103479/234580656-b493a9e6-60e6-44ee-88dc-b6669282b8f4.png)
  - https://docs.prefect.io/latest/
- Prefect는 Python 기반 워크플로 관리 시스템입니다. Prefect를 사용하면 로깅, 재시도, 동적 매핑, 캐싱, 실패 알림 등을 데이터 파이프라인에 쉽게 추가할 수 있습니다
- ![image](https://user-images.githubusercontent.com/47103479/234580785-d05395ca-12bc-4168-b263-b6483b165327.png)
  - https://discourse.prefect.io/t/what-are-the-components-of-prefect-2-0-architecture/909
- Prefect는 Dask 위에 구축되었으며 Dask를 사용하여 분산 환경에서 Prefect 워크플로의 실행을 예약하고 관리합니다.
- Prefect는 워크플로의 일정을 처리 하고 Dask는 각 워크플로 내 작업 의 일정 및 리소스 관리를 처리합니다.
  - 작업 예약: Dask는 워크플로우 내에서 모든 작업 예약을 처리하므로 Prefect는 Dask가 밀리초 대기 시간으로 예약하는 더 작은 작업을 장려할 수 있습니다.
  - Dataflow: Dask가 작업 간의 적절한 정보 직렬화 및 통신을 처리하기 때문에 Prefect는 "데이터 흐름"을 일급 패턴으로 지원할 수 있습니다.
  - 분산 계산: Dask는 클러스터의 작업자에게 작업 할당을 처리하여 사용자가 최소한의 오버헤드로 분산 계산의 이점을 즉시 실현할 수 있도록 합니다.
  - 병렬성: 클러스터에서 실행하든 로컬에서 실행하든 Dask는 선반에서 병렬 작업 실행을 제공합니다.

## Task
- 데이터 파이프라인에서 실제로 수행되는 작업 단위입니다. Task는 단일 작업을 수행하며, 다른 Task에 의존하는 경우가 많습니다.
- 입력 데이터를 받아서 출력 데이터를 생성할 수 있으며, 실행 가능한 코드로 구성됩니다
- | Argument | Description |
  | --- | --- |
  | `name` | Task의 이름입니다. 이름을 지정하지 않으면 함수의 이름이 사용됩니다. |
  | `description` | Task에 대한 설명입니다. docstring에서 가져옵니다. |
  | `tags` | Task에 대한 태그입니다. 실행시 prefect.context.tags에 추가됩니다. |
  | `cache_key_fn` | Task 결과를 캐싱할 때, 결과를 캐시하기 위한 키를 생성하는 함수입니다. 함수는 kwargs를 인자로 받아, 문자열을 반환합니다. |
  | `cache_expiration` | Task 결과 캐시의 만료 기간입니다. |
  | `task_run_name` | Task 실행시 실행 이름입니다. 키워드 인자들을 변수로 사용하여 이름을 생성합니다. |
  | `retries` | Task 실행 실패시 재시도 횟수입니다. |
  | `retry_delay_seconds` | Task 실행 실패시 재시도를 대기하는 시간입니다. |
  | `version` | Task의 버전 정보입니다. |
- ```python
  from prefect import flow, task

  @task(retries=3, cache_key_fn=task_input_hash, cache_expiration=timedelta(days=1))
  def fetch(dataset_url: str) -> pd.DataFrame:
      """Read taxi data from web into pandas DataFrame"""
      df = pd.read_csv(dataset_url)
      return df

  @flow()
  def etl_web_to_gcs()->None:
      """Main ETL function"""
      color = "yellow"
      year = 2021
      month = 1 
      dataset_file = f"{color}_tripdata_{year}-{month:02}"
      dataset_url = f"https://github.com/DataTalksClub/nyc-tlc-data/releases/download/{color}/{dataset_file}.csv.gz"

      df = fetch(dataset_url)

  if __name__ == '__main__':
      etl_web_to_gcs()
  ```

## Flow
- ![image](https://user-images.githubusercontent.com/47103479/230923548-791238b0-5a7d-42e6-8c7d-a169720834be.png)
- 데이터 파이프라인을 정의하고 실행하기위한 추상화된 개념입니다.
- Prefect Flow는 단일 태스크 또는 여러 태스크로 구성될 수 있으며, 이러한 태스크는 일련의 파이썬 함수로 작성됩니다.
- Flow는 다양한 데이터 파이프라인 패턴을 지원합니다. 
- | Argument | Description |
  | --- | --- |
  | `description` | Flow에 대한 문자열 설명을 입력합니다. 입력하지 않으면 데코레이트된 함수의 docstring에서 설명을 가져옵니다. |
  | `name` | Flow의 이름을 입력합니다. 입력하지 않으면 함수의 이름에서 추론됩니다. |
  | `retries` | Flow 실행 실패 시 재시도할 횟수를 지정합니다. 기본값은 0입니다. |
  | <span class="no-wrap">`retry_delay_seconds`</span> | Flow 실행 실패 후 재시도 전 대기할 시간을 지정합니다. retries가 0이 아닌 경우에만 적용됩니다. |
  | `flow_run_name` | Flow 실행 이름을 지정합니다. 이 이름은 Flow 매개변수를 변수로 사용하여 문자열 템플릿으로 제공할 수 있습니다. 함수를 반환하는 함수로 제공할 수도 있습니다. |
  | `task_runner` | Flow 내에서 task 실행에 사용할 task runner를 선택합니다. 지정하지 않으면 ConcurrentTaskRunner가 사용됩니다. |
  | `timeout_seconds` | Flow 최대 실행 시간을 초 단위로 지정합니다. 지정된 시간이 초과되면 Flow가 실패로 표시됩니다. Flow 실행은 다음 task가 호출될 때까지 계속됩니다. |
  | `validate_parameters` | Flow 매개변수가 Pydantic을 통해 검증되는지 여부를 지정합니다. 기본값은 True입니다. |
  | `version` | Flow 버전을 지정합니다. 지정하지 않으면 래핑된 함수가 포함된 파일의 해시 값을 이용하여 버전을 생성하려 시도합니다. 파일을 찾을 수 없는 경우 버전은 null이 됩니다. |

## Blocks
- 블록은 구성 저장을 활성화하고 외부 시스템과 상호 작용하기 위한 인터페이스를 제공하는 Prefect 내의 기본 요소입니다.
- 블록을 사용하면 AWS, GitHub, Slack 및 Prefect로 오케스트레이션하려는 기타 시스템과 같은 서비스로 인증하기 위한 자격 증명을 안전하게 저장할 수 있습니다.
- prefect 내부에서 Block를 눌러 다양한 커넥터를 확인합니다.
- ![image](https://user-images.githubusercontent.com/47103479/230776923-8cf1aaf3-d387-4ab8-b00d-037d45451b70.png)
- Google Cloud Platform 용 Prefect 커넥터를 등록합니다.
  - > prefect block register -m prefect_gcp
  - <img width="859" alt="image" src="https://user-images.githubusercontent.com/47103479/230776975-d6f69474-2c83-4678-ab2c-c43df569ee39.png">
- ```python
  from prefect import flow, task
  from prefect_gcp.cloud_storage import GcsBucket

  @task()
  def write_gcs(path: Path) -> None:
      """Upload local parquet file to GCS"""
      gcs_block = GcsBucket.load("zoom-gcs")
      gcs_block.upload_from_path(from_path=path, to_path=path)
      return
  ```
- Prefect built-in blocks
  - | Block | Slug | Description |
    | --- | --- | --- |
    | Azure | `azure` | Azure Datalake 및 Azure Blob Storage에서 파일로 데이터를 저장하는 블록입니다. |
    | Date Time | `date-time` | 날짜 및 시간을 나타내는 블록입니다. |
    | Docker Container] | `docker-container` | 컨테이너에서 명령을 실행하는 블록입니다. |
    | Docker Registry | `docker-registry` | Docker 레지스트리에 연결하는 블록으로 Docker Engine에 연결할 수 있어야 합니다. |
    | GCS | `gcs` | Google Cloud Storage에 파일로 데이터를 저장하는 블록입니다. |
    | GitHub | `github` | 공개 GitHub 리포지토리에 저장된 파일과 상호 작용하는 블록입니다. |
    | JSON | `json` | JSON 데이터를 나타내는 블록입니다. |
    | Kubernetes Cluster Config | <span class="no-wrap">`kubernetes-cluster-config`</span> | Kubernetes 클러스터와 상호 작용하기 위한 구성을 저장하는 블록입니다. |
    | Kubernetes Job | `kubernetes-job` | Kubernetes Job으로 명령을 실행하는 블록입니다. |
    | Local File System | `local-file-system` | 로컬 파일 시스템에 파일로 데이터를 저장하는 블록입니다 |
    | Microsoft Teams Webhook | `ms-teams-webhook` | 제공된 Microsoft Teams 웹훅을 사용하여 알림을 보내는 블록입니다. |
    | Opsgenie Webhook | `opsgenie-webhook` | 제공된 Opsgenie 웹훅을 사용하여 알림을 보내는 블록입니다. |
    | Pager Duty Webhook | `pager-duty-webhook` | 제공된 PagerDuty 웹훅을 사용하여 알림을 보내는 블록입니다. |
    | Process | `process` | 새 프로세스에서 명령을 실행하는 블록입니다. |
    | Remote File System | `remote-file-system` | 원격 파일 시스템에 파일로 데이터를 저장하는 블록으로 fsspec에서 지원하는 모든 원격 파일 시스템을 지원합니다..  |
    | S3 | `s3` | AWS S3에 파일로 데이터를 저장하는 블록입니다. |
    | Secret | `secret` | 비밀 값으로서 로그되거나 UI에 표시될 때 가려지는 블록입니다. |
    | Slack Webhook | `slack-webhook` | 제공된 Slack 웹훅을 사용하여 알림을 보내는 블록입니다. |
    | SMB | `smb` | SMB 공유에 파일로 데이터를 저장하는 블록입니다. |
    | String | `string` | 문자열 데이터를 나타내는 블록입니다. |
    | Twilio SMS | `twilio-sms` | Twilio SMS를 통해 알림을 보내는 블록입니다. |
    | Webhook | `webhook` | 웹훅을 호출하는 블록입니다. |

## Deployment
- ![image](https://user-images.githubusercontent.com/47103479/231788166-baf1a7a7-4757-4ace-a368-758c3361a611.png)
- deployment는 스트림을 캡슐화하고 API를 통해 일정을 예약하거나 시작할 수 있는 서버 측 아티팩트입니다. flow는 여러 Deployment에 속할 수 있으며 프로그래밍하는 데 필요한 모든 것이 포함된 메타데이터가 있는 컨테이너라고 말할 수 있습니다. 명령줄이나 Python으로 만들 수 있습니다.
- Prefect 워크플로에 대한 배포 생성은 Prefect API를 통해 워크플로를 관리하고 Prefect 에이전트에서 원격으로 실행할 수 있도록 워크플로 코드, 설정 및 인프라 구성을 패키징하는 것을 의미합니다.
- Prefect CLI 또는 UI를 사용하여 생성, 업데이트, 중지 및 삭제할 수 있습니다. Deployment에 대한 상태 및 로그 정보는 Prefect Cloud 또는 Prefect Server UI에서 볼 수 있으며, 원하는 경우 다운로드하여 검사할 수 있습니다. 
- Deployment를 사용하면 Flow를 쉽게 실행하고 관리할 수 있으며, 코드 변경이나 업데이트를 쉽게 반영할 수 있습니다.
- > prefect deployment build parameterized_flows.py:etl_parent_flow -n "Parameterized ETL"
  - <img width="882" alt="image" src="https://user-images.githubusercontent.com/47103479/231798903-101eb987-73f8-4de9-a0eb-994c785e420a.png">
- 배포 결과로 yaml 파일이 생성되었습니다.
  - <img width="644" alt="image" src="https://user-images.githubusercontent.com/47103479/231799216-39365c93-9770-4215-803f-61c85d81ed5f.png">
- ![image](https://user-images.githubusercontent.com/47103479/231800462-a5c7e2f2-d0a7-433b-a4d6-71f3a556cff8.png)
- ![image](https://user-images.githubusercontent.com/47103479/232055239-43e0041d-4463-4ede-ba31-615c2faff673.png)
- ![image](https://user-images.githubusercontent.com/47103479/232056175-6ec51b5e-a38a-4f34-a226-f6206e642505.png)


# Refereence
- https://docs.prefect.io/latest/
- https://examples.dask.org/applications/prefect-etl.html
