# 1강

# 강사님 소개 및 환경 세팅

오향희 강사

freeohh1216@gmail.com

https://bit.ly/sqlt_sw

# 데이터베이스 구조

[메모리 + Background Process] - Instance

====== (커넥션)

Database

→ 두개를 합쳐서 오라클 데이터베이스 서버

특정 커넥션 → (특정 유저로 접속) 세션

### 실제 데이터 베이스 = 제어 파일 + 데이터 파일 + 온라인 리두 로그 파일

- Redo Log file (모든 변경 히스토리 저장) ⇒ select 을 제외한 모든 명령어
    - Archived log file ⇒ DB 운영 mode 는 Noarchive log, archive log 모드가 있는데, archivelog 일때 저장 가능.
- 제어 파일(Control File) ⇒ 메타 데이터
    - 파라미터 파일
- 데이터 파일
    - 백업 파일
- 경고 로그 및 트레이스 파일
    - 트레이스 파일  → Background Process, Server Process(튜닝하기 위한 정보를 OS file로 저장한다)

## 논리적 및 물리적 데이터베이스 구조

- 물리적으로 떨어진 저장 공간 여럿(데이터 파일)을 하나로 관리하기 위해서 논리적 저장 공간 하나(테이블 스페이스)로 저장

```sql
create tablesapce insa

datafile ‘c:\prod\oradata\insa01.dbf’ size 100m;
```

- create ⇒ object(객체) → segment(Table, Index, Undo, Temporary)

```sql
create table emp
...
tablespace insa;

create table dept
...
tablespace insa;

100M ~ 40K ~ 40K
```

emp_1이 40k, dept_1이 40k, emp_2가 40k 받음. → 연속된 블록 공간을 익스텐트(Extent)라고 함

→ 하나의 익스텐트 = 여러개의 오라클 데이터 블

## 작업

1. Parse(Soft, Hard)
    1. Syntax Check
    2. Semantic & Privilege Check(의미, 권한 체크)
        
        ⇒ Dictionary Cache
        
    3. P_code(=parse the form) + 실행 계획 → Hard Parse
        
        ⇒ Library Cache
        
2. Execute
    1. 여기서 Data File에서 가져와서 DB Buffer Cache에 올려 놓는다.
3. Fetch

⇒ select 시, 값을 리턴하는 과정

### Shared Pool → Library Cache, Dictionary Cache

- Dictionary Cache : 권한 처리, 메타 정보(테이블이 있는가 등)를 캐싱해놓음
- Library Cache : 컴파일된 코드, 실행 계획 캐싱

### DB Buffer Cache

- 실행 계획 대로 데이터베이스에서 가져와서 캐싱해놓는 곳

### Redo Log Buffer

- Select 시에는 사용이 안됨

```sql
update emp
set sal = sal * 1.1
where empno = 7788;
```

- 메모리 상(같은 DB Buffer Cache)에 원본 블록에 대한 복사본을 둔다면?

→ 나중에 데이터 파일로 내릴 때 오버라이트가 되어서 문제가 된다.

→ undo segment(block)에 따로 기록

→ select 제외한 모든 명령어는 redo log buffer에 먼저 기록된다.

Large Pool : 병렬 처리 용

Java Pool : 자바 연결용

Streams Pool : 디비 복제 솔루션용아이콘 추가

## Q&A

변경 중에 Select 하는 경우에는 어떤가요?

select 할 때, buffer cache를 봤을 때 변경 중인 commit 되지 않은 것은 가져오지 않는다. → undo segment보고 select를 위한 읽기 일관성 블럭을 다시 만들고 → output 리턴을 해준다.

### Result Cache

- deptno=20을 계속 읽어오는 경우면, DB Buffer Cache에서 긁어 모으는 것도 느리니까.. Result Cache에 저장한다

⇒ 운영계는 계속 데이터가 바뀌기 때문에 잘 쓰지 않는다. 분석계는 쓴다?

## PGA(Program Global Area)

```sql
Select *
from emp
where empno = 30
order by sal desc;
```

- order by는 사용자에게만 제공 되면 되기에 SGA가 아닌 PGA

- 세션 메모리
select sysdate from dual;

→ 에서 25/07/14는 한국 OS라서

→ 내가 작업 중엔 sql 창 하나 → session

→ PGA의 세션 메모리?

- 커서 상태

→ sql 문장 등, 실행 컨텍스트?

- SQL 작업 영역

→ 정렬(sort_area_size), 해시(hash_area_size), 비트맵 병합(create_bitmap_merge_area_size), 비트맵 생성

→ 예전엔 하나씩 해줬다?(수동 제어) → 정렬 크기를 정해주기가 애매하다?

→ pga_aggregate_target 으로 총합을 설정해둔다.(자동적으로 하게끔)

# 옵티마이저 이해

## SQL PLUS

방법 2가지

1. 윈도우
    1. run as command line
    2. 글꼴 바꾸고 ‘ho cls’
    3. show user;
    4. connect sqlt01/oracle
2. cmd
    1. cd \
    2. cd sqlt
    3. sqlplus sqlt01

아래 세팅하는 법

```sql
alter session set nls_date_fomat=’rr/mm/dd’

set linesize 200;

set pagesize 1000;

column ename format a10;

col job for a10;

SQL> col mgr for 99999
SQL> col empno for 99999
SQL> col sal for 99999
SQL> col comm for 99999
```

## 과정

구문 및 의미 확인 → 권한 확인 → 전용 SQL 영역 할당 → 기존 공유 SQL 영역 있는지 확인(컴파일 이전에) → SOFT/HARD PARSING이 나뉨

SQL은 결과 지향의 언어 → 과정을 구현할 수 있는데 그게 PL

```sql
set timing on
DECLARE
    v_cnt NUMBER;
BEGIN
    FOR i IN 1..10000 LOOP
        EXECUTE IMMEDIATE 'SELECT count(*) FROM emp WHERE empno = '||i
        INTO v_cnt;
    END LOOP;
END;
/

select sql_text, sql_id, parse_calls, executions, plan_hash_value
from v$sql
where sql_text like 'SELECT count(*)%'
and   parsing_schema_name = 'SQLT01';

-- 실행 계획에 바인딩을 걸어주면 더 빨라짐

set timing on
DECLARE
    v_cnt NUMBER;
BEGIN
    FOR i IN 1..10000 LOOP
        EXECUTE IMMEDIATE 'SELECT count(*) FROM emp WHERE empno = '||i
        INTO v_cnt;
    END LOOP;
END;
/
```

## 왜 옵티마이저가 필요한가요?

- 인덱스가 항상 빠르진 않다. → 통계적으로 그 양이 적은 경우 인덱스가 빠르나, 아닌 경우 풀 테이블 스캔이 빠름(1% vs 80%)

## Optimizer의 2가지 모드(Query Optimizer)

1. CBO(Cost Based Optimizer)
2. RBO(Rule Based Optimizer) → deprecated

예시)

1)비행기 2)KTX 3)걸어서 ⇒ 무조건 1) 선택하는게 RBO.. 더이상 안쓰는데, 내부적으로는 조금씩 쓰인다고 함

```csharp
show parameter optimizer;
NAME                                 TYPE                   VALUE
------------------------------------ ---------------------- ------------------------------
optimizer_capture_sql_plan_baselines boolean                FALSE
optimizer_dynamic_sampling           integer                2
optimizer_features_enable            string                 11.2.0.2
optimizer_index_caching              integer                0
optimizer_index_cost_adj             integer                100
optimizer_mode                       string                 ALL_ROWS
optimizer_secure_view_merging        boolean                TRUE
optimizer_use_invisible_indexes      boolean                FALSE
optimizer_use_pending_statistics     boolean                FALSE
optimizer_use_sql_plan_baselines     boolean                TRUE

optimizer_mode -> ALL_ROWS -> CBO 중의 한 타입, first_rows도 있음
```

select *

from tab1 (1억건)

where a=10 (10만건)

order by b,c;

### ⇒ 10만건이 리턴되기까지 가장 빠른 방법 → ALL_ROWS

→ 단점 : 정렬 되기전까진 안보임. 웹에서는 잘 안맞는 경우도 있다.(쇼핑), 장점 : 배치성 업무에서는 괜찮다.

### ⇒ 10만건 중 정렬된 첫 번째 행이 리턴되기까지 가장 빠른 실행 계획 → FIRST_ROWS

### 예시

```sql
explain plan for
select *
from emp
where deptno = 10
and sal > 4000;

select * from plan_table;

select * from table(dbms_xplan.display);

------------------------------------------------------------------------------------------
| Id  | Operation                   | Name       | Rows  | Bytes | Cost (%CPU)| Time     |
------------------------------------------------------------------------------------------
|   0 | SELECT STATEMENT            |            |     1 |    38 |     2   (0)| 00:00:01 |
|*  1 |  TABLE ACCESS BY INDEX ROWID| EMP        |     1 |    38 |     2   (0)| 00:00:01 |
|*  2 |   INDEX RANGE SCAN          | EMP_SAL_IX |     3 |       |     1   (0)| 00:00:01 |
------------------------------------------------------------------------------------------
```

- explain plan for : parse만 한다 ⇒ plan_table에 저★(시험)
- 다만 이것만으로는 포맷이 애매해서, 제일 밑의 구문으로 체크

### RBO식 힌트 주기 : 주석 뒤에 + 달기

 — + or /*+ */

```sql
explain plan for
select /*+ rule */ *
from emp
where deptno = 10
and sal > 4000;

-----------------------------------------------------
| Id  | Operation                   | Name          |
-----------------------------------------------------
|   0 | SELECT STATEMENT            |               |
|*  1 |  TABLE ACCESS BY INDEX ROWID| EMP           |
|*  2 |   INDEX RANGE SCAN          | EMP_DEPTNO_IX |
-----------------------------------------------------
```

sal → deptno로 바뀌었다.. Rule로 지정해주었기 때문. 이는 기본적으로 = 으로 하는게 보통은 더 빠르기 때문이다. → 하지만 실제로는 deptno 조건은 3건, sal 조건은 1건이므로 sal이 더 좋은 인덱스

select * from emp (100만건)

from deptno = 10; (50만건)

→ 나중에 parse 할 떄 파악? 너무 늦음 → optimizer가 통계를 만들어 놓음 → 하지만 너무 과거라면?

→ deptno = 10(5명), deptno = 20 (5만명) 으로 되어 있어서..

→ index scan했는데(인덱스 한 블럭, 테이블 한 블럭, 엄청난 I/O)

→ 데이터 변화가 많을 때는, 예쁜 통계 정보를 만들어 놓아야 함.(데이터 베이스 관리자 dba가 만듦 : insert, delete 등의 비율이 10% 넘어가면.. 한다 → 다 볼 순 없으니 샘플 추출해서 함)

→ 통계를 보면.. density : 1/distinct 갯수

→ 통계는 실행 계획 때 임시적으로 생성되기도 한다???

## 옵티마이저 구성요소

S1→ parse → S1

S1 → parse → S2

여기서 S1을 tuning 해봤자 의미 없을 수 있다.

### 실행 계획 예쁘게 보기(sqlplus)

```sql
결과치 + 실행 계획 + 통계 -> set autotrace on -- 이거
실행 계획 + 통계 → set autotrace traceonly;
실행 계획 → set autotrace traceonly explain;
select *
from emp
where sal >= 3000
and sal <= 5000;

select *
from emp
where sal between 3000 and 5000;
-- 이걸로 해도 위와 똑같이 컴파일 됨.
```

set auto trace on

- 결과치
- 실행 계획
- 통계

실행 계획 + 통계 → set autotrace traceonly;

실행 계획 → set autotrace traceonly explain;

### 쿼리 변환기

⇒ 어느 표현이 더 효율적인지 따져보고 쿼리 변형기에서 변환해줌.

```sql
SELECT *
	FROM employees
	WHERE job_id = 'ST_CLERK'
	OR department_id = 10;
	
	
	
SELECT *
	FROM employees
	WHERE job_id = 'ST_CLERK'
UNION ALL
SELECT *
	FROM employees
	WHERE department_id = 10 
	and job_id <> 'ST_CLERK';
	
	
```

- 밑의 것이 cost가 더 높게 나와서 안 바꿈.
- 이걸 수행전에 결정하기 위해서 통계로 예측함

### Query Estimator : 선택성 및 카디널리티

```sql
select column_name, num_distinct, density
from user_tab_columns
where table_name = 'EMPLOYEES';
```

selectivity(선택성 ) = 조건 만족 행 수 / 총 행 수

cardinality = 총 행 수 * 선택성

통계가 없다면 임시로 만든다?? → dynamic sampling?

선택성은 낮을 수록 좋다?

비용 = 단일 블록 I/O + 다중 블록 I/O + CPU 비용 / sreadtim?

```sql
select channel_id, count(*)
from sales
group by rollup(channel_id);

select column_name, num_distinct, density
from user_tab_columns
where table_name = 'SALES';

select prod_id, sum(amount_sold)
from sales
where channel_id = 9
group by prod_id;
```

- channel_id가 25%라고 통계가 잡혀있어서, Full_Scan 했는데, 이는 사실 비효율적이다. 왜냐하면 실제로 channel_id=9의 경우에 아주 적기 때문에

⇒ 이를 위헤서는 정확한 통계 작업을 만들어줘야한다.

```sql
desc dbms_stats -- 여기를 보면 OWNNAME, TABNAME을 지정해줘야한다.

exec dbms_stats.gather_table_stats('sqlt01', 'sales', method_opt=>'for columns size auto channel_id');

begin
dbms_stats.gather_table_stats('sqlt01', 'sales', method_opt=>'for columns size auto channel_id');
end;
/
```

## Q&A

size의 의미??

→ bucket의 개념

→ popular value 개념으로 histogram을 만들어 통계정보를 계산해준다.

→ non popular value도 계산식 있음.

# 실행 계획 생성

- 옵티마이저 실행 계획을 생성
- 계획을 PLAN_TABLE에 저장
- 명령문 자체를 실행하지 않음.

### PLAN_TABLE

단점 : 실제 실행 계획이 아닐 수 있음(위치가 바뀔 수 있음)

### EXPLAIN PLAN

이 PLAN_TABLE 에 행을 삽입

가장 들여쓰기가 많이 된 것부터 순차적으로 읽어주면 된다.

```sql
select * from table(dbms_xplan.display);
```

⇒ 이렇게 파라미터를 하나도 주지 않으면, 직전 실행 계획을 보여준다.

### 힌트 사용 보고

힌트가 어떻게 적용되었는지 확인해볼 수 있음