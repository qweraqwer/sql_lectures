# 3강

# 복습

```sql
문자 단일행 함수 : chr, ascii, instr, substr, ltrim, rtrim, trim, replace, translate, concat, upper, lower, initcap, length
number단일행 함수 : mod, round, trunc, sign, power, ceil, floor, log, exp, cos, sin, tan,....
date 단일행 함수 : between_months, add_months, extract, to_char, last_day, next_day

conversion function :
to_char (number|date, 'format string')
첫번째 전달된 값의 형식과 두번째 인수 format 형식 문자가 동일하지 않아도 변환됨
to_date ('date문자열' , '첫번째 인수의 형식과 일치하는 format string')
to_number('number문자열','첫번째 인수의 형식과 일치하는 format string')

null처리 함수 :
nvl(arg1, arg2) 
nvl2(arg1, arg2, arg3)
coalesce(arg1, ......argn)
nullif(arg1, arg2)

조건처리 함수, 조건처리 표준 표현식 :
decode(expression, 비교값1, 리턴값1, 비교값2, 리턴값2, .....,last리턴값)

case expression when 비교값1 then 리턴값1
                 [when 비교값2 then 리턴값2 
                  when 비교값3 then 리턴값3
                   ....
                  else  last리턴값]   end

case when 조건expression1 then 리턴값1 
     [when 조건expression2 then 리턴값2 
      .....
     else last리턴값] end   

group function(multiple row function, aggregation function) :
count, sum, avg, min, max, variance, stddev
모든 그룹함수는 null이 인수로 전달될 경우 연산에 적용하지 않습니다. ignore
count(null), sum(null), avg(null), min(null), max(null)
count (*) : not null컬럼 기준으로 table의 행수 리턴
group function( [ALL | DISTINCT]column )
avg(number) : 인수로 전달된 컬럼값중에서 null이 포함된 경우 전체 평균값을 리턴하지 않음
               null값을 다른 값으로 치환 후 인수로 전달되어야 전체 평균값을 리턴합니다.

select 컬럼, 컬럼, 그룹함수(컬럼) ....   	---5
from    ~                		---1
where  필터조건            		---2
group by 컬럼|표현식 			---3
having  그룹함수의 조건 			---4
order by   정렬컬럼 ... 			---6
;     

select 절에 선언된 그룹함수를 적용하지 않은 컬럼이 group by절에 선언되지 않으면  오류 발생합니다.
group by절에 선언된 컬럼을 select 절에 선언하지 않은 경우, 오류가 발생하지 않습니다.
그룹함수 조건은 having절에 선언합니다.

select deptno 
from emp
group by deptno*2;  ---? error

select deptno*2
from emp
group by deptno*2 ;

select deptno 
from emp
group by nvl(deptno, 0); ---? error

select nvl(deptno, 0)
from emp
group by nvl(deptno, 0);

listagg( 컬럼, 구분자) within group ( order  by 컬럼 정렬방식)

행(중심) 데이터 -> 특정 열 기준으로 변환 : 피봇화

select    열변환컬럼값 조건 평가 (decode, case) -> data로 사용될 컬럼값 반환 , 집계
...
group by 행인덱스컬럼

--------------------------------------------------------------
join : 2개 이상의 테이블로부터 공통 속성값이 일치할때 행들을 결합해서 검색 결과 생성

inner join, equi join
non-equi join
self join (자기 참조 테이블에서만 가능)
outer join
cartesian product, cross join  (pivot 화 테이블 -> unpivot 변환

select  a.col1, b.col2,...
from  table1 a , table2 b
where  a.join키 = b.join키
[where  a.join키 between b.join키 and b.join키 ]

select  a.col1, b.col2,...
from  table1 a , table1 b
where  a.PK키 = b.FK키

DDL
constraint 제약조건은 column에 정의하는 DML 규칙
not null
primary key  (중복 X, null X) not null + unqiue
unique  (중복 X, null O)
foreign key
chek

select  a.col1, b.col2,...
from  table1 a , table2 b;
 
select  a.col1, b.col2,...
from  table1 a , table2 b
where  a.join키 = b.join키(+)

select  a.col1, b.col2,...
from  table1 a , table2 b
where  a.join키(+) = b.join키 

SQL1999 :
select  a.col1, b.col2, join키 컬럼, ...
from  table1 a  natural join table2 b
[from  table1 a    join table2 b using (join키 컬럼)

select  a.col1, b.col2, 소유자.join키 컬럼, ...
from  table1 a  natural join table2 b
[from  table1 a   join table2 b  on a.join키 = b.join키]

select  a.col1, b.col2,...
from  table1 a left outer join  table2 b  on  a.join키 = b.join키;

select  a.col1, b.col2,...
from  table1 a right outer join  table2 b on  a.join키 = b.join키;
 
--조인 조건 양쪽에 (+)를 선언할 수 없습니다.
select  a.col1, b.col2,...
from  table1 a full outer join  table2 b  on  a.join키 = b.join키;

※ n개의 테이블을 조인할 때 최소 조인 조건은 n-1개 선언

select  a.col1, b.col2,...
from  table1 a cross join  table2 b 
```

## 문제 풀이(조건 필터는 where에? on에?)

```sql
Q> employees, departments에서 location_id가 1700인 모든 부서와 해당 부서의 모든 사원을 결과 집합으로 생성하는 Query를 작성하시오
select a.last_name, b.department_id, b.department_name, b.location_id
from employees a  right outer join  departments b  on a.department_id = b.department_id
where b.location_id =1700 
order by b.department_id ;  --34rows 

select a.last_name, b.department_id, b.department_name, b.location_id
from employees a  ,  departments b  
where  a.department_id(+) = b.department_id
and b.location_id =1700 
order by b.department_id ;  --34rows

Q> employees, departments에서 모든 부서를 포함하고  해당 부서의  급여 10000이상인 사원의 사원이름, 직무, 부서번호, 소속 부서이름을 조인 결과 생성하는 Query를 작성하시오 
select a.last_name, a.job_id, a.salary, b.department_id, b.department_name
from employees a  right outer join  departments b
on a.department_id = b.department_id 
where  a.salary >= 10000
order by b.department_id; --19rows, inner join으로 수행

select a.last_name, a.job_id, a.salary, b.department_id, b.department_name
from employees a  right outer join  departments b
on a.department_id = b.department_id and  a.salary >= 10000
order by b.department_id; --39rows, outer join으로 수행

#outer join을 수행할 때 기준 테이블의 필터 조건은 where절에 선언해야 합니다.
#outer join을 수행할 때 outer연산자가 선언된 테이블의 필터 조건은 where절에 선언하면 outer join이 아닌 inner join으로 수행되므로 반드시 on 조인조건 and 필터조건으로 선언해야 합니다.

select a.last_name, a.job_id, a.salary, b.department_id, b.department_name
from employees a  , departments b
where  a.department_id (+)= b.department_id
and a.salary >= 10000
order by b.department_id; --19rows, inner join으로 수행

select a.last_name, a.job_id, a.salary, b.department_id, b.department_name
from employees a  , departments b
where  a.department_id (+)= b.department_id
and a.salary(+) >= 10000
order by b.department_id; --39rows

```

| 조건 대상 | 조건 위치 | OUTER JOIN 유지 여부 | 설명 |
| --- | --- | --- | --- |
| **기준 테이블 (남는 쪽)** | WHERE절 | ✅ 유지됨 | 조인 이후 필터링 |
| **OUTER 대상 테이블 (NULL이 될 수 있는 쪽)** | WHERE절 | ❌ OUTER 무효, INNER처럼 동작 | Null은 조건 못 만족 |
| **OUTER 대상 테이블** | ON절 또는 `(+)` 사용 | ✅ 유지됨 | OUTER 연산자가 포함된 조인 조건으로 필터링 수행 |
1. **기준 테이블 조건 → WHERE**
2. **(조인되어지는 테이블)NULL될 수 있는 OUTER 테이블 조건 → 반드시 ON절 또는 (+) 조건에 포함**

## Join 연습 문제

```sql
[join연습문제] 
select a.ename, a.sal, a.job, a.hiredate, a.comm
from emp  a , dept  b
where a.deptno= b.deptno
and b.loc='DALLAS'
and a.sal >= 1500 ;

select a.ename, a.sal, a.job, a.hiredate, a.comm
from emp  a join  dept  b  on a.deptno= b.deptno
where b.loc='DALLAS'
and a.sal >= 1500 ;

5.   (employees, departments )  ---3188rows
select e.department_id,  e.last_name , c.last_name
from employees e , employees c
where  e.department_id = c.department_id
and   e.last_name != c.last_name
order by 1;

select e.department_id,  e.last_name , c.last_name
from employees e join employees c
on e.department_id = c.department_id   
and   e.last_name != c.last_name
order by 1;

6.
select e.last_name, e.hire_date, m.last_name, m.hire_date
from  employees e , employees m
where  e.manager_id = m.employee_id
and e.hire_date < m.hire_date;

select e.last_name, e.hire_date, m.last_name, m.hire_date
from  employees e join employees m  on e.manager_id = m.employee_id
where  e.hire_date < m.hire_date;

```

# 집합 연산자(UNION)

```sql
데이터 연결  :
1. 조인
2. subquery
3. 집합연산자 - union, union all, intersect, minus
             두개 이상의 select의 결과 집합을 단일 결과 집합으로 생성
             각 select 의 컬럼 개수, 컬럼 타입의 순서가 일치해야 합니다.
             order by 절은 마지막 select문에서만 선언할 수 있습니다

select의 결과 => result set
#union,  intersect, minus  => select의 첫번째 컬럼으로 정렬(sort연산) 후  중복 체크
# union all => append 방식, 내부적으로 sorting 연산 수행하지 않음
select  col1, col2, col3
from 
where ~
group by~
having ~
union | union all | intersect | minus
select  col4, col5, col6
from 
where ~
group by~
having ~;

desc employees  -----107rows, 현재 근무 부서와 직무
desc job_history   --10rows 과거 근무 부서와 직무 이력 정보

Q> 모든 사원들의 현재 근무 부서와 직무 , 과거 근무 부서 또는 직무 이력 정보를 출력하는 SQL을 작성하시오 (사원번호, 부서번호, 직무를 중복행은 한번만 결과 집합에 포함합니다.)
select employee_id, job_id, department_id
from employees
union 
select employee_id, job_id, department_id
from job_history;

Q> 현재 부서에서 담당하는  직무를 과거에도 수행했었던 사원 정보를 조회하는 SQL을 작성하시오 (사원번호,  직무 출력)
select employee_id, job_id, department_id
from employees
intersect
select employee_id, job_id, department_id
from job_history;

Q> 입사 이후에 직무 또는 부서를 변경한 적이 없는 사원번호만 출력하는 SQL을 작성하시오
(사원번호만 출력)
select employee_id
from employees
minus
select employee_id
from job_history;

Q> 사원들 전체 급여 평균과
   사원들을 부서번호로 그룹핑하고 부서별 급여 평균
   부서번호와 직무로 그룹핑하고 부서번호와 직무별 급여 평균을
   단일 결과 집합으로 생성하시오 (emp)
select to_number(null), '', avg(sal)
from emp
union all
select deptno, '', avg(sal)
from emp
group by deptno
union all
select deptno, job, avg(sal)
from emp
group by deptno, job;
--3번 테이블 반복 read, 비효율 access 성능

select  deptno  , job ,avg(sal)
from emp 
group by rollup (deptno , job);
--table에 1번만 access
-- rollup함수는 파라미터로 전달된 컬럼들로부터 오른쪽 컬럼을 하나씩 제외시키면서 계층 구조 형태로 subtotal, grand total 결과 생성함
-- group by  deptno , job 
-- group by  deptno  
-- group by  ()

Q> group by rollup (A, B, C);   --N+1개의 조합
-->  group by (A, B, C)
-->  group by (A, B)
-->  group by (A )
-->  group by ()

Q> 사원들 전체 급여 평균과
   사원들을 부서번호로 그룹핑하고 부서별 급여 평균과
   직무로만 그룹핑한 급여 평균과
   부서번호와 직무로 그룹핑하고 부서번호와 직무별 급여 평균을
   단일 결과 집합으로 생성하시오
select to_number(null), '', avg(sal)
from emp
union all
select deptno, '', avg(sal)
from emp
group by deptno
union all
select  to_number(null)  , job ,avg(sal)
from emp 
group by job
union all
select deptno, job, avg(sal)
from emp
group by deptno, job;
---4번 table full scan( read)

select  deptno  , job ,avg(sal)
from emp 
group by cube (deptno , job);
--group by deptno, job
--group by deptno
--group by job
--group by ()

Q> group by cube (A, B, C);  --2^3승개의 조합
--group by (A, B, C)
--group by (A, B )
--group by ( B, C)
--group by (A, C)
--group by (A)
--group by ( B )
--group by (C)
--group by ()

Q> (부서번호, 관리자, 직무 ) , (부서번호, 직무), (관리자), () 그룹핑한 급여 평균을 단일 결과집합으로 생성합니다
또는 
(부서번호, 관리자, 직무 ) , (부서번호), (직무), () 그룹핑한 급여 평균을 단일 결과집합으로 생성합니다
SELECT deptno,  mgr, job, AVG(sal)
FROM emp
group by grouping sets ((deptno,  mgr, job),(deptno,  job), (mgr), ());

select  deptno  , job , avg(sal), grouping(deptno), grouping(job)
from emp 
group by rollup (deptno , job);

102페이지 연습문제
140페이지, 141페이지 연습문제
```

## 연습 문제 풀이

```sql
102페이지 연습문제
Q1
SELECT     SUM(o.sales) "총 매출"
FROM     reservation r  JOIN     order_info o 
ON     r.reserv_no = o.reserv_no 
WHERE     r.cancel = 'N';

Q2.
SELECT     i.product_name,     SUM(o.sales) AS TotalSales 
FROM     item i  JOIN     order_info o  ON     i.item_id = o.item_id
GROUP BY  i.item_id ,  i.product_name;

Q3. 
select c.customer_name, o.reserv_no, i.product_name , o.quantity
FROM order_info o  join  reservation  r 
   on o.reserv_no = r.reserv_no
   join  item  i on o.item_id = i.item_id
   join    customer c    on r.customer_id = c.customer_id;

Q4.
SELECT     c1.customer_name AS Customer1, c2.customer_name AS Customer2,  c1.zip_code 
FROM     customer c1  JOIN  customer c2  ON     c1.zip_code = c2.zip_code
WHERE     c1.customer_id != c2.customer_id;

Q5.
SELECT     c.customer_name,     a.address_detail 
FROM     customer c  FULL OUTER JOIN     address a  ON     c.zip_code = a.zip_code;

Q6. 
SELECT r.reserv_no,  nvl(c.customer_name , '예약 고객 없음') "고객 이름"
FROM reservation r  LEFT JOIN    customer c  ON   r.customer_id = c.customer_id;

140페이지, 141페이지 연습문제
Q1.
SELECT customer_id 
FROM reservation
INTERSECT
SELECT customer_id 
FROM order_info o   join reservation  r on o.reserv_no = r.reserv_no;

Q2.
select customer_id 
from customer
minus 
select customer_id
FROM order_info o  join reservation  r  on  o.reserv_no = r.reserv_no
--where cancel='N';

Q3.
SELECT customer_id 
FROM reservation
union 
SELECT customer_id 
FROM order_info o   join reservation  r on o.reserv_no = r.reserv_no;

Q4.
SELECT  branch,  SUM(visitor_cnt)  "방문자"
FROM   reservation
GROUP BY ROLLUP(branch);

Q5.
SELECT item_id, reserv_no, SUM(quantity) "총판매량" ,  SUM(sales) "총매출"
FROM order_info 
GROUP BY CUBE(item_id, reserv_no);
```

# SUBQUERY

```sql
subquery (nested query, inner query) : 하나의 쿼리문 내부에 포함되는 쿼리문
main query(outer query) 에 데이터를 제공하는 역할을 합니다.
subquery의 실행 결과는 join으로 샐행한 결과와 동일할 수 있습니다.

select  .....(subquery)
from table,  (subquery)
where  조건컬럼 연산자 (subquery)
group by
having  조건컬럼 연산자 (subquery)
order by (subquery)

subquery에서 group by, 그룹함수, having 절 등 선언 가능
from절의 subquery에서만 order by절을 선언할 수 있습니다.

Q> employees에서 Abel 사원보다 급여를 많이 받는 사원 조회
select  *
from employees
where salary > (Abel 사원의 급여);
 
select salary
from employees
where last_name = 'Abel';

select  *
from employees
where salary > (select salary
		from employees
		where last_name = 'Abel');

# single row subquery는 단일행 비교 연산자와 함께 사용 : >, >=, <, <=, !=,...
# multiple row subquery는 복수행 비교 연산자와 함께 사용 : in, >all, <all, >any, <any

Q> employees에서 Abel사원의 직무와 동일한 직무를 담당하는 사원 조회
select  *
from employees
where job_id = (select job_id
		from employees
		where last_name = 'Abel');

Q> EMP 테이블에서 사원번호가 7521인 사원과 업무가 같고 
급여가 7934인 사원보다 많은 사원의 사원번호, 이름, 담당업무, 입사일자, 급여를 조회
--where절의 조건마다 subquery 선언 사용 가능
select *
from emp
where  job = ( 7521인 사원  업무 ) 
and sal > ( 7934 사원 급여 );

select empno ename, sal, job, hiredate
from emp
where  job = (select job
                 from emp
                 where empno = 7521)
and sal > (select sal
                 from emp
                 where empno = 7934 );

Q> 직무가 salesman인 사원들의 급여와 동일한 급여를 받는 사원을 검색하시오
select * 
from emp
where sal  in (select sal
	       from emp
	       where job ='SALESMAN'
	       order by sal desc)
and job !='SALESMAN';  ---error

Q> EMP 테이블에서 급여를 제일 많이 받는 사원 조회하시오(subquery 사용 구현)
select *
from emp
where  sal = (최고급여);

select *
from emp
where  sal = (select max(sal)
              from emp)

```

### 이러한 문제를 join으로 구현하려면, non-equi 조인으로서, 부등식을 세워야함.

## 멀티 행 비교

```sql
Q> 각 부서별로 가장 낮은 급여를 받는 사원이름과 급여, 부서번호를 조회하는 SQL을 작성하시오
select ename, deptno, sal
from emp
where deptno in (select deptno
                 from emp)
and sal in (select min(sal)
            from emp
            group by deptno); --non-pair wise비교

select ename, deptno, sal
from emp
where (deptno, sal) in ( select deptno, min(sal)
                         from emp
                          group by deptno);  --pair wise비교, multiple column subquery

Q> EMP 테이블에서 부서별 최소급여가  20번 부서의 최소 급여보다 높은 부서들의 부서번호와 최소급여를 출력하는 SQL 작성하시오
select deptno, min(sal)
from emp
group by deptno
having min(sal) > (20번 부서의 최소급여);

select deptno, min(sal)
from emp
group by deptno
having min(sal) > (select min(sal)
                    from emp
                     where deptno = 20);
-->조인으로 query작성

select deptno, min_sal, min_sal_20
from (select deptno, min(sal) min_sal
      from emp
      group by deptno )  a,  
     (select  min(sal) min_sal_20
      from emp
      where deptno = 20 ) b
where a.min_sal> b.min_sal_20;

Q> 업무가 SALESMAN인 최소 한 명 이상의 사원보다 급여를 많이 받는 
사원의 이름, 급여, 업무를 조회하라 (직무는 salesman이 아닌)
select *
from emp
where sal >any   (select sal
                from emp
                where job = 'SALESMAN')
and   job <> 'SALESMAN';

Q> 업무가 SALESMAN인 모든 사원보다 급여를 많이 받는 
사원의 이름, 급여, 업무를 조회하라 (직무는 salesman이 아닌)
select *
from emp
where sal >all   (select sal
                from emp
                where job = 'SALESMAN')
and   job <> 'SALESMAN';
```

## 상관관계 쿼리 + 조인과 비교

```sql

Q> 사원이 소속된 부서의 평균 급여보다  많은 급여를 받는  사원의 정보를 조회하는 SQL을 작성하시오  (subquery, join)
correlation subquery (상관관계 서브쿼리) 
--subquery에서 main query의 컬럼을 사용해서 수행하므로 subquery가 main query의 후보행수만큼 반복 실행됩니다
select 컬럼,...
from 테이블 main 
where  조건컬럼  연산자  (select 급여 
                  from 테이블  sub
		  where main.column = sub.column)

1. main 쿼리로부터 후보행의 컬럼을 subquery에 전달
2. subquery에서는 전달받은 후보행의 컬럼을 사용해서 실행, 결과를 main query로 전달
3. subquery로부터 전달받은 값을 사용해서 후보행을 평가합니다.
4. main query의 후보행수만큼 1~3반복

#subquery에서는 main query의 컬럼을 참조할 수 있습니다 ( main query에서는 subquery 의 컬럼을 참조할 수 없습니다.)
select ename, deptno, sal
from emp main
where sal  > (select avg(sal)
              from emp
	      where main.deptno = deptno);

select a.ename, a.deptno, a.sal 
from emp a , (select deptno, avg(sal) avgsal
               from emp 
               group by deptno)  b
where  a.deptno = b.deptno
and a.sal  > b.avgsal;

describe employees   --현재 근무 부서와 직무 정보 제공
describe job_history  --과거 근무 부서와 직무 정보, 기간 제공

Q> employees에서 직무 또는 부서를 변경 이력(job_history)이 2번 이상인 사원정보을 조회
select *
from employees main
where   2<= (select count(*)
             from job_history
             where main.employee_id = employee_id);

select a.*
from employees a , (select employee_id , count(*) cnt
                   from job_history
                   group by employee_id  ) b
where  a.employee_id = b.employee_id
and b.cnt >= 2;

Q> emp의 모든 사원 정보와 소속부서의 이름을 함께  조회 , 출력하시오 (subquery)
select ename, deptno, sal, (부서이름 반환하는 select)
from  emp ;

select ename, deptno, sal, (select dname
                            from dept
                            where deptno = main.deptno) "부서이름"
from  emp main;
--select절의 cor-relation subquery 이면서 Scalar Subquery만 선언 가능
  
Q> 사원번호와 이름, 사원의 부서 위치가 DALLAS이면 TOP 출력하고 
      부서 위치가 DALLAS가 아니면 BRENCH로 출력하는 SQL문을 작성하세요
select ename, empno, ( select case loc when 'DALLAS' then 'TOP'
                                       else 'BRENCH' end
			from dept
			where deptno = main.deptno) "T or B" 
from emp main; 

select ename, empno,  
        case   when (select  loc 
                     from dept
                     where deptno = main.deptno ) = 'DALLAS' then 'TOP'
                else 'BRENCH' end  "T or B" 
from emp main; 

select  .....(subquery : 조인형태의 correlation subquery, scalar subquery)
from table,  (subquery : inline view ,  요약집합, 정렬집합)
where  조건컬럼 연산자 (subquery : nested subquery, 단일행, 복수행, mulitple column, correlation subquery )
group by ~
having  조건컬럼 연산자 (subquery : nested subquery , 단일행 subquery)
order by (subquery : 조인형태의 correlation subquery, scalar subquery)

Q> emp테이블에서 사원들의 이름, 급여, 직무, 부서번호를 부서이름으로 내림차순하여 출력하시오
(subquery, join)
select a.ename, a.sal, a.job, a.deptno
from emp a, dept b
where a.deptno=b.deptno
order by b.dname desc;

select ename, sal, job, deptno --, 부서이름
from emp  main
order by  (select dname
           from dept
           where main.deptno = deptno) desc;
```

## Rownum

```sql

Q> employees에서 급여가 높은 3명만 출력하는  SQL을 작성하시오 (급여의 내림차순으로 출력)
select last_name, salary     ---2
from employees               ---1
order by salary desc;        ---3

select rownum, last_name, salary
from employees
order by salary desc;

select rownum, last_name, salary
from (select last_name, salary
      from employees
      order by salary desc );

select rownum, last_name, salary
from (select last_name, salary
      from employees
      order by salary desc )
where rownum < 4 ;

https://livesql.oracle.com/landing/

12c버전부터 sql 구문 지원
select rownum, last_name, salary
from hr.employees
order by salary desc
fetch first 2 rows only;

select rownum, last_name, salary
from hr.employees
order by salary desc
fetch first 2 rows with ties;

Q> employees에서 급여가 높은 순으로 4, 5, 6위의 사원 정보 출력하는  SQL을 작성하시오 (급여의 내림차순으로 출력)
select  rn, last_name, salary
from (select rownum rn,  last_name, salary
	from (select last_name, salary
      		from hr.employees
	      order by salary desc )
	where  rownum < 7)
where rn > 3  ;

select last_name, salary
from hr.employees
order by salary desc
offset 3 rows
fetch next 3 rows only;     --with ties;
```

## Null,Exist

```sql

Q> emp에서 관리자인 사원들만 조회(subquery로)
-- 사원번호가 관리자 컬럼에 존재하는 레코드만 추출
select  empno, ename
from emp
where  empno in (사원테이블에서 조회된 관리자 번호  );

select  empno, ename
from emp
where  empno  in (select mgr
                  from emp) ; ---6 rows

select  empno, ename
from emp  main
where exists ( select '1'
              from emp
               where main.empno = mgr)

Q> 관리자가 아닌 일반 사원들만 조회 (subquery로)
select  empno, ename
from emp
where  empno not in (select mgr
                     from emp) ;
--(<> 7902  and <> 7698 and <> 7839, 7566, 7782, 7788 and <> null)
-- subquery에 null 포함 여부, null 때문에 연산에 영향을 받는지 여부를 체크해야 합니다.

select  empno, ename
from emp
where  empno not in (select mgr
                     from emp
                     where mgr is not null) ;

select  empno, ename
from emp  main
where not exists ( select '1'
                   from emp
                   where main.empno = mgr);
-- null 영향을 받지 않습니다.

```

## 실습 문제

P108. P110, P111, P118, P119

```sql
[subquery 실습 문제]
Q> 사원이 없는 부서 정보 조회

Q> 예약을 한 번도 하지 않은 고객들의 이름(customer_name)과 전화번호(phone_number)를 조회하세요.

Q>가장 높은 가격을 가진 상품의 이름(product_name)과 설명(product_desc)을 조회하세요

Q>'BEVERAGE'이라는 카테고리 ID에 속한 상품들의 총 판매량(quantity)을 조회하세요.

Q> 방문자 수(visitor_cnt)가 4명 이상인 예약의 고객 이름(customer_name), 예약 날짜(reserv_date), 방문자 수를 조회하세요.

Q> employees 테이블에서 커미션(COMMISSION_PCT)을 받는 직원의 사번이 저장되어 있습니다.  부서별 이름과 커미션(COMMISSION_PCT)을 받는 인원수를 출력하는 QUERY를 작성하시오

Q> emp  테이블에서 King사원에게 직접 보고하는 모든 사원 검색(King사원을 관리자로 가지는 모든 사원 검색)

Q> emp  테이블에서 급여가 사원의 소속 부서의 평균 급여보다 높으면 'Above Average', 그렇지 않으면 'Below Average'로 분류하여 직원 이름과 해당 분류를 출력하는 SQL 쿼리를 작성하세요.

```

### 나의 답안

```sql
-- 실습 문제
-- 1
select *
from dept main
where not exists (select 'A'
                    from emp
                    where deptno = main.deptno);

-- 2
select customer_name, phone_number
from customer c
where not exists (select '1'
                    from reservation
                    where customer_id=c.customer_id);

-- 3            
select product_name, product_desc
from item
where price = (select max(price)
                    from item);
                    
-- 4
select sum(quantity)
from order_info
where item_id in (select item_id
                    from item
                    where category_id='BEVERAGE');

-- 5
select (select customer_name
        from customer
        where customer_id=r.customer_id) "고객 이름",
        reserv_date, visitor_cnt
from reservation r
where visitor_cnt >= 4;

-- 6
select (select department_name
            from departments d
            where d.department_id=e.department_id
) "부서 이름", count(*)
from employees e
where e.commission_pct is not null
group by department_id;

-- 7 
select *
from emp
where mgr = (select empno
                from emp
                where ename='KING');
                
-- 8
select ename, (case when sal > (select avg(sal)
                                    from emp
                                    where main.deptno=deptno)
                        then 'Above Average'
                    else 'Below Average' end) "분류"
from emp main;
```