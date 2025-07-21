# 5강

# 복습

```sql
with 쿼리블럭이름1 as (select  ~
                   from ~
                   ~    ),
     쿼리블럭이름2 as  (select  ~
                    from ~ , 쿼리블럭이름1
                   ~    )  
main query;
      
1. 반복되는 subquery를 먼저 선언 후 실행 -> 임시 테이블로 저장하고, 실행 대신 read로 처리
2. 계층 검색 쿼리 (SQL 표준) 

window함수
함수(파라미터) over ( partition by 
                   order by 
rows |range  between unbounded preceding  and  n preceding
                      n preceding	      current row
                      current row	      n following
                      n following	       unbounded following

순위 관련 함수 : rank, dense_rank(), row_number()
이동 집계 : avg() over(), sum() over(), min() over(), ...
행 비교 : lag(), lead()
ntile()
cume_dist()
ratio_to_report()

Q> order_info 테이블에서 월별 매출과 전월 대비 증감률을 계산하시오
WITH  monthly_sales AS ( SELECT SUBSTR(order_no, 1, 6) AS order_month,
                               SUM(sales) AS total_sales 
			            FROM order_info 
			           GROUP BY SUBSTR(order_no, 1, 6)),
     sales_with_lag AS (SELECT  order_month, total_sales
                              , lag(total_sales) OVER (order by order_month) AS prev_month_sales
                        FROM monthly_sales )
SELECT order_month, total_sales, prev_month_sales,
        round( case when prev_month_sales is null then null
 	              when prev_month_sales = 0 then null
                  else (total_sales - prev_month_sales) / prev_month_sales * 100 end , 2)  as  growth_rate_pct
from sales_with_lag
ORDER BY order_month;
      

계층 검색 :
select  level ,....connect_by_root , sys_connect_by_path , connect_by_isleaf, connect_by_iscycle
from 
start with  조건 
connect by   prior PK = FK    ---Top down
connect by   PK = prior FK    ---Bottom-UP

-- connect_by_iscycle 가상 컬럼은 nocycle 키워드와 함께 선언
where 조건  -> 계층 검색 트리에서 노드를 제거
connect by ~  and 조건  -> 계층 검색 트리에서 특정 노드부터 시작되는 브랜치를 제거
```

## MERGE

```sql
merge 

MERGE INTO  target테이블 t
USING source테이블 s
ON (t.pk = s.pk)
WHEN MATCHED THEN 
   UPDATE  SET  t.col = s.col, ...... 
   DELETE  WHERE  조건
WHEN NOT MATCHED THEN 
   INSERT (t.col, t.col2,.....)	 
   VALUES (s.col, s.col2,....) 
   [WHERE  조건];

desc bonus
select * from bonus; 

Q> emp테이블에서 30번부서의 사원들은  커미션의 50%와 급여의 10%를 sal컬럼과 comm컬럼에 update하고 다른 부서의 사원들은 급여의 20%를 sal컬럼에 insert 하고 
update한 급여가 3000상인 사원은 삭제합니다. (bonus 테이블에 변경된 데이터 추가, 수정, ..)

insert into bonus 
select ename, job, sal, comm 
from emp
where deptno = 30;
select * from bonus;  --6rows

MERGE INTO bonus  t
USING  emp s
ON  (t.ename = s.ename)
WHEN MATCHED THEN 
     UPDATE SET  t.sal = s.sal*1.1, t.comm = s.comm*1.5  
     WHERE s.deptno = 30 
     DELETE  WHERE  t.sal>=3000
WHEN NOT MATCHED THEN 
     INSERT (t.ename, t.job, t.sal, t.comm) 
     VALUES (s.ename, s.job, s.sal*1.2, s.comm)
     WHERE deptno in (10, 20); 

select * from bonus;
```

## TCL,TX(TRANSACTION)

```sql
TCL : commit, rollback, savepoint

TX : Unit of work (ACID)
예) 계좌이체, 온라인 구매
최초의 DML을 시작할때 내부적으로 TX 시작되고 TX ID가 자동할당됨
DDL, DCL문을 수행할때 TX 시작되고 TX ID가 자동할당되며, 수행완료 즉시 auto commit; --> rollback 안됨

DML로 구성되는 TX은 명시적으로 commit 또는 rollback해야 TX종료됩니다.
TX ~ing 인 경우 DB Connection을 정상 종료   --oracle server가 commit처리 함
TX ~ing 인 경우 DB Connection을 비정상 종료  ---oracle server가 rollback처리 함 

drop table t_emp purge;

create table t_emp
as select * from emp;

--DML로 변경중인 행에는 배타 락이 설정되어 동시에 변경 못하도록 합니다.
--DML로 변경중인 테이블에는 공유 락이 설정되어 DDL문을 금지, 변경중이지 않은 행을 변경할 수 있습니다.
--select문은 Lock이 걸리지 않습니다.
--select  ~~~~ for update문은 검색 결과 데이터에 lock 을 설정

oracle은 lock과 undo를 사용해서 읽기 일관성을 보장합니다.
변경중인 사용자는 변경중인 값을 read합니다.
변경중이지 않은 사용자는 변경중인 값을 read X, commit되어 저장된 값을 read

select * from t_emp ;
update t_emp set sal = 1000 where deptno = 10; 
select * from t_emp ;
savepoint a1;
update t_emp set sal = 2000 where deptno = 20;
select * from t_emp ;
savepoint a2;
update t_emp set sal = 3000 where deptno = 30;
select * from t_emp ;
savepoint a3;
select * from t_emp ;
rollback to savepoint a2;
select * from t_emp ;
rollback ;
select * from t_emp ;
```

## DDL

```sql
DDL : 객체 (table, view, index, sequence, synonym) 구조 정의하고 생성,  구조변경, 객체 삭제

테이블 생성하려면 create table 권한이 있어야 합니다
tablespace 저장공간에 대한  quota가 있어야 합니다.
select * from session_privs;

#테이블이름, 컬럼이름 naming 규칙
- 영문자 시작, _, # 허용
- 숫자는 두번째 문자부터 사용 가능
- 30자 길이제한 
- (테이블이름, 컬럼이름은 data dictionary에 대문자로 저장)
- 키워드 X,
- 동일 스키마내에 중복 이름 X  

create table  테이블이름 (
   컬럼명  컬럼타입 [제약조건],
   ....
   ....
   
   constraint  제약조건이름 제약조건타입 (컬럼),
    ....
);

create table 테이블이름 [(컬럼명 리스트...)]
AS 
   select ...
   from  ...;

--테이블 구조만 복제
--전체 행과 구조 복제
--일부 행과 구조를 복제 가능

drop table t_emp purge;
create table t_emp
as select empno, ename, sal*2, job, hiredate, deptno
   from emp
   where deptno = 30; --? error

create table t_emp
as select empno, ename, sal*2 sal, job, hiredate, deptno
   from emp
   where deptno = 30;   

create table t_emp (empid, ename, sal, jobid, hiredate, deptid)
as select empno, ename, sal*2  , job, hiredate, deptno
   from emp
   where deptno = 30;   

create table  t_dept (
deptno  number(2)  ,
dname  varchar2(20),
loc     varchar2(20)  default  '서울'
);
desc t_dept
select * from t_dept;

insert into t_dept (deptno, dname) values (10, 'IT'); --O
insert into t_dept  values (20, 'Web', 'Incheon'); --O
insert into t_dept  values (30, 'Mobile', null);  -- O
insert into t_dept  values (40, 'cloud', default); --O
select * from t_dept;

drop table t_dept purge;

--not null 제약조건, 컬럼 레벨에서만 정의할 수 있습니다.
create table t_dept (
deptno  number(2)    not null,
dname  varchar2(20) constraint  tdept_nn  not null,
loc       varchar2(20)  
);

select constraint_name, constraint_type 
from user_constraints
where table_name = 'T_DEPT';  --SYS_CXXXXXX,  TDEPT_NN, C

insert into t_dept (deptno, dname) values (10, 'IT'); 
insert into t_dept ( dname, loc) values ('Web', 'Incheon');  --error
insert into t_dept  values (null, 'Mobile', 'Seoul'); --error

Q> 테이블을 삭제하면 제약조건은?  
drop table t_dept purge;
select constraint_name, constraint_type 
from user_constraints
where table_name = 'T_DEPT'; 
-- 제약조건은 테이블에 종속된 객체이므로 함께 삭제됨

primary key :  not null + unique
               테이블 당 1개만 정의할 수 있음
               
create table t_dept (
deptno  number(2)  primary key, 
dname  varchar2(20) ,
loc       varchar2(20)  
);

select constraint_name, constraint_type 
from user_constraints
where table_name = 'T_DEPT'; 

insert into t_dept (deptno, dname) values (10, 'IT'); 
insert into t_dept ( dname, loc) values ('Web', 'Incheon'); --error
insert into t_dept  values (10, 'Mobile', 'Seoul'); --error

select index_name, uniqueness
from user_indexes
where table_name = 'T_DEPT'; 
--oracle server가 PK가 선언된 컬럼값들에 대해서  unique 인덱스 자동 생성

alter table t_dept disable constraint sys_c000xxx;
--정합성 체크가 비활성화됨, index drop
alter table t_dept ensable constraint sys_c000xxx;
--정합성 체크가 활성화됨, index 자동 생성됨

drop table t_dept purge; 
select index_name, uniqueness
from user_indexes
where table_name = 'T_DEPT'; 
--인덱스도 테이블에 종속적인 객체이므로 삭제됨

uniuqe는 중복된 값을 허용하지 않음(null은 unique한 값으로 평가함)
create table t_dept (
deptno  number(2)  constraint  tdept_uk   unique, 
dname  varchar2(20) ,
loc       varchar2(20)  
);

insert into t_dept (deptno, dname) values (10, 'IT');   --
insert into t_dept ( dname, loc) values ('Web', 'Incheon'); 
insert into t_dept  values (10, 'Mobile', 'Seoul');   --error
insert into t_dept ( dname, loc) values ('BigData', 'Ulsan'); 

select constraint_name, constraint_type 
from user_constraints
where table_name = 'T_DEPT'; 
--oracle server가  unique제약조건이 선언된  컬럼값들에 대해서  unique 인덱스 자동 생성

select index_name, uniqueness
from user_indexes
where table_name = 'T_DEPT'; 

drop table t_dept purge; 

# check 제약조건 : 컬럼값에 대한 범위, 허용되는 값등을 설정 제약조건
예) 성별 컬럼에  F, M값 허용
   price 컬럼에 0보다 큰 값  또는 0~100000범위의 값만 허용

create table users (
userid   varchar2(15),
username    varchar2(20),
gender      char   check  (gender in ('F', 'M')),
point     number(5)  constraint users_ck  check ( point between 0 and 99999)
);

insert into users values ('guest', '손님', 'f', 0);   ---error
insert into users values ('guest', '손님', 'F', 0);   
insert into users values ('guest1', '손님1', 'M', 99999);   
insert into users values ('guest1', '손님1', null, 99999); 
insert into users values ('guest2', '손님2', null,  null); 

select constraint_name, constraint_type 
from user_constraints
where table_name = 'USERS'; 
```

## 오후

```sql

create table  parent (
code   number(4),
category   varchar2(20)
);

insert into parent values (1000, 'Book');
insert into parent values (2000, 'Movie');
insert into parent values (3000, 'Music');
insert into parent values (4000, 'Game');

create table child (
prodid   varchar2(6)  ,
prodname   varchar2(30),
price     number(8, 2), 
code   number(4)  constraint child_fk   references parent(code)
);  --error
--FK가 참조하는 컬럼은 PK 또는 Unique 여야 함

alter table  parent add constraint parent_pk  primary key (code);

create table child (
prodid   varchar2(6)  ,
prodname   varchar2(30),
price     number(8, 2), 
code   number(4)  constraint child_fk   references parent(code)
); 

insert into child values ('B00001', 'Oracle', 10000, 1000);    
insert into child values ('B00002', 'Java', 15000,  1000);
insert into child values ('G00001', '테트리스', 3000, 4000);
insert into child values ('M00001', '오징어게임', 12000, 5000); --error
insert into child values('G00002', '카트라이더', 3000, null);
update child set code = 5000
where prodid = 'G00002';   ---error
delete from parent where code = 1000;   ---error
delete from parent where code = 2000; 

alter table 테이블명  add (컬럼 컬럼타입 제약조건,....);
alter table 테이블명  modify  (컬럼 컬럼타입 not null, ...); --컬럼타입, size, not null 제약조건 추가
alter table 테이블명 add constraint 제약조건이름 ;  --PK, unique, check, fk 제약조건 추가
 alter table 테이블명 drop (컬럼명 ,...)
alter table 테이블명 drop column 컬럼명;
alter table 테이블명 enable constraint  제약조건이름;
alter table 테이블명 disable constraint 제약조건이름;
alter table 테이블명 drop constraint 제약조건이름;
rename  old__table이름   to   new__table이름;
alter table 테이블명 rename column   old_name   to   new_name;

drop table 테이블명;  --이름이 변경되고 recyclebin
drop table 테이블명 purge; --oracle전용, --구조, 데이터 물리적으로 완전 삭제

select * from emp;
drop table emp;

desc emp    ---조회 X
select * from emp;  ---조회 X

select * from recyclebin;

select * from recyclebin;
select * from "BIN$AWp0UWVwTDGAfSMBmsP17Q==$0";

flashback table  emp to before drop;
select * from recyclebin;

desc emp  
select * from emp; 
```

## View

```sql
VIEW  - 데이터보안, 가상 데이터 제공, 간결한 SELECT문 사용.
Simple View - 1개의 테이블로부터 가상컬럼 없이,  필수 컬럼을 모두 포함하고, 표현식 없는 select문으로 정의됨 , DML 이 가능
Complex View  - 1개이상의 테이블로부터 가상컬럼(rowid, rownum, level), 조인 , 그룹핑,  함수표현식이 포함되어 정의한 View , DML 불가

create view emp20_vu
as  select * from emp 
    where deptno = 20;

desc emp20_vu
select * from emp20_vu;

select view_name, text
from user_views;

insert into emp20_vu (empno, ename, sal, deptno)
values (8000, 'Han', 2000, 40); 

select * from emp20_vu;
select empno, ename, sal, deptno from emp;

create or replace view emp20_vu
as  select   empno, ename, sal, deptno  from emp 
    where deptno = 20 with check option;

insert into emp20_vu (empno, ename, sal, deptno)
values (8001, 'Han2', 2000, 40); ---error

insert into emp20_vu (empno, ename, sal, deptno)
values (8001, 'Han2', 2000, 20); ---O

select * from emp20_vu;

create or replace view emp20_vu
as  select   empno, ename, sal, deptno  from emp 
    where deptno = 20 with read only;   ---dml불가

insert into emp20_vu (empno, ename, sal, deptno)
values (8002, 'Han3', 3000, 20); ---error

delete from emp20_vu where empno = 8001;  ---error

select * from emp20_vu;

create or replace view emp20_vu
as  select   empno, ename, sal, deptno  from emp 
    where deptno = 20;

Q>view를 삭제하면 base table은?
drop view emp20_vu ; 
select * from emp;
--view의 메타정보만 data dictionary로부터 삭제됨

create or replace view emp20_vu
as  select   empno, ename, sal, deptno  from emp 
    where deptno = 20;

select * from emp20_vu ; 
select * from emp;

drop table emp purge;
desc emp
select * from emp;

desc emp20_vu

select object_name, status , object_type
from user_objects
where object_type='VIEW';

--base가 되는 테이블이 삭제되면 view는 invalid 객체 상태로 변경됨

```

## Sequence

```sql
sequence - PK를 대신할 수 있는 unique 한 number값(정수값)을 생성해주는 객체
독립적인 객체로 여러 테이블에서 공유해서 사용할 수 있음 

drop table t_dept  purge;

create table t_dept
as select * from dept 
    where 1=2;

create sequence ~
start with ~
increment by ~ 
minvalue  ~
maxvalue  ~
cache | nocache
cycle | nocycle  ----

create sequence tdept_seq 
 increment by  10
 maxvalue 99
 nocache;
 
select * from user_sequences;
--sequence객체는 가상컬럼 currval, nextval 생성됨

select tdept_seq.currval 
from  dual;  --error
--sequence객체는 최초의 값을 발행 후에 현재 시퀀스 값을 조회할 수 있습니다

select tdept_seq.nextval from  dual; 
select tdept_seq.currval from  dual;

insert into t_dept values (tdept_seq.currval, 'IT', 'Seoul');
insert into t_dept values (tdept_seq.nextval, 'Web', 'Seoul');

select * from t_detp;

drop sequence tdept_seq;
-- 테이블에 영향을 주지 않습니다.
```

## 튜닝을 위하여

★인덱스가 사용되도록 WHERE 절을 기술할 것

```sql
[소량의 데이터를 검색하는 Query문인데...where절에 컬럼에 index가 존재하지만
index scan으로 수행 i/o 를 줄일 수 있지만, full table scan으로 수행하는 query들을 개선]

select ename
from emp
where sal*2.1 > 950 ;    ---sal컬럼에 인덱스가 존재함

select ename
from emp
where sal  > 950 / 2.1; 

select ename
from emp
where to_char(hiredate, 'yyyymmdd') = '19801217';  --hiredate컬럼에 인덱스가 존재함

select ename
from emp
where  hiredate  = to_date('19801217','yyyymmdd')

select ename
from emp
where substr(ename, 1, 1) ='S';  --ename 컬럼에 인덱스가 존재함

select ename
from emp
where  ename  like'S%';

select * from customer
where cust_id = 10021;   
---cust_id컬럼에 인덱스가 존재하고 cust_id컬럼은 문자열타입의 컬럼임

select * from customer
where cust_id = '10021'; 

Q> SMITH, Mr.SMITH 를 검색하고자 하는 경우 
select *
from emp
where ename like '%SM%';

select *
from emp
where ename like 'SM%'
or ename like 'Mr.SM%';

select ename from emp
where ename is not null;

select ename from emp
where ename >' ';

select ename from emp
where ename is null

select ename from emp
where commission is null;

select *
from 주문 
where 상태코드 != 4;

select *
from 주문 
where 상태코드 NOT IN (0, 4);
--상태코드는 접수(0), 입금완료(1), 발송준비(2), 발송완료(3), 거래완료(4), 최소(5) 값이 존재

```

## 연습 문제

```sql
Q> 고객별 총 매출액을 계산하고, 전체 고객 중 매출 순위를 출력하세요
SELECT 
    c.customer_id,
    c.customer_name,
    SUM(o.sales) AS total_sales,
    RANK() OVER (ORDER BY SUM(o.sales) DESC) AS sales_rank
FROM
    customer c
    JOIN reservation r ON c.customer_id = r.customer_id
    JOIN order_info o ON r.reserv_no = o.reserv_no
GROUP BY
    c.customer_id, c.customer_name;

Q> 고객별 최근 예약일자와 그 이전 예약일자 함께 출력하세요
SELECT
    customer_id,
    reserv_no,
    reserv_date,
    LAG(reserv_date) OVER (PARTITION BY customer_id 
			   RDER BY reserv_date ) AS prev_reserv_date
FROM reservation
ORDER BY  customer_id, reserv_date;

Q> 고객이 속한 지역의 평균 매출과 고객 개인 매출의 차이를 출력하세요
WITH customer_sales AS (SELECT c.customer_id,  c.customer_name
                              ,  c.zip_code, SUM(o.sales) AS total_sales
                        FROM customer c  JOIN reservation r 
			ON c.customer_id = r.customer_id
			JOIN order_info o 
			ON r.reserv_no = o.reserv_no
			GROUP BY c.customer_id,  c.customer_name,  c.zip_code),
   regional_sales  AS (SELECT  customer_id,  customer_name, zip_code
                             , total_sales 
      ,  AVG(total_sales) OVER (PARTITION BY zip_code) AS avg_zip_sales
			FROM customer_sales)
SELECT  customer_id, customer_name, zip_code, total_sales, avg_zip_sales
        , total_sales-avg_zip_sales as 	ales_difference
FROM regional_sales
ORDER BY zip_code, customer_id;

Q> 제품별로 판매량의 누적 합과 3건씩의 이동 평균을 출력하세요
SELECT item_id, order_no,  quantity
, SUM(quantity) OVER (PARTITION BY item_id 
                      ORDER BY order_no
                       ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW)  ) AS cumulative_quantity
, ROUND(AVG(quantity) OVER (PARTITION BY item_id 
                        ORDER BY order_no
                         ROWS BETWEEN 2 PRECEDING AND CURRENT ROW), 2) AS moving_avg_3
FROM order_info
ORDER BY item_id, order_no;

```