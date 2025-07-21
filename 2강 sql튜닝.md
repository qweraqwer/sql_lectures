# 2강

# 복습

PARSE 단계에서 중요한 것 → 실행 계획 → 통계 정보

실행 계획 → explain plan for의 display → plan_table

session level에서 autotrace

결과치, 실행계획, 통계치

# MES DB 대비

1. 삼성 U 핵심 요약 강의 듣기
2. 기출 ppt 100페이지짜리 3회독

# 실행계획 살펴보기(SQL Developer 중심으로)

p.50부터

## cli 세팅

```sql
ho cls
set linesize 300
set pagesize 500

```

explain plan for → plan_table 쓰면 lpad 써야함 → table function인 dbms_xplan.display(plan table이 이미 있어야 함)

```sql
select * from table(dbms_xplan.display);
```

## SQL Developer

F10 : 실행 계획

F6 : auto trace → 결과, 실행, 통계

간단한 건 developer, 자세한 건 sqlplus

## +a

PGA에서 튜닝할 수 없는 것들 3가지 → 세션 레벨에서의 변경 기록, cursor 정보, 

```sql
select *
from customers
where country_id = 52787;

select *
from table(dbms_xplan.display_cursor(null,null,'ALLSTATS LAST'));
 -- 1 : id, 2: serial number
 -- 예상치를 보여준다. -> 통계정보는 샘플링 기반
 
-- 더 자세한 실행 계획을 보고 튜닝하기 위해선
-- parameter를 건드려야 한다
show parameter statistics
alter session set statistics_level = all;

select *
from customers
where country_id = 52787;

select *
from table(dbms_xplan.display_cursor(null,null,'ALLSTATS LAST'));

-- 이렇게 다시하면 추가 정보를 보여준다.
```

| 항목 | `dbms_xplan.display` | `dbms_xplan.display_cursor` |
| --- | --- | --- |
| **출처** | **Explain Plan (예상)** 실행 계획 테이블 (`PLAN_TABLE`) | **실제 실행된 Cursor**의 실행 계획 (Library Cache) |
| **실행 필요 여부** | `EXPLAIN PLAN FOR`로 미리 실행계획 저장해야 함 | 실제 SQL을 실행하면 자동으로 Cursor에 저장됨 |
| **통계 포함 가능 여부** | `ALLSTATS` 불가 (예상 값만 표시됨) | `ALLSTATS LAST` 가능 (실제 수행 통계 포함) |
| **튜닝 적합성** | 이론적인 예상 계획 (옵티마이저 추정치 기반) | 실제 실행된 계획이므로 튜닝에 훨씬 유용 |
| **필요 설정** | PLAN_TABLE 테이블 필요 | `statistics_level=all` 설정해야 실제 값 보임 |

### SQL Developer를 이용하지 않는 이유?

자세한 실행 계획을 보려면.. 실제 실행을 시켜야 한다. 그런데 SQL Developer는 일부 행만 로드하고, 계속해서 로드하는 형식.. 불완전하다.

→ fetch 시간까지 측정해야하는데, 어렵다.

그래도 확인하고 싶다면?

```sql
set pagesize 500;

alter session set statistics_level = all;

select *
from customers
where country_id = 52787;

select * from table(dbms_xplan.display_cursor(null,null,'allstats last'));

```

이렇게 한 번에 실행시키기

스크립트는 전체 SQL을 **한 번에 실행**하게 하므로:

- `A-Rows` (실제 처리 row 수)
- `Buffers`, `Elapsed`, `CPU` 시간
    
    👉 **정확하게 측정 가능**
    

## 한 줄 씩 자세한 정보를 얻고 싶다면?

→ 힌트 사용하기

```sql
select /*+ gather_plan_statistics */*
from customers
where country_id = 52787;
```

### 실행계획을 확인하는 세 가지 방법

# 4. 인덱스 이해 및 스캔 종류

## 세팅

```sql
alter table customers modify(cust_id not null,
														 cust_last_name not null,
														 cust_first_name not null,
														 cust_city not null);
```

## Full Table Scan

```sql
select *
from customers
where cust_gender = 'M';

select column_name, num_distinct, density
from user_tab_col_statistics
where table_name = 'CUSTOMERS';

select /*+ index(customers CUSTS_GENDER_IX) */*
from customers
where cust_gender = 'M';
```

- cust_gender는 index 임에도, selectivity(=density?)값이 높아서(선택성이 낮아서), 즉 성별이 두 개 밖에 없어서, full scan이 효율적이라 그렇게 한다.
- 아래 쿼리처럼 억지로 힌트를 줘서 index search 해보면.. cost가 훨 씬 많이 나온다.
- Full Table Scan은 한 번에 4~128블럭 씩 읽어올 수 있다.(우리 환경은 128블럭) ↔ 반면에 index는 한 번에 1개의 블럭
- 또 Data Buffer Cache를 이용하지 않고, PGA를 이용할 수 있다는 장점도 있다.

1. where 조건에 index가 없거나, 
2. 있더라도 처리해야할 데이터가 많은 경우 

→ Full Scan이 유리하다.

## rowid 활용

```sql
select deptno, dname, rowid
from dept;

select deptno, dname, rowid
from dept
where rowid='AAAE7gAAEAAAAIDAAA';
```

- rowid는 데이터의 아주 정확한 위치를 가리킨다.

→ 알기만 하면 아주 빠르지만, 어려움. 직접적으로 쿼리에 쓰는 일은 잘 없다.(참고로 행 이전이 있을 수도 있다.)

→ 인덱스 객체를 활용하여 내부적으로 처리

## B*-Tree Index Scan

select * from tab1 where col1 = 230;

1. Index search Algorithm(세그먼트 트리 느낌)으로 인덱스를 먼저 찾음(정렬되어있는 리프 블럭에서 찾는다)
2. 그 인덱스에 매칭되는 row_id로 해당 값을 찾는다.

Balanced Tree → Index 균형을 맞춘다.라는 뜻

where col between 230 and 255이면?

→ 첫번째 리프 블럭을 다 찾아보고

→ (리프 블럭들 끼리는 더블 링크드 리스트 구조로 되어 있어서) 그 다음 리프 블럭으로 바로 갈 수 있다.

### 성능이 느려지는 경우?

table block과 index block은 방식이 다르다!

table block은 bottom up 방식

230을 730으로 바꾼다고 해보자

→ table block은 그냥 바꾸면 됨 

→ 테이블을 full로 꽉 채워 넣지 않기 때문. 가변 길이 데이터 떄문에

→ index block은 업데이트가 어려움. 업데이트 개념이 없다.(왜냐? 정렬 되어있기 때문!)

*가변 길이 데이터는 어떻게 업데이트 처리하나?

→ 다른 블럭으로 이사를 한다?

→ 메타 데이터에 이사했음을 알린다. → 로의 이주 현상?

→ 이런 현상을 방지하기 위하여, 10%를 남기고 테이블을 채운다.(길이가 늘어나는 칼럼을 위하여)

*Index는?

→ update를 delete + insert 로 한다.

→ 그런데 230 제거하고 730 넣으면 정렬 떄문에 overhead 발생

→ 따로 블럭을 할당해서 내부적으로 LinkedList 처럼 삽입해준다.

*추가 사항(index)

1. 계속 삭제를 당하면 새로 블럭을 만들게 됨. 하나라도 키 값이 남아 있으면 재활용이 불가능함.
    
    → 중간 브랜치를 더 많이 만들어서 개선 가능
    
    → 하지만 level이 커지면 성능은 크게 떨어짐
    
    → 재생성이 가능? 
    
    → 인덱스 파티션으로 해결이 가능?
    
2. 계속 값이 추가 되어서 늘어나면, Balanced되지 않는다.(root 기준 오른쪽만 늘어나니까) 

### 정리하면, Table도 업데이트 시 문제가 생길 수 있지만, Index의 문제가 훨씬 크다.

## 인덱스 스캔

### 인덱스 스캔 유형

- Unique
- Range(Descending)
- Skip
- Full and fast full
- Index join

### Index Unique Scan

```sql
select *
from customers
where cust_id = 100866;
```

3가지??

제약 조건 : primary key, unique, not null, Check, foreign key (5가지)

- Check ( grade in (’A’,’B’,’C’,’D’,’F’))
- unique - 무수히 많은 null 허용, primary key - not null + unique

⇒ unique, primary key : 자동으로 unique index 만들어진다.

⇒ 중복된 데이터가 있다면 unique index를 만드는게 실패하게 되는데.. 복합키를 만들어야

## Index Range Scan

```sql
select *
from customers
where cust_id between 70 and 80;

select *
from customers
where cust_id between 70 and 80000;

select /*+ index_desc(sales SALES_TIME_IX) */ *
from sales
where time_id between to_date('2001/01/01', 'yyyy/mm/dd')
                  and to_date('2001/01/02', 'yyyy/mm/dd')
order by time_id desc;

select *
from sales
where time_id between to_date('2001/01/01', 'yyyy/mm/dd')
                  and to_date('2001/01/02', 'yyyy/mm/dd')
order by time_id desc;
```

- 처음에 있는지 보고, 없다면 Database Buffer Cache에 Root, Branch, Leaf 를 올리게 된다.
- 검색해야할 범위가 너무 많아지면 Full Table Scan을 하게 된다.
- 오름 차순이 기본이지만, 내림 차순으로 읽은 경우에는 Descending이라고 실행 계획에 뜬다.
- 꼭 힌트를 안주더라도, 인덱스 기반으로 search하는 경우가 있다.(자동으로 알아서)

select *

from tab1 (1억 건)

where a = 10 (10만 건)

order by b;

→ 정렬하지 않고도 정렬한 듯이 가져올 수 있다.

→ index로 a = 10 찾고, 차례대로..

→ 네이버에서도 이런식으로?

## Index Full Scan

```sql
select cust_city
from customers
order by cust_city;
```

- order by에 인덱스가 있는 경우

→ 순차적으로 리프 블록들 다 탐색함

## Index Fast Full Scan

```sql
select cust_city, count(*)
from customers
group by cust_city;
```

- 리프 블록을 순차적으로 읽을 필요가 없고, 그냥 리프 블록으로 바로 접근해서 읽으면 된다.
- Full scan은 정렬된 순서대로 한 블록씩 읽음(root, branch까지 거쳐가며)
- ↔ root, branch를 거치지 않고, 한 블록씩 읽을 필요도 없다.

## Index Skip Scan

where a = ?

and b = ?

and c = ?

일 때, Selectivity가 가장 좋은 b에 해당하는 거 고르고, 나머지 조건으로 필터링

- 결합 인덱스 만들기?
- 선행 키 칼럼이 없으면 예전에는 못 탔는데, 지금은 가능하다.

→ 원래는 선행 키 칼럼이 구분할 수 있는 게 더 많은 걸 놔야.

→ 그 기본 가이드를 무시하고, 선행 칼럼을 더 구분할 수 없는 것을 놓는다면?

→ 선행 키 칼럼을 무시하고 후인 키 칼럼으로 탐색함.

→ 그 과정에서 후인 키 칼럼 기준으로 skip하면서 탐색하는게 skip scanning

- 원래 index블록 위로 브랜치 안올라오지만, skip은 올라와서 탐색함

## Index Join

```sql
select cust_id, cust_city
from customers
where cust_last_name = 'King';

select cust_id, cust_city
from customers
where cust_last_name LIKE 'S%';
```

- Index Fast Full Scan
- Index  Range Scan

→ 둘 다 rowid를 가지고 있어서 join이 가능하다.

왜 King은 안그러나? → 75건 밖에 없어서 → 2블록 밖에 없다 → 그냥 range로 읽어오면 됨

많아지면..  어렵다?

## Index Skip Scan 실습

time_id + prod_id → non_unique index

```sql
select *
from sales
where time_id between '20090815'
									and '20090817'
	and prod_id = 'A';
```

- =으로 비교하면 안됨. 시분초까지 같아야하기 때문

→ 지금처럼 범위로 주어진다면, 그 구간 싹 다 서치해야함.

→ 결국 prod_id는 그냥 필터링 조건으로만 하게 됨.. 가급적 앞의 것은 =으로 비교할 수 있는 것으로 해야함.

### 세팅

```sql
-- 1. 인덱스 만들기
create index t_p_ix on sales(time_id, prod_id);

-- 2. 스크립트 한 방에 실행

alter session set statistics_level = all;

select /*+ index(sales t_p_ix) */ *
from sales
where time_id between to_date('2001/04/01', 'yyyy/mm/dd')
                  and to_date('2001/04/30', 'yyyy/mm/dd')
and prod_id = 22;

select * from table(dbms_xplan.display_cursor(null,null,'allstats last'));

-- 3. 비교를 위해 히든 파라미터 변경
alter session set "_optimizer_skip_scan_enabled" = false;

alter session set "_optimizer_skip_scan_enabled" = true;

create index p_t_ix on sales(prod_id, time_id);

select *
from sales
where time_id between to_date('2001/04/01', 'yyyy/mm/dd')
                  and to_date('2001/04/30', 'yyyy/mm/dd')
and prod_id = 22;

--어느 쪽을 쓸까?
```

- time_id + prod_id
- prod_id + time_id

둘 중에 어느 복합키가 더 빠를까?

두 번째가 더 빠르다. 

## 연습 문제

```sql
-- 1번
select max(time_id)
from sales
where channel_id = 2;

-- 2
select max(time_id), min(time_id)
from sales;

select max(time_id)
from sales
union all
select min(time_id)
from sales;

```

- max,min이 한 번에 있으면 양쪽에서 같이 접근 못한다.

→ or로 연결된 경우에도 마찬가지..

```sql
select *
from employees
where job_id = 'ST_CLERK'
or department_id = 10;

select *
from employees
where job_id = 'ST_CLERK'
union all
select *
from employees
where department_id = 10;

```

SELECT column, 단일행함수, 그룹함수, 500, 'anc’, (select …)

FROM table 명 | (select …)

WHERE column | 단일행함수              비교연산자 값 | (select …)

GROUP BY

HAVING 그룹 함수       비교 연산자           값 | (select …)

ORDER BY column, 단일행 함수, 그룹함수, 500, ‘anc’, (select …);

### From에 서브 쿼리를 쓰는 경우는 어떤 경우인가?

→ order by를 먼저 해주고 rownum으로 상위 N개 뽑아야하는 경우

→ (일반적으로) 특정 조건의 테이블을 만들어 두기 위함 (→ 사실 with이 더 좋지 않나?)

```sql

select department_id, avg_sal
from(
    select department_id, ROUND(AVG(salary)) AS avg_sal
    from employees
    group by department_id
    order by avg_sal desc
)
where rownum < 6;

```

- 바깥쪽에서 안쪽의 칼럼을 자기 칼럼 처럼 쓸 수 있는데, 괄호와 같이 허용하지 않는 특수 문자가 있는 경우에는 반드시 알리아스를 써 줘야 한다.

### 이걸 문제에 적용시키면

```sql
select max(time_id), min(time_id)
from (select max(time_id) time_id
	from sales
	union all
	select min(time_id) time_id
	from sales);
```

### 뷰를 만들어서 재활용할 수 있다.

```sql
create view dept_avg_sal
as
select department_id, round(avg(salary)) dept_avg
from employees
group by department_id
order by dept_avg desc;
```

- 뷰를 만드는 목적 99%는 from 절에서 쓰려고 하는 거

## 마지막 예제

```sql
drop index t_p_ix;

drop index p_t_ix;

select column_name, num_distinct, density
from user_tab_col_statistics
where table_name = 'SALES';

exec dbms_stats.gather_table_stats('sqlt01','sales', method_opt=>'for all columns size 1');

select channel_id, count(*)
from sales
group by channel_id;

exec dbms_stats.gather_table_stats('sqlt01','sales', method_opt=>'for columns size auto channel_id');

select max(time_id)
from sales
where channel_id=2;

-- 모범 답안
select max(time_id)
from    (select /*+ index_desc(SALES SALES_TIME_IX) */time_id
        from sales
        where time_id >= to_date('1000/01/01', 'yyyy/mm/dd')
        order by time_id desc)
where rownum = 1;
select * from table(dbms_xplan.display_cursor(null,null,'allstats last'));

```

- 이게 가능한 이유. channel_id = 2 에 해당하는 값이 많아서. 뒤에서부터 찾는데, id=2인 값이 많으니까 쌉가능. 결국 발견 확률이다.
- 9로 하면 오래 걸린다.