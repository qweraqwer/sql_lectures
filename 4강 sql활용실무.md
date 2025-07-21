# 4강

# 복습

- 인스턴스 = 메모리 + 프로세스
- 보통 인스턴스 - N개의 데이터 베이스, 오라클은 1인스턴스 - 1데이터 베이스가 원칙
- 그러나 오라클은 최근에 Container 개념 도입하여 여러 데이터 베이스 가능
- DataType 별로 Data Container를 하는게 tablespace. 필수 table space 들이 있다. **필수 테이블스페이스**: `SYSTEM`, `SYSAUX`, `UNDO`, `TEMP`, `USERS` 등이 있음
- Segment : 객체 저장소 단위 (undo segment 등)
- 연속된 data block → extent : storage 할당 단위
- oracle data block - I/O 단위
- OS page - OS block

![오라클 논리 물리 구조.PNG](%EC%98%A4%EB%9D%BC%ED%81%B4_%EB%85%BC%EB%A6%AC_%EB%AC%BC%EB%A6%AC_%EA%B5%AC%EC%A1%B0.png)

### 효율적 SQL 작성 지침

- *보단 필요한 칼럼만
- 조건 부여시, 인덱스가 생성된 컬럼에 함수 적용 및 연산을 적용하지 않는 것이 좋다.
- Like 처음에 % 쓰지 않기(트리 구조라 못찾음)
- DISTINCT 최대한 사용않기
- having 보단 where
- innert join 때는 from 절에 큰 테이블 부터
- 전처리 테이블 따로 보관/관리

- and/or 적절히 사용
- 검색 조건에 자주 사용되는 컬럼에 인덱스 생성하여 검색 속도 높이기(관리자 입장)
- 작은 테이블부터 조인

- join 조건에 = 일떄는 hash join, 아닐 떄는 sort merge join
- union 보다는 union all

```sql
집합연산자 : union, union all, intersect, minus
각 select 의 컬럼 개수, 컬럼 타입의 순서가 일치해야 합니다.
order by 절은 마지막 select문에서만 선언할 수 있습니다
union,  intersect, minus  =>  select 의 첫번째 컬럼으로 정렬 후 중복 행 선택 및 제거
union all =>  append방식

group by  에서 사용할 수 있는 함수  :  cube, rollup, groupingsets
서브 집계, 전체 집계를 포함하는 보고서 생성에 많이 사용하는 함수

subquery :
select .....(scalar, cor-relation subquery)
from   ... (inline view : 요약, 정렬 집합)
where  조건컬럼 연산자  (single row, multiple row, multiple column, cor-relation subquery)
group by
having  조건컬럼 연산자  (single row, multiple row, multiple column, cor-relation subquery)
order by  ..(scalar, cor-relation subquery)

--single row subquery는 단일행 비교 연산자와 함께 사용 : >, >=, <, <=, !=,...
--multiple row subquery는 복수행 비교 연산자와 함께 사용 : in, >all, <all, >any, <any

Top-N Query : from절의 subquery에서는 정렬 데이터 집합 , main query에서 rownum으로 제한
12c버전부터 fetch first n rows only | with ties
          offset n rows
          fetch next n rows only | with ties

cor-relation 서브쿼리와 함께 사용하는 존재 유무를 반환하는 연산자 :  exists, not exists

---------------------------------------------------------
with절 (Common Table Expression, CTE)
동일한 subquery가 여러번 하나의 select문에 nested 되는 경우 , 
subquery를 with에 정의하면 먼저 한번만 수행하고 임시테이블에 결과를 저장
쿼리를 1회성 모듈화 (하나의 select문에서는 재사용 가능, 다른 select문에서는 재사용 X)

--------------------------------------------------------
분석함수(window 함수)
--------------------------------------------------------
계층 구조 검색 
pivot절 , unpivot절

```

(첨부가 안되어서 메일, 카톡 보냄)

[https://www.notion.so](https://www.notion.so)

# With 절

```sql
with절 (Common Table Expression, CTE)
동일한 subquery가 여러번 하나의 select문에 nested 되는 경우 , 
subquery를 with에 정의하면 먼저 한번만 수행하고 임시테이블에 결과를 저장
쿼리를 1회성 모듈화 (하나의 select문에서는 재사용 가능, 다른 select문에서는 재사용 X)

Q> 전체 부서의 평균 급여보다 각 부서별 총급여가 많은 부서의 부서이름 및 총급여를 조회하라
(전체 부서의 평균 급여) < (각 부서별 총급여)
 select  avg(sum(sal))
 from emp
 group by deptno        select deptno, sum(sal)
                        from emp
                        group by deptno

with dept_sumsal as (select b.dname , sum(sal) sum_sal
                        from emp a , dept b
                        where a.deptno = b.deptno
                        group by b.dname)
select  dname, sum_sal 
from dept_sumsal
where sum_sal > (select avg(sum_sal)
                   from dept_sumsal);

--123페이지의 실습 문제
5. employees 테이블에서 각 부서별 총 급여 비용이 전체 회사의 총급여 비용의 8분의
1(1/8)을 초과하는 부서의 부서 이름을표시하는query를 작성합니다. WITH 절을
사용하여이query를 작성하고 query 이름을 SUMMARY로 지정합니다
WITH  SUMMARY AS ( SELECT department_id, SUM(salary) AS SUM_SAL 
                   FROM employees
                   GROUP BY department_id)
SELECT  *
 FROM  SUMMARY 
 WHERE SUM_SAL  > (SELECT round(sum(SUM_SAL)*1/8)
                   FROM  SUMMARY);

SELECT department_id, SUM(salary) AS SUM_SAL 
FROM employees
GROUP BY department_id
having  SUM(salary) > (SELECT  SUM(salary)* 1/8
                       FROM employees);

6.각 고객의 예약건수를 계산하여 예약건수가 전체 평균이상인 고객의 이름(customer_name)과 예약 건수를 조회하세요
with reserv as (select a.customer_id,  b.customer_name, count(reserv_no) r_cnt
               from reservation  a join customer b 
               on a.customer_id = b.customer_id
               group by customer_id, b.customer_name)
select  customer_id,  customer_name, r_cnt "예약건수"
from reserv
where r_cnt >= (select avg(r_cnt)
                from reserv);

select a.customer_id,  b.customer_name, count(reserv_no) r_cnt
from reservation  a join customer b 
on a.customer_id = b.customer_id
group by customer_id, b.customer_name
having  count(reserv_no) > = (select avg(count(reserv_no))
                              from reservation
                               group by customer_id);

7.각 고객의 총매출이'회사원' 직업(job) 그룹의 평균 매출보다 높은 고객의 이름과 매출을 조회하세요
 
select r.customer_id  ,c.customer_name,  c.job  , sum(o.sales )
from reservation  r  join order_info  o  on r.reserv_no = o.reserv_no   
     join customer  c on r.customer_id = c.customer_id
group by r.customer_id, c.customer_name, c.job 
having  sum(o.sales ) > (select    avg(o.sales )
                         from reservation  r  join order_info  o  
                              on r.reserv_no = o.reserv_no   
                              join customer  c on r.customer_id = c.customer_id
                         where c.job = '회사원');

WITH summary AS (select r.customer_id  ,c.customer_name,  c.job  , sum(o.sales) sum_sales 
                 from reservation  r  join order_info  o  
                 on r.reserv_no = o.reserv_no   
                 join customer  c on r.customer_id = c.customer_id
                group by r.customer_id, c.customer_name, c.job )
select customer_name, sum_sales
from summary 
where sum_sales > (select avg(sum_sales)
                   from summary 
                   where job = '회사원');

```

# 분석 함수

![분석함수.PNG](%EB%B6%84%EC%84%9D%ED%95%A8%EC%88%98.png)

```sql
# 분석함수(window 함수)

drop table t1  purge;

CREATE TABLE t1 (c1 NUMBER, c2 NUMBER);

INSERT INTO t1  VALUES (1, 1);
INSERT INTO t1  VALUES (2, 1);
INSERT INTO t1  VALUES (3, 2);
INSERT INTO t1  VALUES (4, 3);
INSERT INTO t1  VALUES (5, 3);
INSERT INTO t1  VALUES (6, 4);
INSERT INTO t1  VALUES (7, 5);
INSERT INTO t1  VALUES (8, 5);
INSERT INTO t1  VALUES (9, 6);
COMMIT;

SELECT C1, C2
       , COUNT(*) OVER (ORDER BY C2 ROWS BETWEEN UNBOUNDED PRECEDING AND  2  PRECEDING ) as r01
       , COUNT(*) OVER (ORDER BY C2 ROWS BETWEEN UNBOUNDED PRECEDING  AND  CURRENT ROW ) AS R02
       ,COUNT(*) OVER (ORDER BY C2 ROWS BETWEEN UNBOUNDED PRECEDING  AND  2  FOLLOWING ) AS R03
       ,COUNT(*) OVER (ORDER BY C2 ROWS BETWEEN UNBOUNDED PRECEDING  AND  UNBOUNDED  FOLLOWING ) AS R04
       , COUNT(*) OVER (ORDER BY C2 ROWS BETWEEN 2  PRECEDING  AND 1  PRECEDING ) AS R05
       , COUNT(*) OVER (ORDER BY C2 ROWS BETWEEN 2  PRECEDING AND CURRENT ROW) AS R06 
      , COUNT(*) OVER (ORDER BY C2 ROWS BETWEEN 2  PRECEDING AND 2  FOLLOWING ) AS R07
       , COUNT(*) OVER (ORDER BY C2 ROWS BETWEEN 2  PRECEDING AND UNBOUNDED  FOLLOWING ) AS R08
       , COUNT(*) OVER (ORDER BY C2 ROWS BETWEEN CURRENT ROW AND 2  FOLLOWING ) AS R09
       , COUNT(*) OVER (ORDER BY C2 ROWS BETWEEN CURRENT ROW AND UNBOUNDED  FOLLOWING ) AS R10
       , COUNT(*) OVER (ORDER BY C2 ROWS BETWEEN   1  FOLLOWING   AND 2 FOLLOWING ) AS R11
       , COUNT(*) OVER (ORDER BY C2  ROWS BETWEEN   1  FOLLOWING   AND  UNBOUNDED FOLLOWING ) AS R12
       , COUNT(*) OVER (ORDER BY C1  ) AS R13
       , COUNT(*) OVER (ORDER BY C2  ) AS R14
       , COUNT(*) OVER (ORDER BY C2  ROWS UNBOUNDED PRECEDING ) AS R15
       , COUNT(*) OVER (ORDER BY C2  ROWS 2  PRECEDING ) AS R16
       , COUNT(*) OVER (ORDER BY C2 ROWS CURRENT ROW ) AS R17       
FROM  t1;

```

### 디폴트는 range between unbounded preceding and current row → 동일 값이면 포함됨.

## 분석 함수를 쓰는 이유?

```sql

Q>각 사원들의 기본 정보(이름, 부서번호, 직무, 급여, 소속부서의 평균 급여)
select ename, a.deptno, job, sal, avg_sal
from emp a, (select deptno, avg(sal) avg_sal
             from emp
             group by deptno) b
where a.deptno = b.deptno;          ---2번 table read
 
select ename, a.deptno, job, sal, (select avg(sal)
                                   from emp
                                   where a.deptno = deptno) avg_sal
from emp a;   ---2번 table read

select ename, a.deptno, job, sal,   avg(sal) over (partition by deptno) avg_sal 
from emp a;  

```

### subquery를 쓰면 O(N^2)이라면, O(NlogN)으로 줄어든다? → 대략적으로만..

## Rank

```sql
SELECT  JOB, ENAME, SAL
        , RANK() OVER (ORDER BY SAL DESC)  ALL_RANK
        , DENSE_RANK() OVER (ORDER BY SAL DESC)  DS_RANK
        , ROW_NUMBER() OVER (ORDER BY SAL DESC)  ROW_N
FROM EMP;

Q> 부서별로 급여를 내림차순하고 순위를 출력하는 Query를 작성하시오
SELECT  DEPTNO, ENAME, SAL
        , RANK() OVER (PARTITION BY deptno  ORDER BY SAL DESC)  ALL_RANK
        , DENSE_RANK() OVER (PARTITION BY deptno  ORDER BY SAL DESC)  DS_RANK
        , ROW_NUMBER() OVER (PARTITION BY deptno  ORDER BY SAL DESC)  ROW_N
FROM EMP;
```

## 그 외

```sql
Q> 각 사원 데이터(이름, 부서번호, 직무, 급여)와 전체 급여 합계, 평균, 최소급여, 최대급여를 함께 출력하시오
SELECT DEPTNO, ENAME, JOB, SAL, SUM(SAL), MAX(SAL),  AVG(SAL), MIN(SAL)
FROM EMP; 

SELECT DEPTNO, ENAME, JOB, SAL, SUM(SAL) OVER()
         , MAX(SAL) OVER(),  AVG(SAL) OVER(), MIN(SAL) OVER()
FROM EMP; 

Q> 각 사원 데이터(이름, 부서번호, 직무, 급여)와 입사년월로 정렬한 후에 , 직전 먼저 입사한 사원의 급여와 해당 사원 다음번에 입사한 사원들을 범위로 급여 평균과 합계를 함께 출력하시오
SELECT DEPTNO, ENAME, JOB, SAL
     ,SUM(SAL) OVER(order by hiredate rows between 1 preceding and 1 following )
     , MAX(SAL) OVER(order by hiredate rows between 1 preceding and 1 following)
FROM EMP; 

Q> 각 사원 데이터(이름, 부서번호, 직무, 급여)와  함께 급여의 내림차순으로 정렬하고  해당 사원의 급여보다 300작고 300큰 범위의 급여를 받는 사원수를 출력하는 SQL을 작성하시오
SELECT DEPTNO, ENAME, JOB, SAL
     , COUNT(*) OVER(ORDER BY SAL DESC RANGE BETWEEN 300 PRECEDING AND 300 FOLLOWING)
FROM EMP;

--max(), min(),  first_value(), last_value()
SELECT EMPNO, ENAME, SAL, DEPTNO       
       , MAX(SAL) OVER(PARTITION BY DEPTNO) MAX_SAL
       , MIN(SAL) OVER(PARTITION BY DEPTNO) MIN_SAL
       , FIRST_VALUE(SAL) OVER(PARTITION BY DEPTNO) first_SAL
       , LAST_VALUE(SAL) OVER(PARTITION BY DEPTNO) last_SAL
FROM EMP;

SELECT EMPNO, ENAME, SAL, DEPTNO       
       , MAX(SAL) OVER(PARTITION BY DEPTNO ORDER BY SAL) MAX_SAL
       , MIN(SAL) OVER(PARTITION BY DEPTNO ORDER BY SAL) MIN_SAL
       , FIRST_VALUE(SAL) OVER(PARTITION BY DEPTNO ORDER BY SAL) first_SAL
       , LAST_VALUE(SAL) OVER(PARTITION BY DEPTNO ORDER BY SAL) last_SAL
FROM EMP;

--over절에 order by절을 선언한 경우 ,  FIRST_VALUE()와 LAST_VALUE()는 partition 내에 처리되는 (현재) 행까지의 범위내에서 첫번째 행의 컬럼값와 마지막 행의 컬럼값을 반환합니다.

# 행비교 함수 : LAG, LEAD => WINDOW절을 안씀
LAG(컬럼|표현식, before n row, null일때 반환할 값)

SELECT   ENAME, DEPTNO, hiredate,  SAL
        , LAG(SAL) OVER (order by hiredate)
        ,  LAG(SAL, 1, 0) OVER (order by hiredate)
         ,  LAG(SAL, 2, 0) OVER (order by hiredate)
FROM EMP;

SELECT   ENAME, DEPTNO, hiredate,  SAL
        , LEAD(SAL) OVER (order by hiredate)
        ,  LEAD(SAL, 1, 0) OVER (order by hiredate)
         ,  LEAD(SAL, 2, 0) OVER (order by hiredate)
FROM EMP;

# NTILE (n) : 데이터를 동일한 크기의 그룹으로 나눔
Q> EMP 테이블에서  급여를 내림차순으로 정렬하고   3개의 그룹으로 나누어 출력하는 SQL 작성
SELECT   ENAME, DEPTNO, hiredate,  SAL
	 ntile(3) over (order by sal desc) as group_rank
FROM EMP;

SELECT   ENAME, DEPTNO, hiredate,  SAL
	 ntile(4) over (order by sal desc) as group_rank
FROM EMP;

# cume_dist() : 행수 기준으로 누적 분포 반환
예> 특정 사원의 급여가 상위 몇 %에 해당하는지 알고 싶을 때
select ename, sal
      , trunc(cume_dist() over (order by sal desc) , 2)
from emp;

#ratio_to_report() : 전체 값에서 특정 값이 차지하는 비율 계산
select ename, sal
       , trunc(ratio_to_report(sal) over ( ) , 2)
from emp;

Q> 부서별 급여의 총계가 전체 급여 총계에서 차지하는 비율을 구하는 SQL을 작성하시오
select deptno,  salbydeptno
       , round( ratio_to_report(salbydeptno) over ( ) , 2)
from (select deptno, sum(sal) salbydeptno 
      from emp
      group by deptno)

```

### 마지막 문제에서.. 이중 윈도우 함수는 안된다. 다시 풀어보기

## 연습 문제

P.163

## 피봇(pivot, unpivot)

```sql
pivot절 , unpivot절
PIVOT  : 행 데이터(long data)를 열 데이터(wide data)로 변환하는 작업
특정 컬럼의 값 기준으로 데이터를 집계해서 행에 있는 데이터를 열로 재구성

select deptno,   sum(decode(job, 'ANALYST', sal)) ANALYST
       , sum(decode(job, 'CLERK', sal)) CLERK
       , sum(decode(job, 'MANAGER', sal)) MANAGER
       , sum(decode(job, 'PRESIDENT', sal)) PRESIDENT
       , sum(decode(job, 'SALESMAN', sal)) SALESMAN
from emp
group by deptno;

select *
from ( select 행index컬럼, 열변환할 컬럼, data컬럼
       from  테이블)
pivot (집계함수(data컬럼) for 열변환할 컬럼 in ('변환할컬럼값' as  변환될컬럼heading
                                           , .... ))

select *
from ( select deptno, job, sal
       from emp)
pivot (sum(sal) for job in ('ANALYST' as ANALYST
                            , 'MANAGER' as MANAGER
			    , 'SALESMAN' as SALESMAN));

Q> employees에서 사원들의 
  전체 사원수, 2002년, 2003년 ,2004년, 2005년 입사한 사원수를 출력
select *
from ( select  to_char(hire_date, 'YYYY') hire_year
       from employees)
pivot ( count(*)  for hire_year in ('2002' as y2002
                                    ,'2003' as y2003
                                    ,'2004' as y2004
                                    ,'2005' as y2005  ))
      join  ( SELECT COUNT(*) AS total_hired
              FROM employees ) total
      on 1=1;

select *
from ( select deptno, job, sal
       from emp)
pivot (sum(sal) for deptno in (10 as D10
                            , 20 as D20
			    , 30 as D30));

drop table pivot_emp purge;
reate table pivot_emp
as SELECT *
   FROM ( SELECT deptno, job, sal FROM  emp)
   PIVOT (sum(sal) FOR Job  IN ( 'CLERK' as clerk
                              , 'MANAGER' as manager
                              , 'SALESMAN' as salesman
                                , 'ANALYST' as analyst
                                , 'PRESIDENT' as president));
describe pivot_emp
select * from pivot_emp;
delete from pivot_emp where deptno is null;
commit;

SELECT LEVEL AS NO 
FROM dual 
CONNECT BY LEVEL <= 5;
```

## 언피봇(Unpivot)

```sql
select * 
FROM pivot_emp a   CROSS JOIN (SELECT LEVEL AS NO 
				FROM dual 
				CONNECT BY LEVEL <= 5) b ;

select deptno,   case no when 1 then 'ANALYST' 
                         when 2 then 'CLERK'
												 when 3 then 'MANAGER' 
                         when 4 then 'PRESIDENT'
                         WHEN 5 THEN 'SALESMAN'  end as JOB
             ,   case no when 1 then analyst 
                         when 2 then clerk
												 when 3 then manager
									       when 4 then president
                         WHEN 5 THEN salesman  end as sum_sal                      
FROM pivot_emp a   CROSS JOIN (SELECT LEVEL AS NO 
				FROM dual 
				CONNECT BY LEVEL <= 5) b ;

#UNPIVOT  : 열 데이터(wide data)를  행 데이터(long data)로 변환
여러 컬럼을 하나의 컬럼의 값들로 재구성 (열들을 행 데이터로 재구성)

SELECT 행index컬럼명, 열index컬럼명,  data컬럼명
FROM (피봇  Data집합 조회하는 SELECT문)
UNPIVOT ( data컬럼명  FOR 열index컬럼명 IN  (컬럼이름 [as 별칭]
                                         , ....
                                          ));

SELECT deptno, job, sum_sal
FROM pivot_emp
UNPIVOT (sum_sal FOR  job IN (analyst as 'ANALYST' 
                              , clerk as 'CLERK'
                              , manager as 'MANAGER'
												      , salesman   AS 'SALESMAN'
                              , president  AS 'PRESIDENT'));

SELECT deptno, job, sum_sal
FROM pivot_emp
UNPIVOT INCLUDE NULLS  (sum_sal FOR  job IN (analyst as 'ANALYST' 
                              , clerk as 'CLERK'
                              , manager as 'MANAGER'
												      , salesman   AS 'SALESMAN'
                              , president  AS 'PRESIDENT'));

Q> pivot_test 테이블 데이터를 unpivot합니다  (startyear, cnt )
create table  pivot_test  
as                              
select  "2001" , "2002", "2003", "2004", "2005", "2006", "2007", "2008" 
FROM ( SELECT EXTRACT(YEAR FROM hire_date) AS hire_year
       FROM employees)
PIVOT (count(hire_year) FOR hire_year IN ( 2001 AS "2001",
                                           2002 AS "2002",
                                           2003 AS "2003",
                                           2004 AS "2004",
                                           2005 AS "2005",
                                           2006 AS "2006",
                                           2007 AS "2007",
                                           2008 AS "2008"));

select startyear, cnt
from   pivot_test
unpivot ( cnt  FOR  startyear in ("2001" as 2001
                                  , "2002" as 2002
                                  , "2003" as 2003
                                  , "2004" as 2004
                                  , "2005" as 2005));
```

## 연습 문제

```sql

Q> pivot 절을 사용해서 reservation 테이블을 활용하여 branch 별로 월별 예약 수를 출력하시오

Q> pivot 절을 사용해서 고객별 월별 매출 총액을 pivot 형태로 출력하세요.(order_info, reservation, customer )

Q> pivot 절을 사용해서 지점(branch)별 예약 취소/정상 수를 출력하시오
```

# 계층 구조 검색

```sql
계층 구조 검색 
delete from emp where empno = 7000;
commit;

SELECT level, ename
FROM emp 
START WITH ename = 'KING' -- empno = 7839 , mgr is null,  job = 'PRESIDENT'
CONNECT BY  prior empno = mgr;  ---top-down 검색

SELECT lpad(' ', (level-1)*3, '_')||ename
FROM emp 
START WITH ename = 'KING'  
CONNECT BY  prior empno = mgr;

SELECT level, ename
FROM emp 
START WITH ename  = 'ADAMS'
CONNECT BY   empno = prior mgr;  ---bottom-up 검색

SELECT lpad(' ', (level-1)*3, '_')||ename,  CONNECT_BY_ROOT ename
FROM emp 
START WITH ename = 'KING'  
CONNECT BY  prior empno = mgr;

SELECT lpad(' ', (level-1)*3, '_')||ename, CONNECT_BY_ISLEAF  
FROM emp 
START WITH ename = 'KING'  
CONNECT BY  prior empno = mgr;

SELECT lpad(' ', (level-1)*3, '_')||ename,  SYS_CONNECT_BY_PATH (ename, '->')
FROM emp 
START WITH ename = 'KING'  
CONNECT BY  prior empno = mgr;

SELECT lpad(' ', (level-1)*3, '_')||ename,  CONNECT_BY_ISCYCLE
FROM emp 
START WITH ename = 'KING'  
CONNECT BY  prior empno = mgr;  --error
--NOCYCLE 키워드와 함께 사용   

SELECT ename,   CONNECT_BY_ISCYCLE
FROM emp 
START WITH ename = 'KING'   
CONNECT BY NOCYCLE prior empno = mgr;

create table t_emp (
   empno NUMBER PRIMARY KEY,
   ename VARCHAR2(50),
   mgr NUMBER,
   CONSTRAINT fk_mgr FOREIGN KEY (mgr) REFERENCES  t_emp(empno)
);

describe t_emp

INSERT INTO t_emp VALUES (1, 'King', NULL);
INSERT INTO t_emp VALUES (2, 'Blake', 1);
INSERT INTO t_emp VALUES (3, 'Clark', 2);
INSERT INTO t_emp VALUES (4, 'Jones', 3);
INSERT INTO t_emp  VALUES (5, 'Scott', 4);
INSERT INTO t_emp VALUES (6, 'Ford', 5);
INSERT INTO t_emp VALUES (7, 'Miller', 6);
INSERT INTO t_emp VALUES (8, 'Smith', 7);
commit;
update t_emp set mgr = 8 where empno = 1;
commit;

SELECT LEVEL, empno, ename, mgr, CONNECT_BY_ISCYCLE
FROM t_emp
START WITH empno = 1
CONNECT BY NOCYCLE PRIOR empno = mgr;

SELECT lpad(' ', (level-1)*3, '_')||ename 
FROM emp 
WHERE ename != 'BLAKE'             ---node를 검색 대상에서 제외시킴
START WITH ename = 'KING'  
CONNECT BY  prior empno = mgr;

SELECT lpad(' ', (level-1)*3, '_')||ename 
FROM emp 
START WITH ename = 'KING'  
CONNECT BY  prior empno = mgr and ename != 'JONES';
---branch를 검색 대상에서 제외시킴

--start with 와 connect by 절은 oracle 전용 문법
--표준 sql문으로 계층 검색하려면 CTE(WITH절)

WITH  w1 (empno, ename, mgr, lv) as (select empno, ename, mgr, 1 as lv
                                     from emp
                                     where mgr is null
                                     union all 
                                     select  c.empno, c.ename, c.mgr, p.lv+1 as lv
                                     from w1 p , emp c  
                                     where p.empno = c.mgr)
select lv, lpad(' ',  (lv-1)*3 , '_')|| ename
from w1;

WITH  w1 (empno, ename, mgr, lv) as (select empno, ename, mgr, 1 as lv
                                     from emp
                                     where ename= 'ADAMS'
                                     union all 
                                     select  c.empno, c.ename, c.mgr, p.lv+1 as lv
                                     from w1 p , emp c  
                                     where p.mgr = c.empno)
select lv, lpad(' ',  (lv-1)*3 , '_')|| ename
from w1;
```

# DML

```sql
DML - insert, update, delete

INSERT INTO 테이블 
VALUES (테이블에 선언된 컬럼순서대로 모든 값을 선언);  --1개의 행이 추가됨

INSERT INTO 테이블 (not null 제약조건이 선언된 컬럼을 포함해서 컬럼 리스트)
VALUES (  선언된 컬럼 리스트의  순서대로 모든 값을 선언); 
--values절에  단일행 함수, null, default  선언 가능

select * from dept;

INSERT INTO dept  
VALUES (50, 'IT Dev', 'Seoul' ); 

INSERT INTO  dept (loc, deptno, dname)
VALUES ('Pusan' , 60, '해외영업'); 

select * from dept;

INSERT INTO  dept (loc, dname)
VALUES ('Pusan' , 'ERP');  ---error,  not null 필수 컬럼 누락

INSERT INTO  dept
VALUES (55, null, null); --- O , null허용 컬럼은 null 허용

INSERT INTO  dept  
VALUES (40, null, null);  ---error, deptno컬럼은 PK 제약조건 선언, 중복값 허용 X

INSERT INTO  dept  
VALUES (120, null, null);  ---error, 컬럼 size 초과

INSERT INTO  emp (empno, ename, hiredate)
VALUES (7003, 'Kim',  sysdate );  ---O , 단일행 함수, 표현식를 values절에 사용 가능

INSERT INTO  emp (empno, ename, hiredate, deptno)
VALUES (7004, 'Song', sysdate,  70);  ---error, 참조 무결성 제약조건 오류 (부모 키가 없음)

select * from emp;
select * from dept;

rollback;

select * from emp;
select * from dept;

INSERT INTO VIEW | (subquery) 
VALUES (VIEW 또는 subquery 선언된 컬럼순서대로 모든 값을 선언); 

INSERT INTO 테이블 
SELECT ~
FROM ~;    ----subquery가 먼저 실행되고, 결과 행이 모두 insert
--select의 컬럼 타입과 순서가 into 절의 테이블 컬럼 타입, 순서에 일치해야 합니다.

insert into (select employee_id, last_name, email, hire_date, job_id
               from employees
               where department_id = 10)
values (300, 'Park', 'akasha.park@gmail.com', sysdate, 'IT_PROG'); 

select * from employees;

drop table t_emp purge;

create table t_emp
as select * from emp
    where 1=2;   --구조만 복제

desc t_emp
select * from t_emp;

insert into t_emp 
select * from emp 
where deptno = 30; 

select * from t_emp;

rollback;
-----------------------------------------------------------------------
UPDATE  테이블 |  뷰 | (subquery)
SET 컬럼명 = new_value ,컬럼명 = new_value ,...;

UPDATE  테이블 |  뷰 | (subquery)
SET 컬럼명 = new_value ,컬럼명 = new_value ,...
WHERE 조건 ;

select * from emp;

update emp
set sal = 0;    -- where절을 생략하면 모든 행의 컬럼값이 단일 값으로 변경됩니다.

select ename, sal
 from emp;

rollback;

update emp
set sal = 0
where deptno = 10;

select ename, sal, deptno
 from emp;

rollback;

update 테이블 | 뷰 | (subquery)
set 컬럼명 = (scalar subquery),  
where 컬럼 비교연산자 (subquery)  ; 

Q > dallas에 근무하는 사원들의 커미션을 2배 인상하는 update문 수행
update emp
set comm = comm*2
where deptno = (select deptno from dept where loc = 'DALLAS');

select * from emp;

rollback;

Q> ADAMS 사원의 급여를  CLARK 사원의 급여와 동일하게 변경하시오
update emp
set  sal = (select sal from emp where ename = 'CLARK')
where ename = 'ADAMS';

--컬럼타입, 컬럼 size , 제약조건을 위배하면 오류 

----------------------------------------------------------
DELETE [FROM]  테이블명 | View | (SubQuery) ;

DELETE [FROM]  테이블명 | View | (SubQuery) 
WHERE  조건;

select * from emp;
delete from emp;
select * from emp;
rollback;

delete from emp
where deptno =10;
select * from emp;

rollback;

#dept : emp = parent : child
#parent 테이블의 레코드중에서 child 테이블의 레코드가 참조하는 경우 삭제 불가
delete from dept where deptno  =10;  --error
delete from dept where deptno  =40;   ---O
rollback;

Q> DALLAS에 근무하는 사원들 삭제
delete from emp
where deptno = (select deptno from dept where loc = 'DALLAS');
select * from emp;
rollback;

```