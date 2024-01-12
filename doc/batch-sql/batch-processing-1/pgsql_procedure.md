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
        
## 스토어드 프로시저
- 스토어드 프로시저(Stored Procedure)는 서로 데이터를 주고받아야 하는 여러 쿼리를 하나의 그룹으로 묶어서 독립적으로 실행하기 위해 사용하는 것으로, 여러 SQL 쿼리를 하나의 블록으로 묶어 놓은 것입니다.
- ![image](https://github.com/mjs1995/muse-data-engineer/assets/47103479/8b383d6d-7aee-48b0-82f0-1c6de278c4a2)
  - https://www.oreilly.com/library/view/understanding-db2-learning/0131859161/0131859161_ch07lev1sec14.html
- 이를 통해 데이터베이스에 저장되어 있는 데이터에 대한 복잡한 처리를 한 번에 수행할 수 있으며, 프로그램 코드와 데이터베이스 사이에서의 데이터 전송을 최소화할 수 있습니다. 특히, 배치 프로그램과 같이 첫 번째 쿼리의 결과를 이용하여 두 번째 쿼리를 실행해야 하는 경우에 매우 유용합니다.
- 스토어드 프로시저를 작성할 때는 다음과 같은 구조를 사용합니다.
  - ```sql
    create [or replace] procedure procedure_name(parameter_list)
    language plpgsql
    as $$
    declare
    -- variable declaration
    begin
    -- stored procedure body
    end; $$
    ```
    - 키워드 'create procedure': 스토어드 프로시저를 생성할 때 사용합니다. 'or replace' 옵션을 추가하면, 이미 존재하는 프로시저가 있다면 그것을 새로운 정의로 대체합니다.
    - 프로시저 이름(procedure_name): 프로시저를 식별하기 위한 이름을 지정합니다.
    - 매개변수 목록(parameter_list): 프로시저가 처리할 입력, 출력 또는 입출력 매개변수들을 정의합니다. 매개변수는 없을 수도, 하나 이상 있을 수도 있습니다.
    - 언어(Language): 프로시저가 작성된 프로그래밍 언어를 명시합니다. 여기서는 'plpgsql'을 사용하는데, 이는 PostgreSQL의 절차적 언어입니다. 다른 데이터베이스 시스템에서는 SQL, C 등 다른 언어를 사용할 수 있습니다.
    - 본문(Body): 실제 스토어드 프로시저의 로직을 작성하는 부분입니다. 여기서 데이터베이스 쿼리, 조건문, 루프 등 다양한 프로그래밍 구조를 사용할 수 있습니다.
- 스토어드 프로시저를 사용한 예제입니다.
  - ```sql
    CREATE OR REPLACE PROCEDURE create_table_employees()
    LANGUAGE plpgsql
    AS $$
    BEGIN
        IF NOT EXISTS (
            SELECT FROM pg_tables
            WHERE schemaname = 'public'
            AND tablename  = 'employees'
        ) THEN
    		CREATE TABLE employees (
    					employee_id serial PRIMARY KEY,
    					full_name VARCHAR NOT NULL,
    					manager_id INT
    				);
    
            INSERT INTO employees (employee_id, full_name, manager_id) VALUES
                (1, 'Michael North', NULL),
                (2, 'Megan Berry', 1),
                (3, 'Sarah Berry', 1),
                (4, 'Zoe Black', 1),
                (5, 'Tim James', 1),
                (6, 'Bella Tucker', 2),
                (7, 'Ryan Metcalfe', 2),
                (8, 'Max Mills', 2),
                (9, 'Benjamin Glover', 2),
                (10, 'Carolyn Henderson', 3),
                (11, 'Nicola Kelly', 3),
                (12, 'Alexandra Climo', 3),
                (13, 'Dominic King', 3),
                (14, 'Leonard Gray', 4),
                (15, 'Eric Rampling', 4),
                (16, 'Piers Paige', 7),
                (17, 'Ryan Henderson', 7),
                (18, 'Frank Tucker', 8),
                (19, 'Nathan Ferguson', 8),
                (20, 'Kevin Rampling', 8);
        END IF;
    END;
    $$;
    ```
  - 프로시저를 데이터베이스에 추가한 후에 다음 명령어로 호출할 수 있습니다.
    - ```sql
      CALL create_table_employees();
      ```
  - 생성된 테이블을 조회합니다.
    - ```sql
      SELECT * FROM employees;
      ```
    - ![image](https://github.com/mjs1995/muse-data-engineer/assets/47103479/0c721615-3c4c-4c83-b2df-803607be4c2e)

# Reference
- [Postgresql doc](https://www.postgresql.org/docs/current/sql-createprocedure.html)
- [Real MySQL 8.0 (2권)](https://product.kyobobook.co.kr/detail/S000001766483)
