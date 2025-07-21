# 2강

# 복습

```
database : 통합된 Data의 집합(저장소) 
           Data 중복 최소화
           실시간 접근성, 동시성 , 공유 지원
           내용에 의한 참조하는 방식

DBMS (database management system) : 메모리와 프로세스들로 구성 (oracle instance, oracle server)
oracle, db2, mysql, postgreSQL, SQL Server 등의 dbms는 메모리와 프로세스 구성에 차이가 있음
논리적, 물리적 독립성 보장
데이터 보안 유지
장애로부터 데이터를 보호, 복구
데이터 무결성 유지

Database 운영 목적 :
운영계 - 트랜잭션 처리 목적(DML), oltp
정보계 - 데이터 분석(데이터 마이닝) 하고 의사결정에 사용, olap, dss, dw, data mart

DBMS마다 메모리 구조, 프로세스 구조, SQL 처리 과정, data 저장하는 형식 등이 모두 다르지만,SQL 표준언어를 사용해서 데이터를 조회, 수정, 추가, 삭제 할 수 있습니다

SQL 분류 :
DML - select, insert, update,  delete, merge
DDL - create, alter,  drop, truncate, comment
DCL - (권한 부여, 회수) grant, revoke
TCL - commit, rollback, savepoint

Transaction : 분리되어 수행할 수 없는 하나의 작업 단위 , Unit of work
              ACID(원자성, 일관성, 격리성, 영속성)
              DB에서 Tx단위 하나 이상의 dml로 구성되거나, 하나의 DDL(autocommit), 하나의 DCL(autocommit)

database 에 저장되는 Data 분류 :
- business data(application data, user data)
- meta data  (system catalog, data dictionary view)

database 기본 객체 :
table - entity의 저장소
        column(속성)+row(record, tuple, entity instance)
        물리적으로 data를 저장하는 기본 객체
        heap table, cluster table, IOT, partition, ...
view - table에 대한 window
       논리적 table
       table에 대한 select문으로 정의
       보안, 복잡한 sql문장을 간결하게 사용
index - 컬럼값+rowid 로 저장, B*-Tree 논리적인 저장 구조로 저장
        소량의 데이터를 빠르게 검색, I/O를 줄여줌
	sort 연산 제거        
sequence - 자동으로 정수 number값 생성기
           예) 게시판의 글번호, 주문번호   
synonym -  스키마.테이블명@dblink명 등의 객체명의 별칭으로
           이름에 간결성, 보안

data dictionary view
USER_XXXX   (소유)
ALL_XXX   (소유+권한)
DBA_XXX   (DBA권한, all)
V$_XXX   (DBA권한 , 동적 성능 정보)

C:\Users\User>sqlplus hr/oracle
SQL> desc user_tables
SQL> select table_name from  user_tables;  --12 rows
SQL> select table_name from  all_tables;    --83 rows
SQL> select table_name from  dba_tables;    --error
SQL> conn / as sysdba
SQL> select table_name from  dba_tables;     --1739 rows

sql 처리 과정 :
1. syntax check
2. semantic check (객체, 권한등을 meta data check)
   shared pool의 data dictionary cache(row cache) <----system tablespace의 data file read (추가적인 i/o 발생 할 수 있음)
3. 요청한 sql의 최적화한 실행계획이 포함된 sql context를  search
   shared pool의 library cache 
   sql context가 cache되어 있는 경우, execute -> fetch ,논리적 i/o 수행, 빠른 응답 가능
    oltp환경에서는 99% soft parsing 수행
4. hard parsing 단계
   optimizer(DBMS핵심엔진)는  객체와 시스템 통계를 기반으로 다양한 실행 경로 수집하고
   각의 수집 경로에 대한 비용(cpu+io+memory) 계산, 비용이 가장 낮은 실행계획을 선택, 실행 가능한 형태로 변환 -> server process에게 전달 -> library cache에 sql 의 최적화한 정보(sql context)를 cache (다음번에 soft parse로 수행하기 위해)
CBO (비용기반의 최적화)
5.  binding -> execute ( buffer cache , data file i/o) -> fetch

검색종류 :
projection - 1개의 target table로부터 column기준 검색, select ~ from~
selection - 1개의 target table로부터 row기준 검색 , select ~ from~ where~
join - 2개이상의 target table로부터 공통 속성(column- parent.PK와 child.FK) 값이 일치할때 table들의 row를 결합해서 검색 결과 생성

schema - db객체들(table, view, index, sequence,..)의 group
         create schema~
         user명 = schema 명 (oracle)
         객체에 대한 namespace를 제공 (객체 이름의 재사용성 제공)

select [distinct]  *  |  column list |  expression [[as] alias]
from  table명 | dual ;

컬럼타입 :
number 
char, varchar2  , CLOB
date , timestamp, timestamp with timezone,  interval year to month, interval day to second
raw, BLOB
BFILE

내장 컬럼(가상컬럼) :
rowid (objectid+data file id + block id + 행순서번호)
rownum 은 결과행에 순차적인 정수 값 반환

null의 의미 : 아직 할당되지 않은, 알수 없는,  산술, 비교, 논리 연산이 불가한
             0과 다름, ' '과 다름, ''는   null로 취급함
 
표현식의 별칭에 공백이 포함되거나, 대소문자를 구별해야 할 경우 "별칭"로 정의합니다.

alter session set 파라미터 = 값;

where절 조건에 사용되는 연산자 : 
비교연산자 : =, <>, !=, >, >=, <, <=  
범위 연산자 : 컬럼 between 하한값 and 상한값
in list 연산자 :  컬럼 in (값,.....)
character pattern matching :  like ' ', 만능문자  %와 _
null을 비교 연산자  : is null, is not null

where절에 여러 조건은 논리 연산자로 결합합니다 (not, and ,or)

heap table에 저장된 데이터를 특정 컬럼기준으로 정렬결과를 조회하려면 order by절 사용
select 	~		---3
from ~            ---1
where ~ 	 ---2
order by ~       ---4

order by 컬럼명  정렬방식(ASC | DESC), 컬럼명  정렬방식(ASC | DESC),....
order by select절에 선언되지 않은 컬럼 
order by  select절에 선언된 표현식 
order by  select절에 선언된 별칭 
order by  select절에 선언된 컬럼 순서 

DB 함수 :
[predefined function]
Single Row Function --Character, Numeric, Date, Conversion, 기타일반
Multiple Row Funcion(Group Function, Aggregation)
Window Function(Analystic Function) 

[custom funtion ]
PL/SQL로 함수 정의, 컴파일, 호출 실행

문자열 값 처리, 날짜 값 처리할때 반드시 ' '로 감싸줘야 합니다. 
날짜 값 처리할때 세션의 파라미터에 설정된 format 형식에 맞춰주면, oracle server가 자동 형변환 처리를 해 줄 수 있습니다.
```

# 2강 시작

## DATE Function 누락된 것 부터

```sql
select  next_day(sysdate, '금') , next_day(sysdate, '월')
from emp;

--to_xxxxx()는 변환함수
select sysdate+to_yminterval('2-6')
       ,sysdate+to_yminterval('0-6') 
       , sysdate+to_dsinterval('3 7:00:00')

from dual;
Q> emp 사원들이 근무 몇년 몇개월 , 전체 몇 주 를 출력하는 sql작성하시오 (절삭처리해서)
select  trunc(trunc(months_between(sysdate, hiredate)) / 12) || '년' 
        trunc(mod (months_between(sysdate, hiredate) , 12)) || '개월' "근속년월" , trunc((sysdate-hiredate)/7) "근속 주"
from emp;
```

## 변환 함수(Conversion Function)

```sql
#conversion function

자동 형변환 : 
where 컬럼 like ''
select 10||10, '10'+'10'
 from dual;

#to_char(number, 'format string') : number값과 'format string'이 일치하지 않아도 변환됨
#to_char(date, 'format string') : date값과 'format string'이 일치하지 않아도 변환됨

select 123456.789, to_char(123456.789, '$999,999,999.00'),
       sysdate, to_char(sysdate,  'YYYY"년"  MM"월" DD"일"')
from dual;

#to_number('숫자로구성된 문자열', 'format string')와  to_date('날짜형식의 값으로 구성된 문자열', 'format string')는 첫번째 인수의 형식과 일치하도록 두번째 인수의 'format string'을 선언하지 않으면 오류 발생

select  to_number('$123,456.789', '999,999.000')
        , to_date('2024년 5월 5일',  'RR/MM/DD')
from dual; --error

select  to_number('$123,456.789', '$999,999.000')
        , to_date('2024년 5월 5일',  'YYYY"년" MM"월" DD"일"')
from dual; 

#CAST (값 as 변환타입) 
select empno, CAST(empno as varchar2(5)) 
from emp;

SELECT CAST('123.45678' AS NUMBER(6,2)) FROM dual; ---?

Q> EMP 테이블에서 입사한 달의 근무 일수를 계산하여 조회하시오
  (단, 토요일과 일요일도 근무일수에 포함한다.)
SELECT empno, ename, hiredate
      , last_day(hiredate)- hiredate "work days" 
FROM emp;

Q> EMP 테이블에서  입사 일자로부터 3개월이 지난 후 돌아오는 첫번째 금요일의 날짜를 출력하시오
SELECT empno, ename, hiredate,   
       next_day(add_months(hiredate, 3), '금')     "interview day"
FROM emp;
```

## 테이블 생성

```sql
drop table orders purge;

CREATE TABLE orders (
  warranty  INTERVAL YEAR TO MONTH
 );

desc orders
select table_name from user_tables;

INSERT INTO orders VALUES ('2-6');
INSERT INTO orders VALUES ('10-0');
INSERT INTO orders VALUES ('100-0'); --error
select * from orders;

drop table task_schedule purge;

CREATE TABLE task_schedule (
    task_id        NUMBER    PRIMARY KEY,
    task_name      VARCHAR2(100),
    duration       INTERVAL DAY(3)  TO SECOND (9)
);

INSERT INTO task_schedule (task_id, task_name, duration)
VALUES (1, 'Backup Job', INTERVAL '1 02:30:15' DAY TO SECOND);

INSERT INTO task_schedule (task_id, task_name, duration)
VALUES (2, 'Data Load', INTERVAL '0 10:00:00' DAY TO SECOND);

INSERT INTO task_schedule (task_id, task_name, duration)
VALUES (3,'100미터 달리기',  INTERVAL '0 0:0:09.456' DAY TO SECOND);

select * from task_schedule;

INSERT INTO task_schedule (task_id, task_name, duration)
VALUES (4,'식물키우기',  INTERVAL '120 0:00:00' DAY TO SECOND); --error

INSERT INTO task_schedule (task_id, task_name, duration)
VALUES (5, '딸기키우기', INTERVAL '99 0:00:01.2345678' DAY TO SECOND); 

INSERT INTO task_schedule (task_id, task_name, duration)
VALUES (5, '딸기키우기', INTERVAL '99 0:00:01.2345' DAY TO SECOND);

SELECT task_id, task_name, duration
FROM task_schedule
WHERE duration > INTERVAL '2' DAY;
```

## null 처리 함수

```sql
# null 처리 함수
nvl(arg1, arg2) : arg1이 null이 아니면 arg1리턴,
		arg1이 null이면 arg2리턴
                  arg1 과 arg2의 타입이 동일
select ename, sal+comm,   sal+nvl(comm, 0)
from emp;

select ename, comm, nvl(comm, 'No Commission')
from emp; --error

select ename, comm, nvl(to_char(comm), 'No Commission')
from emp;

nvl2(arg1, arg2, arg3) : arg1이 null이 아니면 arg2리턴
			arg1이 null이면 arg3리턴
			arg2 과 arg3의 타입이 동일

Q> emp테이블에서 comm이 null이면 sal을 리턴, null이 아니면 sal+comm리턴합니다.
select ename, comm, nvl2(comm, sal+comm, sal)
from emp;

# coalesce(arg1,....argN) : 최초의 null아니 인수를 리턴하고 함수는 종료됨
                           모든 파라미터의 타입은 동일해야 함
select  coalesce(1,2,3, null, 4, 5)
        , coalesce(null, null, null, null, 4, 5)
        , coalesce(null, null, null, null, null)
from dual;

select  coalesce(null, 1, sysdate, 'A')
from dual; --error

#우선순위 값 반환할때 유용 
SELECT 고객명, coalesce(휴대폰번호, 비상연락, 집전화번호, 이메일) as 연락처
FROM 고객정보;

#nullif(arg1, arg2) : arg1과 arg2의 값이 같으면 null을 리턴
                      arg1과 arg2의 값이 같지 않으면 arg1을 리턴
                      arg1과 arg2의 타입은 동일해야 함
                      동일한 값을 구분할때, 중복된 값을  null처리 사용
select nullif(100, 200), nullif(100, 100)
from dual; 

select nullif(100, '백') 
from dual;  --error
```

## 조건

```sql

# if~ then ~else 로직의 조건처리 함수 decode, SQL3 표준 표현식
decode (expression, value1, R1, value2, R2, value3, R3, ....ValueN-1, RN-1, RN)

select decode(10+1, 9, '정답1', 10, '정답2', '정답없음')
from dual;

select decode(10+1, 9, '정답1', 10, '정답2', 11, '정답3', '정답없음')
from dual;

문>  emp테이블에서
     30번 부서사원은 급여를 5% 인상
     20번 부서사원은 급여를 15% 인상
     10번 부서사원은 급여를 10% 인상된 급여를 현재 급여와 함께 출력 
(이름, 부서번호, 급여, 인상된 급여)
select deptno, ename, sal 
       , decode(deptno, 10, sal*1.1 
                        20, sal*1.15,
                        30, sal*1.05, sal)  "Increase"
from emp
order by deptno;

#SQL1999 조건 처리 표준 표현식
case  표현식  when  비교값1 then 리턴값1
            [ when 비교값2 then 리턴값2
	    when  비교값3 then 리턴값3
              ....
            else 리턴값]  

select case 10+1 when 9 then '정답1'
                 when  10 then '정답2'
                 else '정답없음' end
from dual;

select deptno, ename, sal 
       , case deptno when 10 then sal*1.1 
                     when 20 then sal*1.15 
                     when 30 then sal*1.05
                     else  sal end  "Increase"
from emp
order by deptno;

case when 조건표현식1 then 리턴값1
     when 조건표현식2 then 리턴값2
      ....
     else 리턴값 end

select deptno, ename, sal 
       , case  when deptno=10 then sal*1.1 
               when deptno=20 then sal*1.15 
               when deptno=30 then sal*1.05
               else  sal end  "Increase"
from emp
order by deptno;

Q> 사원번호, 이름 , 급여, 급여의 세금을 출력하시오
급여가 1000미만이면 0
급여가 1000이상~2000미만이면 급여의 3%
급여가 2000이상~3000미만이면 급여의 7%
급여가 3000이상~4000미만이면 급여의 12%
급여가 4000이상이면 급여의 15% 

select deptno, ename, sal,
        case  when sal < 1000 then 0
              when sal < 2000 then sal*0.03
              when sal < 3000 then sal*0.07 
              when sal < 4000 then sal*0.12
              else sal*0.15 end "TAX"
from emp;

select deptno, ename, sal,
       decode ( trunc(sal/1000) , 0 , 0
                                , 1, sal*0.03
                                , 2, sal*0.07 
                                , 3, sal*0.12
                                 , sal*0.15)  "TAX"
from emp;

select  greatest(4, 99, 6, 7, 9), LEAST(4, 99, -6, 7, 9)
from dual;

select  greatest('Z', 'S', 'A', 'H'), LEAST('Z', 'S', 'A', 'H')
from dual;

select  greatest('Z', 'S', 100, 50), LEAST('Z', 'S', 100, 50)
from dual;
```

# 그룹함수

```sql
복수행 함수 + 데이터 그룹핑
count([all|distinct] * | expression), 
sum([all|distinct] number) , avg, min, max, variance, stddev
※ 그룹함수의 파라미터 컬럼에  null이 포함된 경우, ignore합니다.

select count(*), count(comm), count(deptno), count(distinct deptno), count(null)
from emp;
--count(*)는  테이블에 not null제약조건이 선언된 컬럼 또는 primary key컬럼을 대상으로 컬럼값의 개수를 반환 (테이블의 행수) 
--존재하는지 여부를 체크할 때 사용할 경우, 사용하지 않도록 합니다.(성능 나쁨)

#sum, avg, variance, stddev 함수 number타입만
select sum(sal), avg(sal), variance(sal), stddev(sal), sum(null), avg(null)
from emp;

select avg(comm), sum(comm)/count(*)
from emp; 
-- 커미션을 받는 사원들의 커미션 평균?, 전체 사원들의 커미션 평균?

select avg(nvl(comm, 0)), sum(comm)/count(*)
from emp; 
※ avg함수에 전달되는 컬럼에 null이 포함된 경우  null처리한 후에 평균 연산 적용되도록 
# min(expression), max(expression) , min(distinct expression), max(distinct expression)
select min(ename), max(ename), min(hiredate), max(hiredate), min(sal), max(sal)
from emp;

select min(null), max(null)
from emp;

```

## Group by

```sql
select ~  표현식          -----4
from  ~                 -----1
[where  filter 조건 ]    -----2
[group  by 컬럼(표현식), ...]  ---3
[order by 컬럼 정렬방식]       ----5

Q> 부서별 급여의 합계, 평균을 조회하시오
select ename, deptno, sal, sum(sal), avg(sal)
from emp
group by deptno;  --error
-- 집계 함수와 함께 선언되는 개별 컬럼은 반드시 group  by절에 선언해야 합니다

select deptno, sum(sal), avg(sal)
from emp
group by deptno;   --hash 연산
-- group  by절에 선언된 컬럼은 그룹함수 없이 select절에 선언될 수 있습니다.(선언되지 않아도 됨)
-- group by 연산은 hash, sort 연산으로 처리됩니다.

select deptno, sum(sal), avg(sal)
from emp
group by deptno
order by deptno;  -- group by연산이 hash 연산으로 처리됩니다.

select sum(sal), avg(sal)
from emp
group by deptno; ---?
--group  by절에 선언된 컬럼은 select절에 선언하지 않아도됩니다.

Q> 부서와 직무로 그룹핑한 행들의  급여의 합계, 평균을 조회 결과로 생성하는 Query를 작성하시오
select  deptno, job,  sum(sal), avg(sal)
from emp
group by deptno, job;
--group by절에는 별칭 X, 컬럼 position X

Q> EMPLOYEES 테이블에서 부서별 소속 사원 수가 4명보다 많은 부서만 부서번호(department_id), 인원수, 급여(salary)의 합을 출력하는 SQL을 작성하시오 (사원수가 많은 부서부터 출력)

select ~  그룹함수 표현식   -----5
from  ~                 -----1
[where  filter 조건 ]    -----2
[group  by 컬럼(표현식), ...]  ---3
[having 그룹함수 조건]        ---4
[order by 컬럼 정렬방식]       ----6

select department_id, count(*) "인원수", sum(salary) "급여합계"
from EMPLOYEES
group by department_id
having count(department_id) > 4;

Q> employees테이블에서 관리자(manager_id)가 없는 사원은 제외하고, 
사원들의 부서(department_id)별 평균급여(salary)가 6000미만인 부서번호와 평균급여를 출력하시오 
단, 평균급여의 내림차순으로 출력하시오
select  department_id, avg(salary) 
from  employees
where manager_id is not null
group by  department_id 
having avg(salary) < 6000   
order by 2 desc;

Q> employees테이블에서 부서별 급여 평균중에서 최고 평균 급여를 검색하는 SQL문 작성하시오
---그룹함수는 nested 해서 사용할 수 있다
select max(avg(salary))
from  employees 
group by  department_id ;
-- 여기에다가 select에 department_id 쓰면 안됨. 갯수가 안맞다

#listagg (컬럼, 구분자) within group (order by 컬럼)
Q> 부서별로 사원들의 이름을 이름의 오름차순으로  , 로 연결해서 단일 문자열로 출력하는 SQL을 작성합니다.
select  deptno,  listagg (ename,', ') within group ( order by ename )
from emp
group by deptno;

select  deptno,  listagg (ename,', ') within group ( order by sal desc )
from emp
group by deptno;

```

### ★집계 함수와 함께 선언되는 개별 컬럼은 반드시 group  by절에 선언해야 합니다

| 주석 내용 | 실제 의미/정정 |
| --- | --- |
| 집계 함수와 함께 쓰려면 컬럼은 반드시 GROUP BY에 있어야 한다 | ✅ 맞음 |
| GROUP BY 컬럼은 SELECT절에 단독으로 쓸 수 있다 | ✅ 맞음 |
| GROUP BY는 hash 연산으로 처리된다 | ⛔ 항상은 아님. 데이터/옵티마이저에 따라 다름 |
| GROUP BY 컬럼은 SELECT절에 안 써도 된다 | ✅ 문법상 가능. ❗하지만 실용성 없음 |

## 문제 풀이

```sql
10.  Employees로부터 전체 사원수, 2002, 2003, 2004, 2005년도에 입사한 사원수를 출력하시오
컬럼 타이틀은 total, 2002, 2003, 2004, 2005로 출력하시오
select  count(*) total
      , sum( decode( to_char( hire_date, 'YYYY'), '2002', 1)) as "2002"
      , count( case extract(year from hire_date) when 2003 then 1 end) as "2003"
from employees ;

11.Employees로부터 직무별로 월급의 합계와   각 부서내에 직무별 월급의 합계를 아래 보기와 같이 출력하시오   컬럼 타이틀은 Job, Dept 20, Dept 50, Dept 80, Dept 90로 출력하시오
select job_id  "Job" , sum(decode(department_id, 20, salary)) "Dept 20"
       , sum(decode(department_id, 50, salary)) "Dept 50"
       , sum(decode(department_id, 80, salary)) "Dept 80"
       , sum(decode(department_id, 90, salary)) "Dept 90"
        , sum(salary) "Total"
from employees
group by job_id;
```

### 이런 걸 pivot이라고 하는데(테이블로 보여주는거..) case 써야 되는듯?

# Join

```sql
join 종류 : 
inner join, equi join
cartesian product, cross join - 모든 행이 조인할 테이블의 모든 행과 조인
non-equi join
outer join - 조인 컬럼값이 null인 경우, 조인 결과 집합에 포함시키키 위한 조인
self join  - 하나의 테이블로부터 행들간에 조인 

join 문법 : 
1. where 조인 조건
2. from절에 조인 조건을 선언 (SQL1999 표준)

Q> emp 사원들에 대해서 사번, 이름, 부서번호, 부서이름을 조회 (emp, dept)
--ERD를 통해서 조인 테이블의 관계 확인
--데이터 모델 설계시에 참조 관계의 테이블에 동일속성이 존재 (동일한 컬럼명, 동일한 타입)
--90%이상의 조인은 inner join, equi join (parent.pk = child.fk)
--parent(dept) : child(emp) = 1:M
--다른 필터 조건이 없으면 equi join의 결과 행수는  최대 M쪽 테이블의 행수입니다.
select  empno, ename,  deptno, dname
from emp, dept;   ---? error ( 동일속성 컬럼의 소유자를 선언하지 않아서...)

delete from dept where deptno > 40;

select  emp.empno, emp.ename,  emp.deptno, dept.dname
from emp, dept;  ---? 14rows X 4rows , cartesian product, cross join
# 조인 조건 선언 X

select  a.empno, a.ename,  a.deptno, b.dname
from emp   a  , dept   b
where  a.deptno = b.deptno;  

select  a.empno, a.ename,  a.deptno, b.dname
from  emp   a  natural join  dept   b ;  --error
--natural join은 oracle server가 조인할 두 테이블에서 이름이 같은 컬럼들의 컬럼값이 일치할때 조인 수행

select  a.empno, a.ename,  deptno, b.dname
from  emp   a  natural join  dept   b ; 
--natural join은 동일이름의 조인키 컬럼에 소유자 테이블명, 별칭을 선언하지 않습니다

Q> employees(107rows)와 departments(28rows)에서 사원들에 대해서 사번(employee_id), 이름(last_name), 부서번호(department_id), 부서이름(department_name)을 조회
select  a.employee_id, a.last_name,  department_id, b.department_name
from employees a natural join departments b;  --32rows

select  a.employee_id, a.last_name,  a.department_id, b.department_name
from employees a  , departments b
where a.department_id = b.department_id
and a.manager_id = b.manager_id;

select  a.employee_id, a.last_name,  department_id, b.department_name
from employees a   join departments b using (department_id);  --106rows
-- join ~using 절은 동일이름의 컬럼 1개로만 조인할 때
--join ~using 절은 동일이름의 조인키 컬럼에 소유자 테이블명, 별칭을 선언하지 않습니다

select  a.employee_id, a.last_name,  a.department_id, b.department_name
from employees a  join departments b  on a.department_id = b.department_id ;  ----106rows

Q> new_emp 사원들에 대해서 사번, 이름, 부서번호, 부서이름을 조회 (new_emp, dept)
--new_emp와 dept 조인할 때 조인 조건 컬럼에 이름이 다르므로 natural join, join ~using로 조인할 수 없습니다. 
--where절 조인조건 선언 또는  join ~ on 조인조건 사용해서 조인 해야 합니다.

select a.*, b.*
from new_emp a join dept b   on a.deptid = b.deptno;

drop table new_emp purge;

```

## 문제 풀이

```sql
Q> customer 테이블에서 zip_code 별로 고객 수, 출생년도의 평균, 최근 가입일을 구하시오.
select zip_code, count(*) 
       , AVG(TO_NUMBER(SUBSTR(birth, 1, 4))) AS avg_birth_year
       , MAX(first_reg_date) AS latest_join_date
from customer
GROUP BY zip_code;

Q> customer 테이블에서 각 직업(job)별로 고객 이름(customer_name)을 ,(콤마)로 연결해서 출력하시오.
SELECT job,
      LISTAGG(customer_name, ', ') WITHIN GROUP (ORDER BY customer_name) AS customers
from customer
GROUP BY job;

Q> customer 테이블에서 성별(sex_code)별로 2017년 11월에 가입한 고객 수를 구하시오.
SELECT sex_code,
       COUNT(CASE WHEN first_reg_date >= TO_DATE('2017-11-01', 'YYYY-MM-DD') AND first_reg_date <= TO_DATE('2017-11-30', 'YYYY-MM-DD') THEN 1 END) as recent_customers
FROM customer
GROUP BY sex_code;

SELECT sex_code,
       COUNT(CASE WHEN first_reg_date like '2017/11%'  THEN 1 END) AS recent_customers
FROM customer
GROUP BY sex_code;

SELECT sex_code,
       COUNT(*) AS recent_customers
FROM customer
WHERE first_reg_date like '2017/11%'
GROUP BY sex_code;

Q> customer 테이블에서 '자영업' 직업을 가진 사람 중 성별에 따라 고객 이름을 연결해서 출력하시오
SELECT sex_code,
       LISTAGG(customer_name, ', ') WITHIN GROUP (ORDER BY customer_name)
FROM customer
WHERE job = '자영업'
GROUP BY sex_code;

Q> reservation 테이블에서 지점(branch)별 총 방문자 수가 10명 이상인 지점만 조회하시오.
SELECT branch, SUM(visitor_cnt) AS total_visitors
FROM reservation
GROUP BY branch
HAVING SUM(visitor_cnt) >= 10; --count(visitor_cnt) >= 10

Q> reservation 테이블에서 예약을 가장 많이 한 고객의 ID와 예약 수를 출력하시오.
select customer, reserves "예약수"
from (select customer_id, count(*) reserves
     FROM reservation
     GROUP BY  customer_id
     order by 2 desc )
where rownum = 1

Q> reservation 테이블에서 고객별로  7~12 월별 예약 건수를 피벗 형태로 출력하시오.
SELECT customer_id, 
       count(decode(SUBSTR(reserv_date, 5, 2), 07, 1)) "07 reserv"
         , count(decode(SUBSTR(reserv_date, 5, 2), 08, 1)) "08 reserv"
         , count(decode(SUBSTR(reserv_date, 5, 2), 09, 1)) "09 reserv"
         , count(decode(SUBSTR(reserv_date, 5, 2), 10, 1)) "10 reserv"
         , count(decode(SUBSTR(reserv_date, 5, 2), 11, 1)) "11 reserv"
         , count(decode(SUBSTR(reserv_date, 5, 2), 12, 1)) "12 reserv"
FROM reservation
group by customer_id
Q> reservation 테이블에서 고객별로 전체 예약 건수, 취소 건수, 취소율(%)을 출력하시오.
```

## Join 2(Non-Equi Join)

```sql
desc salgrade
desc emp
--동일이름컬럼 X, 동일값의 속성 X
Q> emp의 사원들의 급여와 급여의 등급(salgrade테이블의 grade)을 조회 결과로 생성
--non-equi join
select a.empno, a.ename, a.sal, b.grade
from emp a, salgrade b
where a.sal between b.losal and b.hisal;

select a.empno, a.ename, a.sal, b.grade
from emp a  join salgrade b  on a.sal between b.losal and b.hisal;

Q> employees, locations, departments을 조인해서
사원이름, 소속부서이름, 부서가 위치한 city를 출력합니다.
--n개의 테이블을 조인할 때 최소 선언해야 하는 조인조건은 n-1개의 조건을 선언해야 합니다.
select  a.last_name, b.department_name, c.city
from  locations c ,  departments b,  employees a,
where b.location_id = c.location_id 
and a.department_id = b.department_id;  ----16Rows?

select  a.last_name, b.department_name, c.city
from  locations c join  departments b on b.location_id = c.location_id
   join   employees a on a.department_id = b.department_id; 
   
--하나의 테이블에서 PK를 참조하는 FK가 정의된 경우 : 자기 참조 테이블
--자기 참조 테이블에서는 self join이 가능합니다
Q> 사원번호, 사원이름, 관리자번호, 관리자이름 (emp)
select e.empno, e.ename, e.mgr,  m.ename
from  emp e , emp m
where e.mgr = m.empno;

select e.empno, e.ename, e.mgr,  m.ename
from  emp e join  emp m  on e.mgr = m.empno;

employees(107rows) 11개부서..
departments(28rows) 28개 부서 
--조인 조건이 일치하지 않는 경우 inner join (equi join) 결과에서는 포함되지 못함
--outer join은 join 키 컬럼 값이 null인 레코드를 결과 집합에 포함하기 위한 조인
--where절의  조인될 레코드가 없는 조인 조건에 (+) 선언합니다.
SQL 1999문법은  from절의 기준 테이블에 right outer join, left outer join 키워드로 선언합니다.

insert into emp (empno, ename, sal) values (7000, 'kim', 3000);
commit;

Q>부서를 아직 배정받지 못한 사원을 포함해서 사원들의 부서번호와 부서이름을 출력 (emp, dept)
select  a.*, b.dname
from  emp a, dept b
where  a.deptno = b.deptno; --- 7000('kim') 사원 정보 누락됨

select  a.*, b.dname
from  emp a, dept b
where  a.deptno = b.deptno(+); 

select  a.*, b.dname
from  emp a left outer join  dept b  on  a.deptno = b.deptno;

Q>부서별 사원정보를 출력 (사원이 없는 부서정보 40번도 결과집합에 포함)
select  b.deptno,  b.dname, a.*
from  emp a, dept b
where  a.deptno(+) = b.deptno
order by b.deptno ;

select  b.deptno,  b.dname, a.*
from  emp a  right outer join  dept b on a.deptno = b.deptno
order by b.deptno ;

Q>부서를 아직 배정받지 못한 사원을 포함하고, 사원이 없는 부서정보도 결과집합에 포함해서 조인 결과집합을 생성하는 SQL 구현
select  b.deptno,  b.dname, a.*
from  emp a, dept b
where  a.deptno(+) = b.deptno(+)
order by b.deptno ;

select  b.deptno,  b.dname, a.*
from  emp a  full outer join  dept b on a.deptno = b.deptno
order by b.deptno ;

select  emp.empno, emp.ename,  emp.deptno, dept.dname
from emp, dept; 

select  emp.empno, emp.ename,  emp.deptno, dept.dname
from emp cross join dept; 

```