# 스토어드 프로그램
- 스토어드 프로그램은 스토어드 루틴이라고도 하는데, 스토어드 프로시저와 스토어드 함수, 트리거와 이벤트 등을 모두 아우르는 명칭입니다.
- 스토어드 프로그램의 장점
  - 데이터베이스의 보안 향상
  - 기능의 추상화
  - 네트워크 소요 시간 절감
  - 절차적 기능 구현
  - 개발 업무의 구분 
- 스토어드 프로그램의 단점
  - 낮은 처리 성능
  - 애플리케이션 코드의 조각화
- 스토어드 프로그램의 문법
  - 정의부 : 스토어드 프로그램의 헤더 부분, 주로 스토어드 프로그램의 이름과 입출력 값을 명시하는 부분
  - 본문 부분 : 스토어드 프로그램의 바디(Body)라고도 하며, 스토어드 프로그램이 호출됐을 때 실행하는 내용을 작성하는 부분

## 스토어드 함수 
- 스토어드 함수
  - 스토어드 함수는 하나의 SQL 문장으로 작성이 불가능한 기능을 하나의 SQL 문장으로 구현해야 할 때 사용합니다.

## 트리거와 이벤트 
- 트리거
  - 트리거는 테이블의 레코드가 저장되거나 변경될 때 미리 정의해 둔 작업을 자동으로 실행해 주는 스토어드 프로그램으로 데이터의 변화가 생길 때 다른 작업을 기동해 주는 방아쇠인 것입니다.
  - 칼럼의 유효성 체크나 다른 테이블로의 복사나 백업, 계산된 결과를 다른 테이블에 함께 업데이트하는 등의 작업을 위해 트리거를 자주 사용합니다.
  - 트리거 생성
    - ```sql
      CREATE TRIGGER on_delete BEFORE DELETE ON employees
      FOR EACH ROW
      BEGIN
      DELETE FROM salaries WHERE emp_no=OLD.emp_no;
      END;;
      ```
- 이벤트
  - 주어진 특정한 시간에 스토어드 프로그램을 생성할 수 있는 스케줄러 기능 뜻합니다.
  - 이벤트 생성
    - 일회성 이벤트
      - 단 한 번 실행되는 일회성 이벤트를 등록하려면 ON SCHEDULE AT절을 명시하면 됩니다.
      - ```sql
        CREATE EVENT onetime_job
        ON SCHEDULE AT CURRENT_TIMESTAMP + INTERVAL 1 HOUR
        DO
        INSERT INTO daily_rank_log VALUES (NOW(), 'Done');
        ```
    - 반복성 이벤트
      - ```sql
        CREATE EVENT daily_ranking
        ON SCHEDULE EVERY 1 DAY STARTS '2020-09-07 01:00:00' ENDS '2021-01-01 00:00:00'
        DO
        INSERT INTO daily_rank_log VALUES (NOW(), 'DONE');
        ```
