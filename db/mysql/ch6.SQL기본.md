### 서브쿼리
- where 절에서의 서브쿼리
  - 서브쿼리의 결과가 단일행인 경우 문제가 없지만, 여러 행인 경우 any, all, some의 도움이 필요하다.
    - select name, height from usertbl where height >= (select height from usertbl where name = '김경호');
    - select name, height from usertbl where height >= any (select height from usertbl where addr = '경남');

### distinct
- 입사일이 오래된 직원 5명의 사원번호를 알아내라 
  - select emp_no, hire_date from employees order by hire_date limit 5;
  - 쿼리의 결과를 보니 hire_date가 모두 같아서, 입사일 자체가 중복되지 않으면서 오래된 순으로 가져와보고 싶었음
  - select distinct hire_date from employees order by hire_date limit 5;
  - 이 때 emp_no까지 같이 가져오려니까, distinct는 개별 열에 사용할 수 없어서 오류남
  - select emp_no, distinct hire_date from employees order by hire_date limit 5;
  - 위 경우 어떻게 해결??

### group by
- 전체 구매자가 구매한 물품의 개수의 평균
  - select AVG(amount) from buytbl;
  - 분모에는 테이블 행 갯수가 들어간다.
- 각 사용자 별로 한 번 구매 시 물건을 평균 몇 개 구매했는지 평균
  - select userID, AVG(amount) from buytbl group by userID;
  - 분모에는 사용자 별로 그룹핑했을 때 묶이는 행 갯수가 각각 들어간다.
