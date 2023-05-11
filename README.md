# muse-data-engineer

* Technical Skills to becoming a data engineer

## Data Ingestion

* Kafka
  * [이진로그 Binary log(Binlog)](data-ingestion/data-ingestion/binlog.md)
  * [변경 데이터 캡처 CDC](data-ingestion/data-ingestion/cdc.md)
* Embulk
  * [Embulk & Digdag](data-ingestion/data-ingestion-1/embulk.md)
  * [Embulk 코드](data-ingestion/data-ingestion-1/embulk\_code.md)

## Batch Processing

* Hadoop
  * [Hadoop과 HDFS](doc/batch-processing/hadoop\_hdfs.md)
  * [Mapreduce와 YARN](doc/batch-processing/hadoop\_map\_yarn.md)
  * [Hadoop ECO System](https://github.com/mjs1995/muse-data-engineer/blob/main/doc/Batch%20Processing/hadoop_eco.md)
* Spark
  * [Spark 개요](doc/batch-processing-1/spark\_base.md)
  * [Spark 튜닝](doc/batch-processing-1/spark\_tuning.md)
  * [Spark 최적화](doc/batch-processing-1/spark\_optimization.md)
  * [Spark Yarn](doc/batch-processing-1/spark\_yarn.md)
  * [Spark 클러스터 매니저](doc/batch-processing-1/spark\_cluster\_manager.md)
  * [Spark 조인과 셔플](doc/batch-processing-1/spark\_join.md)
* Batch SQL
  * Presto
    * [Presto 개요](doc/batch-sql/batch-processing-1/presto\_base.md)
    * [Presto 튜닝](doc/batch-sql/batch-processing-1/presto\_tuning.md)
    * [Presto 쿼리 Processing](doc/batch-sql/batch-processing-1/presto\_query\_processing.md)
    * [Trino 개요](doc/batch-sql/batch-processing-1/trino\_base.md)
  * Hive
    * [Hive 개요](doc/batch-sql/batch-processing/hive\_base.md)
    * [Hive 아키텍처](doc/batch-sql/batch-processing/hive\_architecture.md)
    * [Hive 포맷](doc/batch-sql/batch-processing/hive\_format.md)
    * [HiveQL](doc/batch-sql/batch-processing/hive\_hiveql.md)

## workflow

* Airflow
  * [Airflow 개요](workflow/airflow/airflow\_base.md)
  * [Airflow 아키텍처](workflow/airflow/airflow\_architecture.md)
* Dbt
  * [dbt 개요](workflow/dbt\_base.md)
* Prefect
  * [Prefect 개요](workflow/prefect\_base.md)

## BI

* [OLAP vs OLTP](bi/olap.md)
* [데이터 모델링과 DW/DM](bi/data\_modeling\_dw\_dm.md)
* [데이터 레이크와 클라우드 DW](bi/data\_lake.md)

## Back-End Development

* [인프라 기초](back-end-development/infra\_based.md)
* [클라우드와 온프레미스](back-end-development/onpremises\_cloud.md)
* [모놀리틱 아키텍처와 마이크로서비스 아키텍처](back-end-development/msa.md)
* Kubernetes
  * [Kubernetes 개요](back-end-development/kubernetes/kubernetes\_base.md)
  * [Kubernetes 오브젝트 모델](back-end-development/kubernetes/kubernetes\_object.md)
  * [Kubernetes 파드](back-end-development/kubernetes/kubernetes\_pod.md)
  * [Kubernetes 레플리케이션](back-end-development/kubernetes/kubernetes\_replica.md)
  * [Kubernetes 서비스](back-end-development/kubernetes/kubernetes\_service.md)
  * [Kubernetes 볼륨](back-end-development/kubernetes/kubernetes\_volume.md)
  * [Kubernetes 컨피그맵과 시크릿](back-end-development/kubernetes/kubernetes\_config\_secret.md)
  * [Kubernetes 디플로이먼트](back-end-development/kubernetes/kubernetes\_deployment.md)
* Docker
  * [Docker 개요](back-end-development/docker/docker\_base.md)

## Programming Language

* Python
  * [Python과 프로파일링](programming-language/programming-language/python\_profiling.md)
  * [Python 컴파일](programming-language/programming-language/python\_comfile.md)
  * [Python 비동기](programming-language/programming-language/python\_async.md)
  * [Python multiprocessing](programming-language/programming-language/python\_multiprocessing.md)
