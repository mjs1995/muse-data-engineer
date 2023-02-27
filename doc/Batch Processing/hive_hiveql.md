# HiveQL : 쿼리
- ![image](https://user-images.githubusercontent.com/47103479/217271415-eb16aeed-5f74-4893-8539-7f0c95b9d23e.png)
  - https://cwiki.apache.org/confluence/display/hive/design
- SELECT ... FROM 절
  - SQL에서 SELECT 프로젝션(projection) 연산자. FROM 절은 레코드를 선택하기 위해 필요한 테이블. 뷰 또는 중첩 쿼리(nested query)를 식별합니다.
  - 컬렉션 데이터형의 컬럼을 선택하면 하이브는 출력을 위해 JSON(Java Script Object Notation)문법을 사용합니다.
  - ARRAY 데이터형은 쉼표로 구분된 목록이 [...]로 둘려싸여 있습니다.
  - MAP의 경우 쉼표로 구분된 키:값 쌍의 목록을 {...}로 둘러싸는 JSON 표현을 사용합니다.
  - STRUCT로 JSON 맵 형식을 사용합니다.
 - 하이브는 오버 플로우나 언더플로우가 발생할 때 더 넓은 범위의 데이터형이 존재하더라도 결과를 자동으로 변환하지 않는 자바 데이터형 규칙을 따름니다. 
- 기타 내장 함수
  - > parse_url(url,partname,key) : HOST, PATH, QUERY, REF,PROTOCOL, AUTHORITY, FILE, USERINFO, QUERY:<key>. 옵션 키는 마지막에 QUERY:<key>를 요청함 
  - > find_in_set(s, 쉼표로 구분된 String) : 쉼표로 구분된 문자열에서 s의 색인을 반환함. 찾지 못하면 NULL이 반환됨
  - > locate(substr,str,pos) : str의 post 위치로부터 substr이 있는 색인을 반환함 
  - > instr(str,substr) : str에서 substr의 색인을 반환함
  - > str_to_map(s,delim1,delim2) : delim1을 키-값 쌍의 구분자로 사용하고 delim2를 키와 값의 구분자로 사용하여 문자열 s를 파싱한 후 맵을 생성함 
  - > sentences(s,lang,locale) : 문자열 s를 단어의 배열로 이루어진 문장의 배열로 반환함 
  - > ngrams(array<array<string>>, N, K, pf) : 텍스트에서 top-K n-gram을 반환함. pf는 정밀도
  - > context_ngrams(array<array<string>>,array<string>, int K, int pf> : ngrams와 같지만 출력 배열에서 두 번째 단어 배열로 시작하는 n-gram을 찾음
  - > in_file(s, filename) : filenmae 파일에서 문자열 s가 나타나면 true를 반환함 
- WHERE 절
  - WHERE 절에서 컬럼 별칭을 사용할 수 없습니다. 하지만 중첩 SELECT 문은 사용할 수 있습니다.
  - ```sql
    SELECT e.* FROM 
    (SELECT name, salary, deductions['Federal Taxes"] as ded,
    salary * (1 - deductions['Federal Taxes"]) as salary_minus_fed_taxes
    FROM employees) e
    WHERE round(e.salary_minus_fed_taxes) > 70000;
    ```
  - LIKE와 RLIKE
    - 하이브는 LIKE 절을 자바 정규표현식을 사용할 수 있는 RLIKE절로 확장합니다. 
    - 마침표(.)는 어떠 한 문자와 일치하고 별(*)은 왼쪽에 있는 것이 0번에서 여러 번 반복되는 것을 의미합니다.
    - (x|y) 표현식은 x 혹은 y가 일치하는 것을 의미합니다.
- 조인 문
  - 하이브는 고전적인 SQL 조인 문을 제공하며 동등 조인(EQUAL-JOIN)만 제공합니다.
  - 조인 최적화
    - 하이브는 쿼리의 마지막 테이블이 가장 크다고 가정합니다. 다른 테이블을 버퍼링하려고 시도하고 각 레코드에 대해서 조인을 수행하면서 마지막 테이블을 흘려보냅니다. 조인 쿼리를 구성할 때는 가장 큰 테이블이 가장 마지막에 오도록 합니다.
    - 하이브는 쿼리 최적화(optimizer)이기에 어떤 테이블을 마지막으로 흘려보내야 하는지 지정하는 힌트(hint) 메카니즘을 제공합니다.
      - ```sql
        SELECT /** STREAMTABLE(s) */ s.ymd, s.symbol, s.price_close, d.dividend
        FROM stocks s JOIN dividends d ON s.ymd = d.ymd AND s.symbol = d.symbol
        WHERE s.symbol = 'AAPL';
        ```
    - 중첩 SELECT 문
      - ```sql
        SELECT s.ymd, s.symbol, s.price_close, d.dividend FROM
        (SELECT * FROM stocks WHERE symbol = 'AAPL' AND exchange = 'NASDAQ') s
        LEFT OUTER JOIN
        (SELECT * FROM dividends WHERE symbol = 'AAPL' AND exchange = 'NASDAQ') d
        ON s.ymd = d.ymd;
        ```
    - 중첩 SELECT 문은 데이터 조인 전에 파티션 필터를 적용하는 데 필요한 푸시다운(push down)을 수행합니다.
      - 푸시다운은 WHERE 절의 술어 중 일부를 떼어내어 미리 실행하는 것을 말합니다. 리지주얼(residual)은 푸시다운 후에 남은 술어를 일컫습니다. 
      - 하이브는 조인을 수행한 후에 WHERE 절을 평가합니다. WHERE 절은 NULL이 되지 않는 컬럼값에 대해서만 필터를 적용하는 술어를 사용해야합니다. 하이브 문서와는 달리 파티션 필터는 외부 조인의 ON 절에서 동작하지 않습니다. 
  - 왼쪽 세미 조인
    - 왼쪽 세미 조인(LEFT SEMI-JOIN)은 오른쪽 테이블에서 ON의 술어를 만족하는 레코드를 찾을 경우 왼쪽 테이블의 레코드를 반환하며 하이브는 오른쪽 세미 조인을 지원하지 않습니다.
  - 맵 사이드 조인(Map-side Join)
    - 만약 한 테이블만 빼고 모두 작다면 작은 테이블은 메모리에 캐시하고 가장 큰 테이블은 맵퍼로 흘려보낼 수 있습니다. 하이브는 메모리에 캐시한 작은 테이블로부터 일치하는 모든 것을 찾아 낼 수 있기 때문에 맵에서 모든 조인을 할 수 있음. 이렇게 하면 일반 조인 시나리오에서 필요한 리듀스 단계를 제거할 수 있습니다.
    - ```sql
      SELECT /*+ MAPJOIN(d) */ s.ymd, s.symbol, s.price_close, d.dividend 
      FROM stocks s JOIN dividends d ON s.ymd = d.ymd AND s.symbol = d.symbol
      WHERE s.symbol = 'AAPL'
      ```
    - 하이브는 오른쪽 외부 조인과 완전 외부 조인에 대해서 최적화를 지원하지 않습니다.    
- ORDER BY와 SORT BY
  - ![image](https://user-images.githubusercontent.com/47103479/217562032-a7ddef9d-a1ac-4263-81b7-83005d3c30d5.png)
    - https://sqlrelease.com/sort-by-order-by-distribute-by-and-cluster-by-in-hive
  - 하이브는 ORDER BY 대신 데이터를 각 리듀서에 정렬하는 SORT BY를 추가합니다. 각 리듀스의 출력이 정렬되도록 지역 정렬(local ordering)을 수행하는 것을 의미합니다.
- SORT BY와 함께 사용하는 DISTRIBUTE BY
  - ![image](https://user-images.githubusercontent.com/47103479/217562172-11c79363-7475-4f64-b246-36cc92911e42.png)
    - https://sqlrelease.com/sort-by-order-by-distribute-by-and-cluster-by-in-hive
  - DISTRIBUTE BY는 맵의 출력을 리듀서로 어떻게 나누어 보내는지를 제어합니다. 
  - 기본적으로 맵리듀스는 맵퍼가 출력하는 키에 대해서 해시값을 계산하고 해시값을 이용하여 키-값 쌍을 가용한 리듀서로 균등하게 분산하려고 노력합니다. 
  - 맵퍼에서 출력한 키-값 쌍의 값을 계산하여 리듀서를 선택하는 것은 파티셔너의 역할 
  - SORT BY는 리듀서 안에서 데이터 정렬을 제어하는 반면 DISTRIBUTE BY는 리듀서가 처리할 로우를 어떻게 받는지 제어한다는 점에서 GROUP BY처럼 동작합니다.
  - 하이브는 SORT BY 절 전에 DISTRIBUTE BY 절을 사용할 것을 요구하므로 주의해야합니다.
  - SORT BY 절은 여러 리듀서를 사용하여 데이터를 정렬하는 반면 ORDER BY 절은 단일 리듀서를 사용하여 모든 데이터를 함께 정렬하기 때문에 대용량 데이터 세트를 정렬해야 할 때 ORDER BY 대신 SORT BY를 사용해야 합니다. 따라서 많은 수의 입력에 대해 ORDER BY를 사용하면 실행하는 데 많은 시간이 걸립니다.
- CLUSTER BY
  - ![image](https://user-images.githubusercontent.com/47103479/217562644-469150fb-a973-4a03-a208-242b0fe15774.png)
    - https://sqlrelease.com/sort-by-order-by-distribute-by-and-cluster-by-in-hive
  - DISTRIBUTE BY ... SORT BY 또는 간단히 사용하는 CLUSTER BY 절은 출력 파일 간에 전체 정렬을 이루면서 SORT BY의 병렬 처리를 사용하도록 하는 방법 
- 데이터 표본을 만드는 쿼리
  - 매우 큰 데이터셋에 대해서 전체를 사용하는 것이 아니라 어떤 쿼리를 수행하여 나온 결과를 대표 표본으로 하여 작업하고자 하는 경우가 종종 있습니다.
  - 하이브는 테이블을 버킷으로 구성하여 표본을 만든느 쿼리로 이를 지원합니다.
  - 버킷 테이블에 대한 입력 푸루닝
    - pruning(푸루닝)은 데이터 분석 작업에 불필요한 데이터를 미리 잘라내는 작업을 말합니다. 가지치기, 추리기 정도로 번역할 수 있으나 데이터베이스나 데이터 분석에 많이 사용하는 용어입니다. 

# Reference 
- https://www.amazon.com/Programming-Hive-Warehouse-Language-Hadoop/dp/1449319335
- https://www.amazon.com/Hadoop-Definitive-Guide-Tom-White/dp/1449311520
