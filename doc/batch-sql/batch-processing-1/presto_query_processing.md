# Presto Query Processing
- ![image](https://user-images.githubusercontent.com/47103479/216752571-aa89dabb-0da0-4937-b74c-1331bb11b66e.png)
- 쿼리 실행 계획은 다음과 같습니다.
  - ![image](https://user-images.githubusercontent.com/47103479/216752904-5a522e6d-692b-4860-827a-e3a8a644d0ad.png)
  - > explain analyze select * from open_data.highway_traffic where sumdate > 20220101 limit 10
  - Fragment structure : fragment는 프레스토의 단일 노드 혹은 여러 노드로 만들어진 일하는 단위
    - ![image](https://user-images.githubusercontent.com/47103479/216753488-042941d1-ed6b-43ff-80f8-fab15f79e618.png)
  - Distribution / Row layout
    - ![image](https://user-images.githubusercontent.com/47103479/216753600-7218dbaa-3e39-448f-ad0c-a4feea51c32c.png)
  - Performance stats
    - ![image](https://user-images.githubusercontent.com/47103479/216753655-11c306dc-fbfb-4aff-8f78-ea51a337f3af.png)
- EXPLAIN과 EXPLAIN ANALYZE 비교 
  - EXPLAIN: plan structure + cost estimates
  - EXPLAIN ANALYZE: plan structure + cost estimates + actual execution statistics
    - ![image](https://user-images.githubusercontent.com/47103479/216754574-b65d081f-dee8-4a53-a721-a18059ccd702.png)

## EXPLAIN 
- ![image](https://user-images.githubusercontent.com/47103479/216752661-2cd855c1-9a95-4330-a7fd-21d34603919e.png)
  - https://trino.io/Presto_SQL_on_Everything.pdf
- > EXPLAIN [ ( option [, ...] ) ] statement 
- 명령문의 논리적 또는 분산 실행 계획을 표시하거나 명령문의 유효성을 검사합니다 
- 프레스토의 쿼리 실행 형태
  - ANSI-SQL로 작성된 프레스토에서 실행 가능한 쿼리 문을 받습니다.
  - 프레스토 워커에서 실행할 수 있는 쿼리 형태로 내부적으로 변환합니다.
  - 실행 단계별로 나눕니다.
  - 각 단계마다 분할해서 수행 가능한 업무 단위로 할 일을 쪼갭니다. 각 일은 입력 데이터와 출력 데이터가 존재하는 블랙박스 형태로 만들어져서 fragment(프레스토의 단일 노드 혹은 여러 노드로 만들어진 일하는 단위. 각 부분/각각의 곳 정도로 나타냈다)에서 실행됩니다.
  - 부분 간에 필요시 데이터를 교환할 수 있는 연결고리를 구성합니다.
- > explain select * from open_data.highway_traffic where sumdate > 20220101 limit 10
  - ![image](https://user-images.githubusercontent.com/47103479/216756211-a9f4083e-043b-4160-b25b-4bf476f8df76.png)
- 우선 WHERE조건이 해당되는 테이블을 ScanFilter하는 것을 알 수 있습니다. 프레스토의 각 부분(단일 혹은 여러 노드)에서 일하는 유형은 보통 single 혹은 hash가 가장 많이 사용되며, 여기에서는 Single 유형으로 실행되었습니다.
  - SINGLE : Fragment는 단일 노드에서 실행됩니다.
  - HASH : 조각은 해시 함수를 사용하여 배포된 입력 데이터로 고정된 수의 노드에서 실행됩니다.
  - ROUND_ROBIN : Fragment는 라운드 로빈 방식으로 분산된 입력 데이터를 사용하여 고정된 수의 노드에서 실행됩니다.
  - BROADCAST : 프래그먼트는 고정된 수의 노드에서 실행되며 입력 데이터는 모든 노드에 브로드캐스트 됩니다.
  - SOURCE : 조각은 입력 분할에 액세스 하는 노드에서 실행됩니다.

## EXPLAIN ANALYZE
- > EXPLAIN ANALYZE [VERBOSE] statement
  - 명령문을 실행하고 각 작업의 비용과 함께 명령문의 분산 실행 계획을 표시합니다. VERBOSE옵션은 더 자세한 정보와 낮은 수준의 통계를 제공합니다
  - 각 계획 노드에 대해 몇 가지 추가 통계(예: 노드 인스턴스당 평균 입력, 관련 계획 노드에 대한 평균 해시 충돌 수)를 볼 수 있습니다. 이러한 통계는 쿼리에 대한 데이터 이상(편차, 비정상적인 해시 충돌)을 감지하려는 경우에 유용합니다.
  - LimitPartial은 limit 문구를 수행하기 위해 필터링을 수행한 워커에서 데이터 일부를 가져오는 작업입니다.
  - RemoteExchange, LocalExchange는 각 노드의 데이터를 모으는 작업입니다.
  - 모은 데이터에서 최종 limit을 실행하고 이를 output으로 내놓습니다.
- > EXPLAIN ANALYZE VERBOSE select * from open_data.highway_traffic where sumdate > 20220101 limit 10
  - Constant folding / Column pruning
    - ![image](https://user-images.githubusercontent.com/47103479/216754846-ee4c2d70-a735-4e73-817b-1fa9e7d4b705.png)
  - Limit pushdown / Partial limit pushdown
    - ![image](https://user-images.githubusercontent.com/47103479/216755146-7309a3f4-6683-49af-b4cb-742f962c01c8.png)
  - Skew
    - ![image](https://user-images.githubusercontent.com/47103479/216756398-57da2f1e-f6e5-4321-90ee-96a3b97949ad.png)

# Reference
- https://prestodb.io/docs/current/sql/explain.html
- https://techblog.woowahan.com/2556/
- https://trino.io/Presto_SQL_on_Everything.pdf
