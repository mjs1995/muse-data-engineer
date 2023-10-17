# terrafrom 'host' is not a valid URL
- 테라폼으로 airflow helm차트를 배포하던 중에 에러가 발생했습니다. 에러가 발생된 소스 코드입니다.
- ```shell
  provider "kubernetes" {
    host             = google_container_cluster.primary.endpoint
    token            = data.google_client_config.default.access_token
    cluster_ca_certificate = base64decode(
      google_container_cluster.primary.master_auth[0].cluster_ca_certificate
    )
  }
  ```
- 이는 프로바이더의 host 주소가 이상하다고 나온 에러였습니다.
- ```shell
  mun_js@cloudshell:~/terraform-airflow-gke (ggke-401900)$ terraform refresh
  data.google_client_config.default: Reading...

  Invalid attribute in provider configuration

    with provider["registry.terraform.io/hashicorp/kubernetes"],
    on airflow.tf line 7, in provider "kubernetes":
    7: provider "kubernetes" {

  'host' is not a valid URL
  ```
- 콘솔에서 직접 확인해 보니 https 부분이 빠진채로 반환하고 있습니다.
- ```shell
  mun_js@cloudshell:~/terraform-airflow-gke (ggke-401900)$ terraform console
  > google_container_cluster.primary.endpoint
  "3x.6x.xxx.xxx"
  ```
- 해당 주소가 먼저 GKE 클러스터의 외부 엔드포인트와 일치하는지 확인합니다.
  - ![image](https://github.com/mjs1995/muse-data-engineer/assets/47103479/d96cedda-4dc6-4a0c-968a-1ca58bf5e933)
- 관련하여 [깃허브 이슈](https://github.com/hashicorp/terraform-provider-kubernetes-alpha/issues/174)와 [테라폼 공식 문서](https://registry.terraform.io/providers/hashicorp/google/latest/docs/guides/using_gke_with_terraform#interacting-with-kubernetes)를 참고하여 해결했습니다.
- ```shell
  provider "kubernetes" {
    host             = "https://${google_container_cluster.primary.endpoint}"
    token            = data.google_client_config.default.access_token
    cluster_ca_certificate = base64decode(
      google_container_cluster.primary.master_auth[0].cluster_ca_certificate
    )
  }
  ```
