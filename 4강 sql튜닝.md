# 4강

# 세팅

cmd 환경에서 할 것

```sql
C:\Users\User>cd \
C:\>cd sqlt
C:\SQLT>sqlplus sqlt01/oracle
SQL> set linesize 200
SQL> set pagesize 500
SQL> alter session set statistics_level = all;

Session altered.

Elapsed: 00:00:00.00
SQL> set timing off

@xplan

```

- 도스에서 그 디렉터리의 스크립트 xplan을 사용할 수 있음

→ 오늘은 cmd 환경에서 하는 이유

- SQL> alter session set statistics_level = all;

→ 실행 계획은 libarary cache에 저장 되는데.. 이 부분까지 들음

# 7. 테이블 액세스 최적화

```sql
select /*+ index(customers(cust_credit_limit)) */
			 cust_id, cust_last_name, cust_year_of_birth, cust_city, country_id
from   customers
where cust_credit_limit between 1500 and 3000
and cust_year_of_birth = 1990;
```

- /*+ index(customers(cust_Credit_limit)) */ 이것도 가능
- @xplan으로 library cache에 저장된 것과, 실제 실행 계획을 보여준다?
- 어떻게 개선할 수 있을까?

```sql
-- 1. 통계 정보 체크
select column_name, num_distinct, density
from user_tab_col_statistics
where table_name = 'CUSTOMERS';

-- 2. 인덱스 활용
select /*+ index(customers(cust_year_of_birth)) */
			 cust_id, cust_last_name, cust_year_of_birth, cust_city, country_id
from   customers
where cust_credit_limit between 1500 and 3000
and cust_year_of_birth = 1990;

-- 3.결합 인덱스 만들기 
create index credit_year_ix on customers(cust_credit_limit, cust_year_of_birth);

select /*+ index(customers credit_year_ix) */
			 cust_id, cust_last_name, cust_year_of_birth, cust_city, country_id
from   customers
where cust_credit_limit between 1500 and 3000
and cust_year_of_birth = 1990;

-- 4. 결합 인덱스 2
create index year_credit_ix on customers(cust_year_of_birth, cust_credit_limit);

select /*+ index(customers year_credit_ix) */
			 cust_id, cust_last_name, cust_year_of_birth, cust_city, country_id
from   customers
where cust_credit_limit between 1500 and 3000
and cust_year_of_birth = 1990;
```

- dictionary table(system table) 에서 통계 체크 - 대문자로 저장되어 있음.
- 싱글 인덱스 후 결합 인덱스 해보기.. 분포도가 안좋은거 부터 + 범위로 한 것을 우선했더니

→ 첫 번째 컬럼에 대한 범위 검색을 하지 않더라.. 그냥 바로 skip scan

1. 구별이 잘 되고
2. = 으로 비교되는 컬럼을 앞에 두고 결합 인덱스를 만들면..

성능이 향상 된다.

## 문제 2

```sql
select channel_id, count(*)
from sales
where channel_id <=4
and to_char(time_id, 'YYYY') >= '2000'
GROUP BY channel_id;

```

- 어떻게 튜닝할까?

```sql
exec dbms_stats.gather_table_stats('sqlt01', 'sales', method_opt=>'for all columns size 1');
alter system flush buffer_cache;
alter system flush shared_pool;

-- 기존 쿼리
select channel_id, count(*)
from sales
where channel_id <=4
and to_char(time_id, 'YYYY') >= '2000'
GROUP BY channel_id;

-- 오른쪽으로 넘김
select channel_id, count(*)
from sales
where channel_id <=4
and time_id >= to_date('2000/01/01','yyyy/mm/dd')
GROUP BY channel_id;

-- 결합 인덱스 1
create index channel_time_ix on sales(channel_id,time_id);

select /*+ index_ffs(sales channel_time_ix) */channel_id, count(*)
from sales
where channel_id <=4
and time_id >= to_date('2000/01/01','yyyy/mm/dd')
GROUP BY channel_id;
```

- 같은 쿼리라면, 함수가 더 적게 쓰인 것이 빠르다.

→ 왼쪽에 쓰이면 그 쿼리 횟수 만큼 쓰이지만, 오른쪽에 넣으면 한 번만 

### 인덱스 정리해주기

```sql
drop index credit_year_ix;

drop index year_credit_ix;

drop index channel_time_ix;

drop index time_channel_ix;
```

## 문제 4

```sql
-- 고객 테이블(CUSTOMERS)에서 출생년도(cust_year_of_birth)가 
-- 가장 늦 고객 5명을 출력하는 문장을 성능을 최적화 하여 작성하세요.

select cust_id,cust_first_name, cust_year_of_birth
from	(select cust_id,cust_first_name,cust_year_of_birth
	from customers
	order by cust_year_of_birth desc)
where rownum <=5;

select /*+ index_desc(customers(cust_year_of_birth)) */
			 cust_id,cust_first_name, cust_year_of_birth
from customers
where cust_year_of_birth > 1900 -- 암시적 변형 때문에 인덱스를 못 타는 상황을 방지
and rownum <= 5;
```

- 어떻게 개선?

→ max 구하듯이

# 8. 조인 튜닝 방법론

### 조인 비교 방법 3가지

1. Nested_Loop
2. Sort_Merge
3. Hash

```sql
update emp
set deptno = null
where empno = 7876;
commit;

select e.empno, e.ename, e.deptno, d.deptno, d.dname
from   emp e, dept d
where  e.deptno(+) = d.deptno
union
select e.empno, e.ename, e.deptno, d.deptno, d.dname
from   emp e, dept d
where  e.deptno = d.deptno(+);

select e.empno, e.ename, e.deptno, d.deptno, d.dname
from   emp e, dept d
where  e.deptno = d.deptno
and    e.sal >= 3000;

select empno, ename, deptno, dname
from   emp join dept
on     emp.deptno = dept.deptno
and    emp.sal >= 3000;
```

- 조인할 때, 테이블 이름 . 붙여주는 것을 강력 권고(겹치지 않더라도, 라이브러리 캐시 효율 향)
- 에일리어스 쓰는 것도 권장. library에 메모리 저장 때문에?
- parse에서의 1. syntax 2. sementic & privilege 2번은 select에서 체크 한다. 예전에는 os file 떨궈서 했다면, 지금은 statistics_level = all 해서 sqlplus, sql developer에서 체크 가능
- inner join과 outer join

→ full outer join은 언제 쓰는가?

→ 계획과 실제를 동시에 나타내려고 하는 경우

```sql
select e.empno, e.ename, e.deptno, d.deptno, d.dname
from   emp e, dept d
where  e.deptno = d.deptno(+)
and    d.dname = 'SALES';

select e.empno, e.ename, e.deptno, d.deptno, d.dname
from   emp e, dept d
where  e.deptno = d.deptno(+)
and    d.dname(+) = 'SALES';
```

- distinct, group by는 hash를 써서 더 이상 정렬하지 않는데..

→ 집합 연산자는 여전히 쓴다.

→ 따라서 ANSI 기법 보다 성능이 안좋음.

- 첫 번째 쿼리에서 outer join이 제대로 적용되지 않음. why?

→ inner join 처럼 처리되기 떄문.

→ 두 번째 조건에도 (+)붙어줘야함.

→ 안 붙이면 첫 번쨰 조건을 먼저 체크하지만, 두 번쨰에 붙이면, 두 번째 조건을 먼저 체크함

→ ANSI 표준으로 드가자

```sql

--left on x
select e.empno, e.ename, e.sal, e.deptno, d.deptno, d.dname
from   emp e left outer join dept d
on     e.deptno = d.deptno;

--right on x
select e.empno, e.ename, e.sal, e.deptno, d.deptno, d.dname
from   emp e right outer join dept d
on     e.deptno = d.deptno;

--full on x
select e.empno, e.ename, e.sal, e.deptno, d.deptno, d.dname
from   emp e full outer join dept d
on     e.deptno = d.deptno;

-- and
select e.empno, e.ename, e.sal, e.deptno, d.deptno, d.dname
from   emp e left outer join dept d
on     e.deptno = d.deptno
and    d.dname = 'SALES';

-- where
select e.empno, e.ename, e.sal, e.deptno, d.deptno, d.dname
from   emp e left outer join dept d
on     e.deptno = d.deptno
where    d.dname = 'SALES';

```

- ANSI로 하면 and랑 where랑 다르다.

→ and로 하면 뒤에 것이 먼저 체크가 된다.

→ where로 하면 앞의 것이 먼저 체크가 된다.(inner join 처럼 되어버림)

⇒ and를 쓰자.

```sql

-- oracle legacy
select e.empno, e.ename, e.deptno, d.deptno, d.dname, s.grade
from   emp e, dept d, salgrade s
where  e.deptno = d.deptno
and    e.sal between s.losal and s.hisal;

-- ANSI
select e.empno, e.ename, e.deptno, d.deptno, d.dname, s.grade
from   emp e join dept d
on     e.deptno = d.deptno
and    e.sal>=3000
join   salgrade s
on     e.sal between s.losal and s.hisal;

select e.empno, e.ename, e.deptno, d.deptno, d.dname, s.grade
from   emp e join dept d
on     e.deptno = d.deptno
where  e.sal>=3000
join   salgrade s
on     e.sal between s.losal and s.hisal; -- error

select e.empno, e.ename, e.deptno, d.deptno, d.dname, s.grade
from   emp e join dept d
on     e.deptno = d.deptno
join   salgrade s
on     e.sal between s.losal and s.hisal
and    e.sal>=3000;

select e.empno, e.ename, e.deptno, d.deptno, d.dname, s.grade
from   emp e join dept d
on     e.deptno = d.deptno
join   salgrade s
on     e.sal between s.losal and s.hisal
where  e.sal>=3000;
```

- 원래 한 쪽 테이블의 조건을 어디다가 써도 상관없는데, where은 중간에 들어가면 안됨
- 그리고 outer join을 하는 경우를 생각해서 가급적 and로 하자.

## 서브 쿼리

SELECT column, 단일행 함수, 그룹함수, 500, ‘anc’, (select…)

```sql
select *
from emp
where sal > (select avg(sal)
							from emp);
```

- 비상관 서브쿼리 : 메인 쿼리의 데이터가 필요 없는 경우
- 상관 서브쿼리 : exist, 메인 쿼리를 먼저 읽음. 서브 쿼리를 실행하기 위해서 메인 쿼리의 데이터가 필요 있는 경우.

```sql
select *
from dept
where deptno = any (select deptno
													from   emp);
													
													
			

select *
from dept d
where exists (select 1
							from   emp e
							where e.deptno = d.deptno);
							
							
							
select *
from emp e
where exists (select 1
							from orders o
							where a.cust_id = e,empno);
	
	

-- 부정형
							
select *
from dept
where deptno = not in (select deptno
													from   emp);													

-- 부정형
							
select *
from emp e
where not exists (select 1
							from orders o
							where o.cust_id = e.empno);
		
		
		
SELECT *
FROM dept
WHERE deptno NOT IN (SELECT deptno FROM emp);					
							
							
SELECT *
FROM dept
WHERE deptno NOT IN (
  SELECT deptno
  FROM emp
  WHERE deptno IS NOT NULL  -- ✅ 이게 핵심!
);
					

```

- 비교할 때, null이 하나라도 있다면, 나머지들도 비교가 안되므로, 서브 쿼리 조건에 is not null 조건을 걸어줘야 한다.

```sql

select department_id, round(avg(salary)) avg_sal
from (select department_id, round(avg(salary)) avg_sal
			from employees
			group by department_id
			order by round(avg(salary)) desc)	
where rownum <=3 ;

create view avg_sal
as 
select department_id, round(avg(salary)) avg_sal
from employees
group by department_id
order by round(avg(salary)) desc;

select *
from avg_sal
where rownum <=3;

with avg_with as (select department_id, round(avg(salary)) avg_sal
									from employees
									group by department_id
									order by round(avg(salary)) desc)									
select *
from avg_sal
where rownum <=3;

update emp
set deptno = 20
where empno = 7876;
commit;
```

1. 서브쿼리 방식
2. 뷰 만들기
3. with 방식

⇒ 셋 다 성능 차이는 없다

⇒ 그러나 반복적으로 사용하는 경우에 with 쓰는게 좋음.

```sql
select deptno, sum(sal)
from emp
group by deptno
having sum(sal) > (select avg(sum(sal))
									 from   emp
									 group by deptno);
									 
									 
									 
with dept_sum_with as(select deptno, sum(sal) sum_sal
											from   emp
											group by deptno)
select *
from dept_sum_with
where sum_sal > (select avg(sum_sal)
								 from dept_sum_with);
```

- 일반 칼럼을 같이 select 하면 집계 함수를 두 번 못 쓴다.(최고중에 최고를 구하는거?)
- 밑의 방식대로 하면 효율적으로 할 수 있다. 3건의 평균만 구하니까.. 중복 계산을 막음.

→ 반복적으로 쓰여지는 경우에, with으로 하는게 더 나을 수 있음.

## Nested Loops Join

→ 선행 테이블 후 후행 테이블 접근(기준 테이블 기준으로 다 비교하기, 더 작은 테이블을 기준으로 하기)

→ 적은 양의 데이터에서 사용 가능

→ 랜덤 엑세스가 많은 편

→ 배치성이냐, 웹이냐? 

→ join된 결과값을 바로 리턴할 수 있기 때문에 웹에서 좋을 수 있음(first_rows)

driving table(기준 테이블), inner table(비교 테이블)

→ 비교 테이블의 가짓수가 많고 1:M에서 M을 담당하기에 인덱스를 만들면 효율적이다.

→ 읽었던 것을 또 읽는 random access를 줄 일 수 있다.

```sql
select /*+ leading(d) use_nl(e) */
			 d.department_id, d.department_name, e.last_name, e.salary
from   departments d join employees e on d.department_id = e.department_id;
```

- 왜 nest loops가 두 개인가?(실행 계획에서)

→ sort는 pga에서 한다, 성능은 메모리 크기에 좌우된다.

→ sort-merge join 도 그렇다.

→ 같은 블럭이면 한꺼번에 가져옴. 그게 여기선 (index rowid)로 표현된다.(예전에는 안 그랬음)

→ 그 부분을 나타내려고 하다보니 조인을 두 번하는 것처럼 보이는 것임.

- 힌트 없이 돌린다면 어떻게 될까?

## Sort Merge Join

메모리를 많이 사용 한다

→ 너무 많이 사용한다 싶으면 has join을 선택하게 되나, 반대라면 이걸로 감

1. 정렬 후 조인 수행

```sql
select /*+ leading(d) use_merge(e) no_index(d) */
			 d.department_id, d.department_name, e.last_name, e.salary
from   departments d join employees e on d.department_id = e.department_id;

select /*+ leading(e)  use_merge(s) no_index(e) */
				e.empno, e.ename, e.sal, s.grade
from emp e join salgrade s
on e.sal between s.losal and s.hisal; 

select /*+ leading(e)  use_merge(s) no_index(e) */
				e.empno, e.ename, e.sal, s.grade
from emp e, salgrade s
where e.sal between s.losal and s.hisal;

```

- 여기서 nest loop 이야기가 왜 나왔지??

## Hash Join

- 해시 파티셔닝을 이용
- = 에서만 사용 가능. 범위에서는 어려움

```sql
select /*+ leading(d) use_hash(e) */
			 d.department_id, d.department_name, e.last_name, e.salary
from   departments d join employees e on d.department_id = e.department_id;

select /*+ leading(e)  use_hash(s) */
				e.empno, e.ename, e.sal, s.grade
from emp e join salgrade s
on e.sal between s.losal and s.hisal; 

select /*+ leading(e)  use_merge(s) */
				e.empno, e.ename, e.sal, s.grade
from emp e join salgrade s
on e.sal between s.losal and s.hisal;

```

- hash로 썼는데 왜 nested loop가 떳는가?

→ 범위 비교라서 옵티마이저가 nested loop로 했는데

→ 실제로는 hash가 더 유리

# 9. 조인 튜닝 사례 연구

## 문제 1

Nested Loop 방식으로 풀라

```sql
select /*+ leading(c) use_nl(s) */ c.cust_id,
						c.cust_last_name,
						c.cust_city,
						c.cust_credit_limit,
						s.prod_id,
						s.time_id,
						s.quantity_sold
		from customers c, sales s
		where c.cust_id   = s.cust_id
			and c.cust_city = 'Los Angeles'
			and c.cust_credit_limit > 3000
			and s.time_id between to_date('1999/01/01', 'YYYY/MM/DD')
												and to_date('1999/12/31', 'YYYY/MM/DD');
```

- 우선 leading, inner 갯수를 체크해본다. s가 압도적으로 많으므로 이건 패스
- inner 쪽의 인덱스를 만들어서 낭비를 줄이는 것이 nested loop의 기본 전략
- 실행 계획을 체크해보면, 실제 조회 건수는 1936개지만, cust_ix로 인덱스 스캔할 때는 8044건을 조회한다. 낭비가 발생하는 셈
- nested loop은 인덱스 기반이므로, 그쪽으로 생각해보면.. cust_ix까지는 인덱스 서치가 되나, time_id까지는 되지 않음 → 복합 인덱스를 만들자

```sql
create index cust_time_ix on sales(cust_id, time_id);
```

- 하고 다시 돌리면 잘 나온다. 명시적으로 index로 서치하도록 지정해줘도 된다.

```sql
select /*+ use_nl(c s) index(s cust_time_ix) */ c.cust_id,
						c.cust_last_name,
						c.cust_city,
						c.cust_credit_limit,
						s.prod_id,
						s.time_id,
						s.quantity_sold
		from customers c, sales s
		where c.cust_id   = s.cust_id
			and c.cust_city = 'Los Angeles'
			and c.cust_credit_limit > 3000
			and s.time_id between to_date('1999/01/01', 'YYYY/MM/DD')
												and to_date('1999/12/31', 'YYYY/MM/DD');
```

## 문제 3

```sql
select /*+ leading(c) use_nl(c s) */
						c.cust_id, c.cust_last_name, COUNT(s.prod_id)
		from customers c, sales s
		where s.cust_id (+) = c.cust_id
			and s.channel_id (+) = 9
			and c.cust_gender = 'F'
			and c.cust_year_of_birth between 1940 and 1949
		group by c.cust_id, c.cust_last_name ;
```

```sql
no_place_group_by
								opt_param('_b_tree_bitmap_plans','false')
```

- 우리가 쓰는 대부분의 인덱스는 비트맵 인덱스다?
- 실행 계획을 보면 sales 테이블에서 59건 밖에 조회가 안되어서, 얘를 기준으로 하는게 더 나을 수 있다.(원래는 outer 바꾸면 안되는데, 여기서는 그렇게 함)

```sql
select /*+ leading(s) use_hash(s c) */
						c.cust_id, c.cust_last_name, COUNT(s.prod_id)
		from customers c, sales s
		where s.cust_id (+) = c.cust_id
			and s.channel_id (+) = 9
			and c.cust_gender = 'F'
			and c.cust_year_of_birth between 1940 and 1949
		group by c.cust_id, c.cust_last_name ;
		
		
		
select /*+ swap_join_inputs(s) use_hash(s c) */
				c.cust_id, c.cust_last_name, COUNT(s.prod_id)
from customers c, sales s
where s.cust_id (+) = c.cust_id
	and s.channel_id (+) = 9
	and c.cust_gender = 'F'
	and c.cust_year_of_birth between 1940 and 1949
group by c.cust_id, c.cust_last_name ;
```

## 문제 4

```sql
select /*+ use_nl(p s) no_place_group_by */
						p.prod_id, p.prod_name, SUM(s.quantity_sold) as sold_sum
		from products p, sales s
		where p.prod_id = s.prod_id (+)
		group by p.prod_id, p.prod_name;
```

- 함수를 적게 사용하는게 더 좋다.
- 미리 그룹 함수로 만들어 두기(서브 쿼리)

```sql
-- Oracle Legacy
select p.prod_id, p.prod_name, s.cnt
from products p, (select prod_id, sum(quantity_sold) cnt
									from sales
									group by prod_id) s
where p.prod_id = s.prod_id(+);

-- ANSI
select p.prod_id, p.prod_name, s.cnt
from products p left outer join (select prod_id, sum(quantity_sold) cnt
																	from sales
																	group by prod_id) s
on p.prod_id = s.prod_id;

```