# 5강

# 세팅

```sql
C:\Users\User>cd \

C:\>cd sqlt

C:\SQLT>sqlplus sqlt01/oracle

SQL*Plus: Release 11.2.0.2.0 Production on 금 7월 18 08:36:23 2025

Copyright (c) 1982, 2014, Oracle.  All rights reserved.

Connected to:
Oracle Database 11g Express Edition Release 11.2.0.2.0 - 64bit Production

SQL> set linesize 200
SQL> set pagesize 500
SQL> alter session set statistics_level = all;

Session altered.

Elapsed: 00:00:00.00
SQL> set timing off
```

# 10. 서브 쿼리 튜닝 방법론

```sql
-- M : 1, subquery
select cust_id, cust_first_name, cust_city
from customers
where country_id in (select country_id
															from  countries
															where country_region = 'Africa');
															
-- M : 1, join 															
select cu.cust_id, cu.cust_first_name, cu.cust_city
from   customers cu join countries co
on     cu.country_id = co.country_id
and    co.country_region = 'Africa';

-- 1 : M, subquery
select country_id, country_name, country_region
from   countries
where  country_id in (select country_id
											from   customers);
											
-- 1: M, join											
select distinct co.country_id, co.country_name, co.country_region
from   countries co join customers cu
on     co.country_id = cu.country_id;
```

- 대부분의 서브쿼리는 join으로 바꿀 수 있다.
- subquery로 하게 되면 hash join semi라는 말이 나오게 되는데, 이러한 transformation은 일반적으로 권장되지 않는다. 하지만 정확하게 같게 하려면 join에 distinct를 붙여야하기 때문에, 이 경우에는 그냥 서브 쿼리로 작성하는게 맞다고 할 수 있다.

```sql
select country_id, country_name, country_region
from   countries
where  country_id in (select /*+ no_unnest */ country_id
											from   customers);
```

- 옛날 방식인 FILTER 방식이라 접근하는 BLOCK 자체는 아주 많은 것을 알 수 있다.

## Ex2

```sql
select employee_id, first_name, department_id, 
			(select department_name
				from   departments d
				where  d.department_id = e.department_id) 부서명칭,
			( select avg(salary) from employees) 사원전체평균급여
from employees e;
```

- select 절에는 서브쿼리를 잘 쓰지 않는다.

```sql
select prod_id, prod_name, (select sum(amount_sold)
														from   sales s
														where  s.prod_id = p.prod_id) 상품별총매출
from products p;

-- 이렇게 쓰면, s에 판매가 되지 않은 상품은 기록되지 않으므로.. 나오지 않을 수 있다
-- 따라서 outer join을 해야함
select p.prod_id, p.prod_name, sum(s.amount_sold)
from products p left outer join sales s
on p.prod_id = s.prod_id
group by p.prod_id, p.prod_name;

select p.prod_id, p.prod_name, s.sum_amt
from products p left outer join (select prod_id, sum(amount_sold) sum_amt
																from   sales
																group by prod_id) s
on p.prod_id = s.prod_id;

```

- sales의 데이터 양이 많으니까, 미리 prod_id 별로 그루핑 해놓는다면 성능 향상에 도움이 되지 않을까?
- cost는 그냥 예상치

## Ex3

```sql
select cust_id, cust_first_name, cust_city, cust_credit_limit
from customers c1
where cust_credit_limit > (select /*+ no_unnest */ avg(cust_credit_limit) *2
														from  customers c2
														where c2.cust_city = c1.cust_city);
														

														
```

### 내 답안

```sql

with c2 as(select /*+ no_unnest */ cust_city, avg(cust_credit_limit) *2 avg_credit_limit2
														from  customers
														group by cust_city
)
select c1.cust_id, c1.cust_first_name, c1.cust_city, c1.cust_credit_limit
from customers c1 join c2
on c1.cust_city	= c2.cust_city
and c1.cust_credit_limit > c2.avg_credit_limit2;
```

### 모범 답안

```sql
select c1.cust_id, c1.cust_first_name, c1.cust_city, c1.cust_credit_limit
from customers c1 join (select cust_city, avg(cust_credit_limit) avg_credit
												from   customers
												group by cust_city) c2
on c1.cust_city = c2.cust_city
and c1.cust_credit_limit > c2.avg_credit*2;
```

## Ex4(ANTI JOIN)

```sql
-- not exist
select *
from departments d
where not exists (select 1
									from employees e
									where e.department_id = d.department_id);
									
									
-- not in									
select *
from departments d
where department_id not in (select department_id
													from employees); -- no result
													

-- not in + is not null
select *
from departments d
where department_id not in (select department_id
														from employees
														where department_id is not null);
```

- not exist는 값의 존재 유무를 join을 통해서 확인만 함. 실제 조인 결과를 리턴하진 않음
- not in으로 바꾸게 되면 안됨. null 때문에

→ in 이라면, null이 있더라도, 하나만 true가 되면 다 true가 되어서 상관없음(= or 연산)

→ not in 이라면 null이 있을 때, (~~ ≠ null) and 의 연속.. 하나가 무조건 false가 되어서 안됨. 즉 어떤 값이라도 널과 같지 않다고 할 수 없어서, not in이 안되는 것

(→ not exist는 다 비교해보고 만족하는게 없으면 되므로, 되는 )

→ where ~~ is not null 조건 ㄱㄱ

## VIEW Merge

```sql
select channel_id, sum(amount_sold)
from sales
group by channel_id
order by channel_id;

select *
from (select /*+ no_merge */ channel_id, sum(amount_sold)
				from sales
				group by channel_id)
order by channel_id;
```

- 시간이 약간 더 빨라짐

## WITH 절

```sql
select deptno, sum(sal)
from emp
group by deptno; -- avg(sum(sal))은 2개 값을 리턴해서 이 경우에 할 수 없음

select avg(sum(sal))
from emp
group by deptno;

with dept_sum_sal as (select deptno, sum(sal) sum_sal
											from   emp
											group by deptno)
select *
from dept_sum_sal
where sum_sal > (select avg(sum_sal)
									from  dept_sum_sal);
```

- 한번만 사용되면 inline_view로 자동 처리가 된다.(2번 이상이면 임시 테이블 형성 및 접근)

### 문제

고객이 거주하는 지역의 국가코드(country_id), 국가(country_name), 국가별 고객 명수, cust_credit_limit의 관계가 필요합니다.
국가별 cust_credit_limit의 합계가 전체 cust_credit_limit의 평균보다 큰 국가의 정보만 표시됩니다.

```sql
with sum_ccl_by_country as (select country_id, count(*) country_cnt, sum(cust_credit_limit) sum_ccl
															from customers
															group by country_id)
select c.country_id, c.country_name, scbc.country_cnt, scbc.sum_ccl
from countries c join sum_ccl_by_country scbc
on c.country_id = scbc.country_id
and scbc.sum_ccl > (select avg(sum_ccl)
									from  sum_ccl_by_country);
```

### 답안

```sql
select co.country_id, co.country_name, count(*), sum(cu.cust_credit_limit)
from countries co, customers cu
where co.country_id = cu.country_id
group by co.country_id, co.country_name
having sum(cust_credit_limit) > (select avg(sum(cust_credit_limit))
																	from customers
																	group by country_id);
																	
																	
																	
with country_credit as (select country_id, count(*) cnt, sum(cust_credit_limit) sum_credit
													from customers
													group by country_id)
select co.country_id, co.country_name, cu.cnt, cu.sum_credit
from countries co, country_credit cu
where co.country_id = cu.country_id
and   cu.sum_credit > (select avg(sum_credit)
											from country_credit);																		
																	

```

freeohh1216@gmail.com

오향희

# 여기부터 박태현’s

```sql
select /*+ qb_name(main) */ *
from emp
where sal > (select /*+qb_name(sub) */ sal
						 from emp
						 where empno=7566);
```

## **Case Study**

```sql
- -- 개선전
select cust_last_name, cust_postal_code
from customers c
where exists (select 1	from sales s	where s.cust_id=c.cust_id)
and cust_year_of_birth = 1980;
--- year of birth index 사용(효과 미미)
select /*+ index(c(cust_year_of_birth))*/
cust_last_name, cust_postal_code
from customers c
where exists (select 1	from sales s	where s.cust_id=c.cust_id)
and cust_year_of_birth = 1980;
select /*+ index(c(cust_year_of_birth)) */
cust_last_name, cust_postal_code
from customers c
where exists (select /*+ nl_sj */ 1	from sales s	where s.cust_id=c.cust_id)
and cust_year_of_birth = 1980;

- -- s에 대해 해당 고객의 연도가 조건에 안맞을 수 있다(비효율)
select /*+ no_query_transformation */	prod_id, cust_id, channel_id, amount_sold
from sales s
where exists (select 1	from customers	where cust_id=s.cust_id	and cust_year_of_birth=1923)
and channel_id>2;
select prod_id, cust_id, channel_id, amount_sold
from sales s
where exists (select /*+ index(c(cust_year_of_birth)) no_index(c(cust_id))*/1	from customers c	where c.cust_id=s.cust_id	and cust_year_of_birth=1923)
and channel_id>2;

select prod_id, to_char(time_id, 'yyyy/mm'), sum(amount_sold)
from sales
where time_id between to_date('2001/01/01','yyyy/mm/dd') and to_date('2001/12/31','yyyy/mm/dd')
group by to_char(time_id, 'yyyy/mm'), prod_id;

select sum(amount_sold)
from sales
where time_id >= to_date('2001/01/01', 'yyyy/mm/dd')
and time_id >= to_date('2002/01/01', 'yyyy/mm/dd');

select sum(amount_sold)
from sales
where time_id >= to_date('2001/01/01', 'yyyy/mm/dd')
and time_id < to_date('2001/02/01', 'yyyy/mm/dd')

--- 일단 결과는 나옴
select prod_id, sum(amount_sold),
			sum( case when to_char(time_id, 'yyyy/mm') = '2001/01' then amount_sold end) "2001/01",
			sum( case when to_char(time_id, 'yyyy/mm') = '2001/02' then amount_sold end) "2001/02",
			sum( case when to_char(time_id, 'yyyy/mm') = '2001/03' then amount_sold end) "2001/03",
			sum( case when to_char(time_id, 'yyyy/mm') = '2001/04' then amount_sold end) "2001/04",
			sum( case when to_char(time_id, 'yyyy/mm') = '2001/05' then amount_sold end) "2001/05",
			sum( case when to_char(time_id, 'yyyy/mm') = '2001/06' then amount_sold end) "2001/06",
			sum( case when to_char(time_id, 'yyyy/mm') = '2001/07' then amount_sold end) "2001/07",
			sum( case when to_char(time_id, 'yyyy/mm') = '2001/08' then amount_sold end) "2001/08",
			sum( case when to_char(time_id, 'yyyy/mm') = '2001/09' then amount_sold end) "2001/09",
			sum( case when to_char(time_id, 'yyyy/mm') = '2001/10' then amount_sold end) "2001/10",
			sum( case when to_char(time_id, 'yyyy/mm') = '2001/11' then amount_sold end) "2001/11",
			sum( case when to_char(time_id, 'yyyy/mm') = '2001/12' then amount_sold end) "2001/12"
from sales
where time_id >= to_date('2001/01/01', 'yyyy/mm/dd')
and time_id < to_date('2002/01/01', 'yyyy/mm/dd')
group by prod_id;

--- 튜닝(집계를 미리 해서 조인하도록)
select prod_id,
			 sum(case when ym='2001/01' then amt_sum end) "01/01매출",
			 sum(case when ym='2001/02' then amt_sum end) "01/02매출",
			 sum(case when ym='2001/03' then amt_sum end) "01/03매출",
			 sum(case when ym='2001/04' then amt_sum end) "01/04매출",
			 sum(case when ym='2001/05' then amt_sum end) "01/05매출",
			 sum(case when ym='2001/06' then amt_sum end) "01/06매출",
			 sum(case when ym='2001/07' then amt_sum end) "01/07매출",
			 sum(case when ym='2001/08' then amt_sum end) "01/08매출",
			 sum(case when ym='2001/09' then amt_sum end) "01/09매출",
			 sum(case when ym='2001/10' then amt_sum end) "01/10매출",
			 sum(case when ym='2001/11' then amt_sum end) "01/11매출",
			 sum(case when ym='2001/12' then amt_sum end) "01/12매출"		 
from (select prod_id, to_char(time_id, 'yyyy/mm') ym, sum(amount_sold) amt_sum
			from sales
			where time_id >= to_date('2001/01/01', 'yyyy/mm/dd')
			and time_id < to_date('2002/01/01', 'yyyy/mm/dd')
			group by prod_id, to_char(time_id,'yyyy/mm'))
group by prod_id;
```

### 🔑 핵심 힌트: `NL_SJ` (Nested Loops Semi Join)

- 옵티마이저가 `EXISTS` 조건을 **Nested Loops Semi Join 방식**으로 실행하도록 유도
- **SEMI JOIN**: `EXISTS`처럼 **조건을 만족하는 첫 번째 행만 찾으면 바로 종료**
- **NESTED LOOPS**: 외부(customers) → 내부(sales)를 반복 검색

---

## ✅ 왜 `NL_SJ`가 성능을 향상시키는가?

| 이유 | 설명 |
| --- | --- |
| **짧은 탐색** | Semi Join은 **첫 번째 매칭되는 값만 찾으면 즉시 종료**됨 (더 이상 sales 조회하지 않음) |
| **인덱스 활용** | sales 테이블에 **cust_id 인덱스가 있다면** → 빠르게 매칭 가능 |
| **외부 테이블 필터 선적용** | 1980년생 고객만 먼저 골라내면 → **sales 탐색 횟수 자체가 줄어듦** |
| **Nested Loop 구조 적합** | outer(고객 수) × inner(빠른 인덱스 매칭) → 반복은 있지만 빠르게 종료됨 |

[freeohh1216@gmail.com](mailto:freeohh1216@gmail.com)