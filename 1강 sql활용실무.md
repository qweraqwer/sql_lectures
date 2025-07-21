# 1강

# 박기연 강사님

# Introduction

1. 물리적
2. 논리적

raw data → program, process → service, information

              ↔ File data(write, read) → 공유 x → 중복(prgram 별로 data 저장) → 정확성(동기화해야함), 일관성

물리적 File

논리적 저장소 Database = (논리적으로 가장 큰) Data들의 저장소

였는데, Big data 이후에 ‘논리적’은 사라짐

여러 App들이 하나의 DataBase에 접근하는 것을 hot data라고 한다.

→ 그런데 사람이 일일이 나눠 줄 수 없으니 program 필요

→ queue에 쌓아두고 나눠줘야 함, 보안/장애 복구도 필요

그래서 필요한 것이 DBMS(Process + Memory) 필요. 

DBA(데이터 베이스 관리자)

Data : 속성(Record)들에 대한 집합

Data record와 참조 관계

⇒ 계층형 DBMS 1:M(물리적 pointer 라서 부모 삭제되면 대거 수정 필요)

⇒ N : M Network

⇒ (1970 논문 : 수학 관계 대수로 참조 관계 무결성 보장) Data 집합 2차원 구조 → table

⇒ Oralce RDBMS(정형 Data 저장, table), IBM DB 2

⇒ 1980 객체 지향 언어 → 가변적 구조

⇒ 1990 ODBMS (무결성이 효율적이지 않았으나..) → ORDBMS

최근 오라클만큼 성능이 좋은 PostgreSql?

1990 : Internet의 등장 → 전자 상거래?

⇒ data 증가

⇒ 2010 Smart Phone → 데이터 더 증가

⇒ Big Data (File을 Cluster로), NoSQL - Not Only SQL

Memory+Process+Database(data protocol, 타입, 형식)이 달라도 sql로 대통합이었는데..

→ NoSQL는 program API로

⇒ ORDBMS : OLTP(OnLine Transaction Processing , 트랜잭션의 목적이 강하다), NoSQL : OLAP(OnLine Analytical Processing), DSS, DW(주로 읽기, 분석 목적)

PostgreSQL RBDMS + BigData NoSQL

# 데이터 베이스

- 필요로 하는 데이터를 일정한 형태로 저장

⇒ 논리적 table = Row + column ⇒

### View → 1. Window 보안 2. 복잡한 Select 간결하게

Database = Table + Views + Indexes + Other Objects + SYSTEM CATALOG

Index : 검색 성능, SORT 연산 제거

데이터의 두 가지 종류

1. Application Data, Business Data, User Data
2. DBMS → DB 상태, 성능 이라는 Data →Meta Data(유저의 권한 등등) → Data Dictionary, SYSTEM Catalog라고 부른다

ORDBS

정형 Data → RDBS

반정형 Data → ORDBS

비정형 Data (외부 ,파일) → Read only

NoSQL → 반정형, 비정형

NoSQL도 sql 기반이라고? → 그렇다고 합니다.(잘 알아두면 전이성이 크다)

# SQL

Structured Query Language

- 선언적 언어, 결과지향적 언어 ↔ 과정 기준 : 절차적 언어(프로그래밍 언어)
- 과정 기술 못함, 예외 처리 못함

- DDL : Data Definition Language - table, view 등 만들고 삭제?
- DML : Data Manipulation Language - select, insert 등
- DCL : Data Control Language - Data 보안 1) 인증 2) 권한인데, DCL은 권한을 제어함(grant, revoke)
- TCL : Transaction Control Language - transaction : “Unit of Work” ex. 온라인 구매 - DML들로 구성된 하나의 Transaction, COMMIT 등

# System Catalog

Data dictionary : 메타 데이터

View이다??

# Schema (중요)

table, index, view → schema(논리적 데이터 객체의 묶음)

→ Oracle은 유저 이름 = Schema

DB2는 아니다.

Table, Sequence, View, Index

Sequence : 고유한 숫자를 생성하는 객체 → primary key : unique + not null

PL/SQL

Stored Procedure, Function, Trigger, ★Constraint(데이터 무결성을 보장하는 규칙)

# 데이터 모델(data model)

→ 숙련자 용

# 오라클 설치

비번 : oracle

```sql
Window 로고 + R 
실행> cmd
C:\Users\User>sqlplus /nolog -- sqlplus 툴만 켜고 접속은 하지 않음

SQL> conn / as sysdba -- 최고 관리자 권한인 sysdba로 접속

SQL> alter user hr -- hr 사용자 설정을 바꾸겠다
  2  identified by oracle -- 비밀번호를 oracle로 변경
  3  account unlock; -- 계정 잠금 해제

SQL> conn hr/oracle -- hr 계정으로 접속
```

## *참고(반드시 알아야할 것)

Database, DBMS - table, view, index

→ User Data / Business Data

→ Meta DAta

SQL - DML, DDL, DCL, TCL

schema 

Transaction : Unit of Work, ACID(원자성, 일관성, 격리적, 고립성, 영속성)

Table : Row(Record, Tuple, Entity Instance) + Column(속성)

Table - Entity, Relation??

table 내에 한개의 primary key column 있어야 함(unique + not null)

# 테이블

사원(child),            부서(parent)

부서 번호 : fk         번호 : pk

(참조 컬럼의 값, null 허용)

참조 무결성(Refrential Integrity) 제약 조건?? → 참조하는 값이 부모 테이블(pk)에 반드시 존재해야함

```sql
select tname from tab; -- 테이블 목록 조회,dept, dmp가 없으면?
@c:\scott.sql -- sql 명령어들을저장한 스트립트 파일(.sql) -- 그전에 scott c로 옮겨야

describe emp
desc dept
-- desc 또는 describe 는 tool 명령어입니다.
```

# Sql developer

oracle에서 제공하는 기본 툴

select, insert, from, where → 키워드, 키워드는 대소문자 구분 X

테이블명, 컬럼명 대소문자 구분 X

컬럼값은 대소문자 구분합니다.

select, insert, from, where, … 키워드는 테이블명으로 사용할 수 없음.

### 접속

host

HW → OS → DBMS(Instance 이름, DB명) → sqlplus tool로 local connect, 인증(IPC)

toad, 오렌지 같은 것도 있음

romote host (Toad, SQL Developer) → user, password, host port, service 명

⇒ 2 tier 구조

DBMS를 이용하는데 중간에 WAS가 있는 경우가 있음

Client → Was → DBMS

3Tier 구조

```sql

C:\Users\User>services.msc -- 여기서 oracle service 확인 가능

C:\Users\User>lsnrctl status

LSNRCTL for 64-bit Windows: Version 11.2.0.2.0 - Production on 07-7월 -2025 10:59:30

Copyright (c) 1991, 2014, Oracle.  All rights reserved.

Connecting to (DESCRIPTION=(ADDRESS=(PROTOCOL=IPC)(KEY=EXTPROC1)))
STATUS of the LISTENER
------------------------
Alias                     LISTENER
Version                   TNSLSNR for 64-bit Windows: Version 11.2.0.2.0 - Production
Start Date                07-7월 -2025 09:58:50
Uptime                    0 days 1 hr. 0 min. 39 sec
Trace Level               off
Security                  ON: Local OS Authentication
SNMP                      OFF
Default Service           XE
Listener Parameter File   C:\oraclexe\app\oracle\product\11.2.0\server\network\admin\listener.ora
Listener Log File         C:\oraclexe\app\oracle\diag\tnslsnr\DESKTOP-5FBE16T\listener\alert\log.xml
Listening Endpoints Summary...
  (DESCRIPTION=(ADDRESS=(PROTOCOL=ipc)(PIPENAME=\\.\pipe\EXTPROC1ipc)))
  (DESCRIPTION=(ADDRESS=(PROTOCOL=tcp)(HOST=DESKTOP-5FBE16T)(PORT=1521)))
  (DESCRIPTION=(ADDRESS=(PROTOCOL=tcp)(HOST=DESKTOP-5FBE16T)(PORT=8080))(Presentation=HTTP)(Session=RAW))
Services Summary...
Service "CLRExtProc" has 1 instance(s).
  Instance "CLRExtProc", status UNKNOWN, has 1 handler(s) for this service...
Service "PLSExtProc" has 1 instance(s).
  Instance "PLSExtProc", status UNKNOWN, has 1 handler(s) for this service...
Service "XEXDB" has 1 instance(s).
  Instance "xe", status READY, has 1 handler(s) for this service...
Service "xe" has 1 instance(s).
  Instance "xe", status READY, has 1 handler(s) for this service...
The command completed successfully

```

## 인스턴스?

**인스턴스 = 메모리(SGA, System Global Area) + 백그라운드 프로세스들의 집합**

**하드디스크에 저장된 데이터베이스를 실제로 "실행"하는 엔진**

→ **"인스턴스"는 'DB가 살아 있는 상태'**라고 이해하면 정확합니다.

| 개념 | 의미 |
| --- | --- |
| **Database** | 물리적인 데이터 파일들의 집합 |
| **Instance** | DB를 **실행 중인 상태** (메모리 + 프로세스) |
| ✅ 그래서 "Instance"라고 부름 | → DB가 살아서 움직이게 만드는 **동작 단위**이기 때문입니다 |

Oracle DBMS (Instance --)

Memory 등..

Listener가 Client(Tool)로 부터  connect될 때, host, port, service를 받음

Listener가 전달 해주고 Oracle server에서 1. 인증 2. Server Process(SQL 처리) PGA 생성 3. Session(정보) tool

### PC → 사용자 프로세스 → tnsname.ora → 서버 프로세스 (연결 되면 중간 tnsname.ora 안 거침)

- Oracle Client에게 사용자 프로세스(OS Process?)를 하나 할당해주는 이유(만들어주는게 아니라, 서버 프로세스를 생성하거나 연결시켜준다) → Transaction 을 해야 해서. → 서버 프로세스와 통신을 통해 SQL 실행, 트랜잭션 처리 등 함
- 서버 프로세스에서 수행하는 거 : Syntax check, Semantic check,

SQL Context(실행 계획) - shared pool의 library cache에서 찾아서 있으면, 실행해서 fetch함 → soft parse라고 함(OLTP에서는 99% 이상 soft parse)

SGA??

navigation?

1. Optimizer → DBMS 핵심 엔진 → SQL 최적화

Path 수집 - Path 조합, 비용 예측(Cpu + Memort + I/O) - Meta 정보 추가 I/O + 비용 계싼

Server Process는 Raw Source로 받고, 이걸 다시 Libary cache로 저장함. why? hard parsing 하지 않기 위해. soft parsing하기 위해

1. Excute
2. fetch

기억할 것 : Soft Parsing과 Hard Parsing. 가끔 sql 작성 규칙이 아주 엄격한 경우는 이걸 막기 위함

file로 읽는 것보다 disk에서 읽는게 성능이 만 배는 차이 난다. 최소한만 캐시하는게 좋겠지?

```sql
[사용자 SQL 입력]
   ↓
tnsnames.ora 통해 접속 정보 확인
   ↓
[Listener] → 접속 요청 전달
   ↓
[Oracle Server]
   ├─ 인증
   ├─ Server Process + PGA 생성
   └─ Session 생성
       ↓
     SQL 파싱
       ↓
  ┌────────────┐
  │ Shared Pool│ ← 실행 계획 있는지 확인 (Soft Parse)
  └────────────┘
       ↓
    Optimizer 실행 (필요시 Hard Parse)
       ↓
     SQL 실행 → 결과 Fetch

```

| 키워드 | 의미 |
| --- | --- |
| **User Process** | 클라이언트 측 SQL 실행 프로세스 |
| **Server Process** | Oracle 서버가 할당한 SQL 처리용 프로세스 |
| **tnsnames.ora** | 접속 정보를 담은 주소록 파일 |
| **Listener** | 외부 접속 요청을 수신해 Oracle에 연결 |
| **SGA** | 공유 메모리 (SQL 캐시, 데이터 캐시 등) |
| **PGA** | 개별 사용자 전용 메모리 |
| **Soft Parse** | 실행 계획 재사용 |
| **Optimizer** | SQL 실행 최적화 로직 |

```sql
1. 사용자가 SELECT 문 작성

2. 사용자 프로세스 → 서버 프로세스에게 SQL 전달

3. Syntax Check
   - 문법 오류 있는지 확인

4. Semantic Check
   - 테이블 존재하는지
   - 컬럼 존재하는지
   - 접근 권한 있는지
   - 이 때, **Data Dictionary (Row Cache)** 참조

5. Shared Pool의 Library Cache 조회 (Soft Parse 시도)
   - 이전에 동일한 SQL + 동일 사용자라면 → 실행계획 재사용 (Soft Parse 성공)

6. Soft Parse 실패 시 → Optimizer 작동 (Hard Parse 발생)
   - 여러 접근 경로(path) 탐색
   - 비용 기반 최적화 (CPU, I/O 등)
   - 최적의 실행 계획 수립

7. 실행 계획(SQL Context)을 Library Cache에 저장

8. SQL 실행 (Execute 단계)
   - 필요한 데이터는 먼저 **Buffer Cache**에서 찾음
   - 없다면 → **Disk(Data File)**에서 읽음
   - 읽은 데이터는 다시 Buffer Cache에 저장됨

9. 결과를 사용자에게 반환 (Fetch)

```

| 용어 | 설명 |
| --- | --- |
| **Soft Parse** | 이전에 Library Cache에 실행 계획이 저장되어 있으면 → 그걸 재사용 |
| **Hard Parse** | 실행 계획이 없어 Optimizer가 새로 만들어야 함 |
| **Row Cache (Data Dictionary)** | 테이블 구조, 권한 같은 메타데이터 캐시 |
| **Library Cache** | SQL 문과 실행계획 저장소 |
| **Buffer Cache** | 실제 테이블의 데이터 캐시 |
| **Data File** | 디스크에 저장된 실제 데이터 파일 (최후 보루) |

# 기본 Select 문

1. Projection : 데이터베이스에서 원하는 속성, 열을 선택 조회 (열 중심)
2. Selection : 특정 조건 만족하는 행, 레코드 검색
3. Join : 두 개 이상 테이블을 연결

select ~ from ~ oracle 에서는 필수절

select 표현식 mysql, postgresql, db2, ms sql  server 등 필수절

project 검색 :

select [distinct] * | 컬럼명, … | 표현식

from 테이블명;

db2, ms sql server → pk 기준으로 오름차순 저장 구조(기본 테이블, cluster table)

oracle → block에 insert한 순서대로(힙)

## 데이터 조희 기본

```
select  ~ from  ~ oracle 에서는 필수절
select 표현식 mysql, postgresql, db2, ms sql server등 필수절

project 검색 :
select  [distinct] * | 컬럼명, .....| 표현식
from  테이블명;

select *
from emp;  --테이블의 모든 데이터 조회

--insert한 순서대로  또는 block에 저장된 순서대로 반환 , oracle 은 heap table
--db2와 sql server dbms에서는 기본 table 저장 구조 PK기준으로 정렬되어서 저장 구조
```

## sal*1.1 처리 구조

```sql

select deptno, ename, job, sal
from emp;
--조회할 컬럼만 select절에 선언
--테이블에 선언된 컬럼 순서에 영향을 받지 않고 조회할 컬럼 선언 가능

select deptno, ename, job, sal, sal*1.1
from emp ;
--컬럼에 함수와 연산자를 적용 => expression 표현식
-- expression 표현식은 메모리에서 수행되는 연산, (물리적으로 database에 영향을 주지 않습니다)

```

### sal*1.1은 메모리에서 처리된다. 즉 원래 데이터 베이스는 바뀌지 않음(메모리 접근 구조로 이해하기)

## 칼럼 타입

```sql
desc emp
또는 
describe emp

[oracle 컬럼타입]
number (p, s) - 정수와 실수를 저장하는 타입, binary로 값으로 저장
char(~2000)- 고정길이 문자열
varchar2(~4000) - 가변길이 문자열 (ANSI협회 에서 정의된 varchar, text, string.. 호환됩니다.)
date :  (__세기 __년 __월 __일 __시 __분 __초) , 7byte 
timestamp : date+ fractional second (1/10억) 
timestamp with timezone
interval year to month
interval day to second
RAW(~2000) : binary data
BLOB (~4GB)
CLOB (~4GB) 
rowid : 내부적으로 사용하는 (논리적)행주소 , 가상 컬럼, 내장 컬럼, 
         objectid+fileid+blockid+행순서번호
        index에 저장되는 필수 값
select rowid, empno, ename
from emp;
BFile  : file단위로 메타정보(경로와 파일명 등)만 database에 저장, file은 OS의 FileSystem에 저장
         읽기 전용의 streaming  서비스에 적합
```

## 산술, 결합 연산자

```sql
#컬럼 타입별로 적용 가능한 연산자가 다릅니다.
# number 타입 컬럼에는 산술연산자, 결합연산자를 적용
select  ename, sal + 100, sal-100, sal*2, sal/2
from emp;
#문자열(char, varchar2)은 결합연산자 : ||
select ename||job
from emp; 

select sal||hiredate 
from emp;  --결과는 문자열로 변환

select sal||' ' ||hiredate 
from emp;
--DB에서는 날짜 상수값과 문자열 상수값은  ' '로 감싸줘야 합니다.

select ename||' works as '||job
from emp;

```

## 날짜(sysdate)

```sql
#날짜 타입의 표현식에서 가능한 연산 : +n, -n, date타입-date타입

select sysdate 
from dual; --  가상 테이블(dummy 컬럼이 존재하는 테이블, sys계정 소유의 테이블, public synonym)

--DB Connection 된 session의  nls_date_format파라미터중의 기본값은 'RR/MM/DD' 
alter session set nls_date_format='YYYY/MM/DD HH24:MI:SS';
select sysdate 
from dual;

select sysdate , sysdate+1, sysdate-1
from dual;

select sysdate , sysdate+1/24, sysdate-1/24
from dual;

select sysdate , sysdate+5/1440, sysdate-5/1440
from dual;

select sysdate-hiredate
from emp; 
```

세션 객체 : 논리적으로 연결상태 관리

→ 설정 파라미터 RR/MM/DD

## null, distinct에 대해서

```sql
--null은 값이 할당되지 않음 
--null은 산술, 비교, 논리 모든 연산에서 null을 리턴
--null은  ''는 동일 의미,  ' '은 null과 다름,  0을 의미하지 않습니다

select ename, sal, comm, sal+comm
from emp;

select  deptno
from emp; --모든 사원(14명)의 소속 부서번호 조회

select distinct deptno
from emp;  --중복 제거 (sorting연산, hash함수)

select  deptno, distinct job
from emp;   --- ? error
-- distinct 키워드는 select, insert, update, delete 키워드 다음에만 선언할 수 있습니다.

select distinct deptno,  job
from emp; 

```

### 참고(물리, 논리적 저장 단위)

| 개념 | 비유 | 설명 |
| --- | --- | --- |
| **테이블스페이스 (Tablespace)** | 도서관 | 데이터베이스 전체에서 공간을 나눈 큰 단위입니다. 각 테이블스페이스는 데이터가 저장될 “큰 구역”입니다. |
| **세그먼트 (Segment)** | 책장 | 테이블, 인덱스처럼 실제 데이터가 저장되는 객체 하나에 할당된 공간입니다. |
| **익스텐트 (Extent)** | 책장의 한 줄 | 세그먼트(책장)는 너무 크니까 “한 줄씩” 공간을 배정해서 조금씩 확장합니다. |
| **블록 (Block)** | 책 하나 | 가장 작은 저장 단위로, 실제로 데이터를 담는 곳입니다. Oracle에서는 1 블록 = 몇 KB (예: 8KB) |

```sql
[테이블스페이스]
 └── 세그먼트 (EMP 테이블)
     └── 익스텐트 #1
         └── 블록 1
         └── 블록 2
         ...
     └── 익스텐트 #2
         └── 블록 ...

```

## 문자 출력, Alias, rownum

```sql
Q>  A 문자 출력
select 'A'
from dual;

Q>  'A' 문자 출력
select '''A'''
from dual; 

select q'['A']'
from dual;

select q'{'A'}'
from dual;

Q>  "A" 문자 출력
select '"A"'
from dual;

select ename, sal, sal*12 as salary  --별칭 (컬럼 표현식 대신 컬럼 heading을 rename처리)
from emp;

select ename, sal, sal*12 salary  --별칭 (컬럼 표현식 대신 컬럼 heading을 rename처리)
from emp;

select ename, sal, sal*12 "Increase Salary"  --별칭에 공백 포함 및  대소문자 구별하려면 " "사용
from emp;

select rownum, empno, ename
from emp;

```

## Selection 검색(where, 비교 연산자)

```sql
selection 검색--------------------------------------------------------------------
#selection검색 - 1개의 대상 table, 행 기준 검색, 행 조건 선언
select ~
from ~ 
where  조건;   -- 행 기준 검색,  컬럼 비교연산자 비교값
--비교연산자 :    =, >, <, >=, <=, !=, <>, ^=

Q>emp테이블의 사원 데이터중에서 10번 부서 소속의 사원 정보만 검색
select *
from emp
where deptno = 10 ;

Q> emp테이블의 사원 데이터중에서 직무가 salesman인  사원 정보만 검색
--문자열 컬럼은 대소문자 구별하고, ' '을 사용해서 비교값을 선언해야 합니다.
select *
from emp
where job = 'SALESMAN';

Q> emp테이블의 사원 데이터중에서 1982년1월 1일 이후에 사원 정보만 검색
-- 세션의 format  형식  'RR/MM/DD'과 일치하므로 자동 형변환?
alter session set nls_date_format ='RR/MM/DD';
--유효한 날짜 형식의 값인 경우, 비교하려는 컬럼타입에 맞춰 형변환을 자동으로처리해줌
select *
from emp
where hiredate >= '1982/01/01';   --3rows

alter session set nls_date_format='YYYY/MM/DD HH24:MI:SS';
select *
from emp
where hiredate >= '1982/01/01 00:00:00';

Q>emp테이블의 사원 데이터중에서 급여가 2000이상 ~3000이하를 받는 사원 정보 검색
--범위 비교 연산자  : between 하한값 and 상한값 (하한값 포함, 상한값 포함)
--1개이상의 조건을 논리 연산자와 함께 선언 :   조건1 and 조건2
select *
from emp
where  sal  between 2000 and 3000;  --5rows

select *
from emp
where sal >= 2000 
and sal <=3000;   

Q>emp테이블의  사원 데이터중에서 직무가  analyst 또는 manager인 사원 검색
--inlist 연산자( membership 연산자 ) :   비교컬럼  in (값, 값, 값, ...)
--1개이상의 조건을 논리 연산자와 함께 선언 :  조건1 or 조건2

select *
from emp
where job in ('ANALYST', 'MANAGER');

select *
from emp
where job ='ANALYST'
or job = 'MANAGER';
```

## 데이터 베이스가 Excel에 비해 갖는 장점

1. 서로 다른 Sheet에서 추출 가능
2. Index 
3. 등등

## Character Pattern Matching

```sql
--character pattern matching 연산자 :   like '문자열 패턴', 만능문자 %,_
-- % 만능문자 zero or more,  문자 or 숫자 or 특수문자 허용
-- _ 만능문자 1개의 문자 or 숫자 or 특수문자 허용
--복잡한 패턴은 정규표현식과 regexp_xxxx함수를 사용합니다

Q>사원이름 중에서 A로 시작하는 사원 정보 검색
select *
from emp
where ename like 'A%';

Q> 사원이름중에서 N으로 끝나는 사원 정보 검색
select *
from emp
where ename like '%N';

Q> 사원이름중에서 세번째 문자에 E인  사원 정보 검색
select *
from emp
where ename like '__E%';

-- drop table test purge;
-- 실습을 위한 테이블 생성
create table test (
name  varchar2(20)
);

desc test

insert into test values ('korea');
insert into test values ('seoul');
insert into test values ('incheon');
insert into test values ('dongtan');
insert into test values ('dong%tan');
insert into test values ('dong_tan');
insert into test values ('dong%1');
insert into test values ('dong_2');
commit; 

Q> test테이블로부터 name에 dong%로 시작하는 데이터를 검색하시오
select * 
from test
where name like 'dong%%';  ---X

select * 
from test
where name like 'dong\%%' escape '\'; 

Q> test테이블로부터 name에 dong_로 시작하는 데이터를 검색하시오
select * 
from test
where name like 'dong_%';  ---X

select * 
from test
where name like 'dong?_%' escape '?';

```

## Null

### null은 비교 연산 불가능.(산술 연산도 불가능이지만)

→ is null, is not null

```sql
-- null값을 비교하기 위한 연산자는  is null, is not null 연산자
Q> 사원중에서 커미션이 null인 사원 조회 (커미션 받지 않은 사원)
select *
from emp
where comm = null; ---X

select *
from emp
where comm is null;

Q> 사원중에서 커미션을 받는  사원 조회
select *
from emp
where comm != null;  ---X

select *
from emp
where comm is not null;
```

## 논리 연산자

### ★★우선순위 : NOT, AND, OR 순

```sql

# 조건이 여러 개를 선언하는 경우 , 논리 연산자  not, and, or
Q> 업무가 PRESIDENT이고 급여가 1500 이상이거나 업무가 SALESMAN인 사원의 사원번호, 이름, 업무, 급여를  SELECT하시오
select  empno, ename, job, sal 
from emp
where job = 'PRESIDENT' and sal >= 1500 or job = 'SALESMAN';

Q> 급여가 1500 이상인 사원중 업무가  PRESIDENT 이거나  SALESMAN 인 사원번호, 이름, 업무, 급여를 SELECT하시오

select  empno, ename, job, sal 
from emp
where  sal >= 1500 and (job = 'PRESIDENT' or job = 'SALESMAN');

select  empno, ename, job, sal 
from emp
where  sal >= 1500 and  job in ( 'PRESIDENT' , 'SALESMAN');

Q>EMP 테이블에서 입사일자가 82년도에 입사한 사원의 사번, 성명, 담당업무, 급여, 입사일자, 부서번호를 출력하라.
select  empno, ename, job, sal, hiredate
from emp
where hiredate between '82/01/01' and '82/12/31';

select  empno, ename, job, sal, hiredate
from emp
where hiredate >= '82/01/01' 
 and hiredate <= '82/12/31';

select  empno, ename, job, sal, hiredate
from emp
where hiredate like '82%';  ---문자열 변환되어 비교처리됨, index scan 불가,
```

## 문제 풀이

```sql

Q5> EMP 테이블에서 담당하고 있는 업무의 종류를 출력하여라.
select distinct job
from emp;

Q3> EMP 테이블에서 이름과 업무를 "KING is a PRESIDENT" 형식으로 출력하여라.
select ename ||' is a '|| job
from emp;

Q2> EMP 테이블에서 ENAME를 '성 명'으로, SAL를 '급 여'로 출력하여라.
select ename as "성 명" , sal  as "급 여"
from emp;

Q15> select * 
from emp
where sal+nvl(comm, 0)  >   sal*1.1;

where 조건컬럼  not  between 하한값 and 상한값
where 조건컬럼  not in (값1, 값, 값3,...)
where 조건컬럼  not like '패턴문자%'
where 조건 컬럼 is not null
```

## 정렬

```sql
--------------------------------------------------------------------------
oracle 의 table은 heap table입니다.
(insert한 순서 또는 block에 저장된 순서로 결과를 반환)
특정 컬럼 기준으로 정렬된 결과를 생성하려면,  메모리에서 정렬 연산을 수행
정렬은 물리적으로 저장된 data와 무관...

select~
from ~
[where ~]
order by  정렬기준컬럼   정렬방식;
--정렬방식에는 오름차순(asc), 내림차순(desc)을 선언합니다.
--정렬방식 선언을 생략하면 오름차순(asc)이 기본값입니다.
--oracle에서는 null은 오름차순으로 정렬할 경우 뒤에 , 내림차순으로 정렬할 경우 앞에 (기본) 

Q> 사원데이터를 급여의 내림차순으로 정렬한 집합 출력하는 selet문 구현
select ename, sal
from emp
order by sal  desc;

--order by절에 1차 정렬컬럼과 정렬방식 선언 후, 2차 정렬컬럼과 정렬방식 , .............
Q> 사원들을  부서번호의 오름차순과 동일 부서 데이터는 급여의 내림차순으로 정렬한 집합 출력하는 selet문 구현
select deptno, ename, sal
from emp
order by deptno , sal desc;

select  deptno, sal, ename, (sal+nvl(comm, 0))*12 "Annual Salary"
from emp;

Q> select절에 선언되지 않은 날짜 컬럼으로 정렬 가능?  ---O
select  deptno, sal, ename, (sal+nvl(comm, 0))*12 "Annual Salary"
from emp
order by hiredate;
Q> order by절에 select절의 표현식으로 정렬 가능? ---O
select  deptno, sal, ename, (sal+nvl(comm, 0))*12 "Annual Salary"
from emp
order by (sal+nvl(comm, 0))*12 desc;

Q> order by절에 select절의 별칭으로  정렬 가능?  ---O
select  deptno, sal, ename, (sal+nvl(comm, 0))*12 "Annual Salary"
from emp
order by "Annual Salary" desc;

Q> order by절에 select절에 선언된 컬럼 position(offset)으로 정렬 가능? --O
select  deptno, sal, ename, (sal+nvl(comm, 0))*12 "Annual Salary"
from emp
order by 3 ;

Q16>
select reserv_date, visitor_cnt             ---3
from reservation                          ---1
where visitor_cnt > =5                     ---2 (filter조건)
order by reserv_date, visitor_cnt desc ;     ----4

```

# 함수

★DB 함수는 반드시 1개의 값을 반환

복잡한 SQL → 간결하게 작성

## 종류

1. 단일 행 함수 → 1행 1결과
2. 다중 행 함수 → n행 1결과
3. 분석 함수(Window Function) → partition 내에서 범위를 이동해가면서 계산

```sql
함수------------------------------------------------------------
기능을 재사용 가능하도록 모듈화

--DB 함수는 반드시 하나의 값을 리턴합니다.

--단일행함수
--그룹함수, 집계함수, multiple row function
--분석함수 , window함수

단일행 함수 - character function
              number function
              date function
              conversion function
              null처리 함수와 조건처리함수

select length('korea'), length('대한민국'), lengthb('korea'), lengthb('대한민국')  
from dual;
--substr(문자열column, start offset, length)
-- Oracle은 이 경우 1부터 센다. -1은 그냥 원래대로 역순
select  substr('hello world' , 7, 3 ), substr('hello world' , 7  )
        , substr('hello world' , -5, 3 )
from dual;

--replace(문자열column, source문자열, target문자열)
--translate(문자열column, source문자열, 한문자단위로매핑해서변환문자열)
select  replace('hello world' , 'world', 'sql' )
from dual;

select  translate('hello world' , 'world', 'sql73' )
from dual;

--instr(문자열column, search문자열, start offset, n번째 위치(요소))
SELECT instr('CORPORATE FLOOR','OR') , instr('CORPORATE FLOOR','OR', 1, 1) 
       , instr('CORPORATE FLOOR','OR', 3, 2)
       , instr('CORPORATE FLOOR','or') 
FROM DUAL;

#단일행함수는 nested할 수 있습니다.
--rtrim, ltrim, trim : 공백제거 또는 왼쪽 , 오른쪽에 문자를 제거
SELECT length('  hello  world  '), length( trim('  hello  world  '))
from dual;  --공백 제거 (왼쪽, 오른쪽)

SELECT  ltrim('hello  world', 'he'), rtrim('hello  world', 'rld')
from dual;

SELECT  trim(leading 'h' from 'hello  world'  ), trim( trailing 'd' from 'hello  world')
from dual;

```

## 정규 표현식

```sql
SELECT REGEXP_COUNT('apple, banana, mango, melon', '[a-z]+') AS word_count
FROM dual;

SELECT REGEXP_COUNT('My ID is 1234 and my code is 9876', '\d+') AS number_count
FROM dual;

SELECT REGEXP_INSTR('abc123def456', '\d+') AS first_digit_pos
FROM dual;

SELECT REGEXP_REPLACE('abc123def456', '\d+', '###') AS replaced_text
FROM dual;

```

## 수학

```sql
SELECT POWER( 2, 10) 결과1, CEIL (3.7) 결과2, FLOOR (3.7) 결과3
from dual;

select  sign(1000), sign(-2000), sign(0)
from dual; 

SELECT round(123.4567, 2) , round(123.4567, 0) , round(123.4567), round(12345.67 , -2)
from dual;

-- 0이 소숫점 첫번쨰 자리, 그 자리'에서' 반올림하라는 뜻

SELECT trunc(123.4567, 2) , trunc(123.4567, 0) , trunc(123.4567), trunc(12345.67 , -2)
from dual;

--함수는 select절, where절 사용 가능
select mod(3, 2) 
from dual;

Q> 사원번호가 홀수인 사원 정보만 검색 (emp)
select  empno, ename
from emp
where mod(empno, 2) = 1;

```

## 날짜

```sql
datetime function -----------------------------------------------------------------
alter session set nls_date_format='YYYY/MM/DD HH24:MI:SS';

select  sysdate, systimestamp, current_date, current_timestamp, sessiontimezone
from dual;
-- current_xxx는 세션에 설정된 timezone기반의 현재 시간을 반환
-- sysdate, systimestamp 는 OS에 설정된 현재 시간 반환

alter session set time_zone='+3:00';

select  sysdate, systimestamp, current_date, current_timestamp, sessiontimezone
from dual;

날짜 데이터로부터 년도, 월, 일, 요일, 시간 ...등 추출 (문자 데이터로 추출, 수치 데이터 추출) : to_char(), extract()

select  to_char(sysdate, 'MONTH'),  extract(month from sysdate)
       , to_char(sysdate, 'MON'), to_char(sysdate, 'MM')
from dual;

select  to_char(sysdate, 'DAY'),  extract(DAY from sysdate)
       , to_char(sysdate, 'D'), to_char(sysdate, 'DDD') , to_char(sysdate, 'DY')
from dual;

-- month면 day에서 반올림, year면 month에서 반올림
-- 15이하면 내림, 16일부터 올림
select  round(to_date('24/06/15', 'RR/MM/DD'), 'MONTH')
        , round(to_date('24/06/16', 'RR/MM/DD'), 'MONTH')
        , round(to_date('24/06/15', 'RR/MM/DD'), 'YEAR')
        , round(to_date('24/07/15', 'RR/MM/DD'), 'YEAR')
from dual;

select  trunc(to_date('24/06/30', 'RR/MM/DD'), 'MONTH')
        , trunc(to_date('24/06/16', 'RR/MM/DD'), 'MONTH')
        , trunc(to_date('24/12/01', 'RR/MM/DD'), 'YEAR')
        , trunc(to_date('24/07/15', 'RR/MM/DD'), 'YEAR')
from dual;

select  months_between(sysdate, hiredate), add_months(sysdate, 6)
from emp;

select  last_day(to_date('2012/02/01'))
       , last_day(to_date('2100/02/01'))
       , last_day(sysdate)
from emp; 

4.
  selec *
  from emp
  where  substr(ename, 1, 1) >'K' and substr(ename, 1, 1) <'Y';
6. 
  select  ename, instr(ename, 'L', 1)
  from emp;
7.
  select job , ltrim(job, 'A'), trim(leading 'A'  from job),
         sal , ltrim(sal, '1'),  trim (leading '1'  from sal)
   from emp
   where deptno = 10;
```

# 수업 노트 1 원본

```sql
Window 로고 + R 
실행> cmd
C:\Users\User>sqlplus /nolog

SQL> conn / as sysdba

SQL> alter user hr
  2  identified by oracle
  3  account unlock;

SQL> conn hr/oracle

select tname from tab;  -- 테이블 목록 조회

SQL> @c:\scott.sql    --sql 명령어들을 저장한  스크립트 파일(.sql)

SQL> describe emp   --테이블 구조 조회

SQL> desc dept   --테이블 구조 조회

-- desc 또는 describe 는 tool 명령어입니다.

select, insert, from, where,....키워드는 대소문자 구분 X
테이블명, 컬러명 대소문자 구분 X
컬럼값은 대소문자 구분합니다
select, insert, from, where,....키워드는 테이블명, 컬럼명, user명으로 사용할 수 없습니다.
SQL문장은 ; 으로 종료합니다.

Window 로고 + R 
실행> services.msc

cmd> lsnrctl status

------------------------------------------------------------------------
select  ~ from  ~ oracle 에서는 필수절
select 표현식 mysql, postgresql, db2, ms sql server등 필수절

project 검색 :
select  [distinct] * | 컬럼명, .....| 표현식 
from  테이블명;

select *
from emp;  --테이블의 모든 데이터 조회

--insert한 순서대로  또는 block에 저장된 순서대로 반환 , oracle 은 heap table
--db2와 sql server dbms에서는 기본 table 저장 구조 PK기준으로 정렬되어서 저장 구조

select deptno, ename, job, sal
from emp;
--조회할 컬럼만 select절에 선언
--테이블에 선언된 컬럼 순서에 영향을 받지 않고 조회할 컬럼 선언 가능

select deptno, ename, job, sal, sal*1.1
from emp ;
--컬럼에 함수와 연산자를 적용 => expression 표현식
-- expression 표현식은 메모리에서 수행되는 연산, (물리적으로 database에 영향을 주지 않습니다)

desc emp
또는 
describe emp

[oracle 컬럼타입]
number (p, s) - 정수와 실수를 저장하는 타입, binary로 값으로 저장
char(~2000)- 고정길이 문자열
varchar2(~4000) - 가변길이 문자열 (ANSI협회 에서 정의된 varchar, text, string.. 호환됩니다.)
date :  (__세기 __년 __월 __일 __시 __분 __초) , 7byte 
timestamp : date+ fractional second (1/10억) 
timestamp with timezone
interval year to month
interval day to second
RAW(~2000) : binary data
BLOB (~4GB)
CLOB (~4GB) 
rowid : 내부적으로 사용하는 (논리적)행주소 , 가상 컬럼, 내장 컬럼, 
         objectid+fileid+blockid+행순서번호
        index에 저장되는 필수 값
select rowid, empno, ename
from emp;
BFile  : file단위로 메타정보(경로와 파일명 등)만 database에 저장, file은 OS의 FileSystem에 저장
         읽기 전용의 streaming  서비스에 적합

#컬럼 타입별로 적용 가능한 연산자가 다릅니다.
# number 타입 컬럼에는 산술연산자, 결합연산자를 적용
select  ename, sal + 100, sal-100, sal*2, sal/2
from emp;
#문자열(char, varchar2)은 결합연산자 : ||
select ename||job
from emp; 

select sal||hiredate 
from emp;  --결과는 문자열로 변환

select sal||' ' ||hiredate 
from emp;
--DB에서는 날짜 상수값과 문자열 상수값은  ' '로 감싸줘야 합니다.

select ename||' works as '||job
from emp;

#날짜 타입의 표현식에서 가능한 연산 : +n, -n, date타입-date타입

select sysdate 
from dual; --  가상 테이블(dummy 컬럼이 존재하는 테이블, sys계정 소유의 테이블, public synonym)

--DB Connection 된 session의  nls_date_format파라미터중의 기본값은 'RR/MM/DD' 
alter session set nls_date_format='YYYY/MM/DD HH24:MI:SS';
select sysdate 
from dual;

select sysdate , sysdate+1, sysdate-1
from dual;

select sysdate , sysdate+1/24, sysdate-1/24
from dual;

select sysdate , sysdate+5/1440, sysdate-5/1440
from dual;

select sysdate-hiredate
from emp; 

--null은 값이 할당되지 않음 
--null은 산술, 비교, 논리 모든 연산에서 null을 리턴
--null은  ''는 동일 의미,  ' '은 null과 다름,  0을 의미하지 않습니다

select ename, sal, comm, sal+comm
from emp;

select  deptno
from emp; --모든 사원(14명)의 소속 부서번호 조회

select distinct deptno
from emp;  --중복 제거 (sorting연산, hash함수)

select  deptno, distinct job
from emp;   --- ? error
-- distinct 키워드는 select, insert, update, delete 키워드 다음에만 선언할 수 있습니다.

select distinct deptno,  job
from emp; 

Q>  A 문자 출력
select 'A'
from dual;

Q>  'A' 문자 출력
select '''A'''
from dual; 

select q'['A']'
from dual;

select q'{'A'}'
from dual;

Q>  "A" 문자 출력
select '"A"'
from dual;

select ename, sal, sal*12 as salary  --별칭 (컬럼 표현식 대신 컬럼 heading을 rename처리)
from emp;

select ename, sal, sal*12 salary  --별칭 (컬럼 표현식 대신 컬럼 heading을 rename처리)
from emp;

select ename, sal, sal*12 "Increase Salary"  --별칭에 공백 포함 및  대소문자 구별하려면 " "사용
from emp;

select rownum, empno, ename
from emp;

selection 검색--------------------------------------------------------------------
#selection검색 - 1개의 대상 table, 행 기준 검색, 행 조건 선언
select ~
from ~ 
where  조건;   -- 행 기준 검색,  컬럼 비교연산자 비교값
--비교연산자 :    =, >, <, >=, <=, !=, <>, ^=

Q>emp테이블의 사원 데이터중에서 10번 부서 소속의 사원 정보만 검색
select *
from emp
where deptno = 10 ;

Q> emp테이블의 사원 데이터중에서 직무가 salesman인  사원 정보만 검색
--문자열 컬럼은 대소문자 구별하고, ' '을 사용해서 비교값을 선언해야 합니다.
select *
from emp
where job = 'SALESMAN';

Q> emp테이블의 사원 데이터중에서 1982년1월 1일 이후에 사원 정보만 검색
-- 세션의 format  형식  'RR/MM/DD'과 일치하므로 자동 형변환?
alter session set nls_date_format ='RR/MM/DD';
--유효한 날짜 형식의 값인 경우, 비교하려는 컬럼타입에 맞춰 형변환을 자동으로처리해줌
select *
from emp
where hiredate >= '1982/01/01';   --3rows

alter session set nls_date_format='YYYY/MM/DD HH24:MI:SS';
select *
from emp
where hiredate >= '1982/01/01 00:00:00';

Q>emp테이블의 사원 데이터중에서 급여가 2000이상 ~3000이하를 받는 사원 정보 검색
--범위 비교 연산자  : between 하한값 and 상한값 (하한값 포함, 상한값 포함)
--1개이상의 조건을 논리 연산자와 함께 선언 :   조건1 and 조건2
select *
from emp
where  sal  between 2000 and 3000;  --5rows

select *
from emp
where sal >= 2000 
and sal <=3000;   

Q>emp테이블의  사원 데이터중에서 직무가  analyst 또는 manager인 사원 검색
--inlist 연산자( membership 연산자 ) :   비교컬럼  in (값, 값, 값, ...)
--1개이상의 조건을 논리 연산자와 함께 선언 :  조건1 or 조건2

select *
from emp
where job in ('ANALYST', 'MANAGER');

select *
from emp
where job ='ANALYST'
or job = 'MANAGER';

--character pattern matching 연산자 :   like '문자열 패턴', 만능문자 %,_
-- % 만능문자 zero or more,  문자 or 숫자 or 특수문자 허용
-- _ 만능문자 1개의 문자 or 숫자 or 특수문자 허용
--복잡한 패턴은 정규표현식과 regexp_xxxx함수를 사용합니다

Q>사원이름 중에서 A로 시작하는 사원 정보 검색
select *
from emp
where ename like 'A%';

Q> 사원이름중에서 N으로 끝나는 사원 정보 검색
select *
from emp
where ename like '%N';

Q> 사원이름중에서 세번째 문자에 E인  사원 정보 검색
select *
from emp
where ename like '__E%';

drop table test purge;

create table test (
name  varchar2(20)
);

desc test

insert into test values ('korea');
insert into test values ('seoul');
insert into test values ('incheon');
insert into test values ('dongtan');
insert into test values ('dong%tan');
insert into test values ('dong_tan');
insert into test values ('dong%1');
insert into test values ('dong_2');
commit; 

select * from test;

Q> test테이블로부터 name에 dong%로 시작하는 데이터를 검색하시오
select * 
from test
where name like 'dong%%';  ---X

select * 
from test
where name like 'dong\%%' escape '\'; 

Q> test테이블로부터 name에 dong_로 시작하는 데이터를 검색하시오
select * 
from test
where name like 'dong_%';  ---X

select * 
from test
where name like 'dong?_%' escape '?';

-- null값을 비교하기 위한 연산자는  is null, is not null 연산자
Q> 사원중에서 커미션이 null인 사원 조회 (커미션 받지 않은 사원)
select *
from emp
where comm = null; ---X

select *
from emp
where comm is null;

Q> 사원중에서 커미션을 받는  사원 조회
select *
from emp
where comm != null;  ---X

select *
from emp
where comm is not null;

# 조건이 여러 개를 선언하는 경우 , 논리 연산자  not, and, or
Q> 업무가 PRESIDENT이고 급여가 1500 이상이거나 업무가 SALESMAN인 사원의 사원번호, 이름, 업무, 급여를  SELECT하시오
select  empno, ename, job, sal 
from emp
where job = 'PRESIDENT' and sal >= 1500 or job = 'SALESMAN';

Q> 급여가 1500 이상인 사원중 업무가  PRESIDENT 이거나  SALESMAN 인 사원번호, 이름, 업무, 급여를 SELECT하시오

select  empno, ename, job, sal 
from emp
where  sal >= 1500 and (job = 'PRESIDENT' or job = 'SALESMAN');

select  empno, ename, job, sal 
from emp
where  sal >= 1500 and  job in ( 'PRESIDENT' , 'SALESMAN');

Q>EMP 테이블에서 입사일자가 82년도에 입사한 사원의 사번, 성명, 담당업무, 급여, 입사일자, 부서번호를 출력하라.
select  empno, ename, job, sal, hiredate
from emp
where hiredate between '82/01/01' and '82/12/31';

select  empno, ename, job, sal, hiredate
from emp
where hiredate >= '82/01/01' 
 and hiredate <= '82/12/31';

select  empno, ename, job, sal, hiredate
from emp
where hiredate like '82%';  ---문자열 변환되어 비교처리됨, index scan 불가,

Q5> EMP 테이블에서 담당하고 있는 업무의 종류를 출력하여라.
select distinct job
from emp;

Q3> EMP 테이블에서 이름과 업무를 "KING is a PRESIDENT" 형식으로 출력하여라.
select ename ||' is a '|| job
from emp;

Q2> EMP 테이블에서 ENAME를 '성 명'으로, SAL를 '급 여'로 출력하여라.
select ename as "성 명" , sal  as "급 여"
from emp;

Q15> select * 
from emp
where sal+nvl(comm, 0)  >   sal*1.1;

where 조건컬럼  not  between 하한값 and 상한값
where 조건컬럼  not in (값1, 값, 값3,...)
where 조건컬럼  not like '패턴문자%'
where 조건 컬럼 is not null

--------------------------------------------------------------------------
oracle 의 table은 heap table입니다.
(insert한 순서 또는 block에 저장된 순서로 결과를 반환)
특정 컬럼 기준으로 정렬된 결과를 생성하려면,  메모리에서 정렬 연산을 수행
정렬은 물리적으로 저장된 data와 무관...

select~
from ~
[where ~]
order by  정렬기준컬럼   정렬방식;
--정렬방식에는 오름차순(asc), 내림차순(desc)을 선언합니다.
--정렬방식 선언을 생략하면 오름차순(asc)이 기본값입니다.
--oracle에서는 null은 오름차순으로 정렬할 경우 뒤에 , 내림차순으로 정렬할 경우 앞에 (기본) 

Q> 사원데이터를 급여의 내림차순으로 정렬한 집합 출력하는 selet문 구현
select ename, sal
from emp
order by sal  desc;

--order by절에 1차 정렬컬럼과 정렬방식 선언 후, 2차 정렬컬럼과 정렬방식 , .............
Q> 사원들을  부서번호의 오름차순과 동일 부서 데이터는 급여의 내림차순으로 정렬한 집합 출력하는 selet문 구현
select deptno, ename, sal
from emp
order by deptno , sal desc;

select  deptno, sal, ename, (sal+nvl(comm, 0))*12 "Annual Salary"
from emp;

Q> select절에 선언되지 않은 날짜 컬럼으로 정렬 가능?  ---O
select  deptno, sal, ename, (sal+nvl(comm, 0))*12 "Annual Salary"
from emp
order by hiredate;
Q> order by절에 select절의 표현식으로 정렬 가능? ---O
select  deptno, sal, ename, (sal+nvl(comm, 0))*12 "Annual Salary"
from emp
order by (sal+nvl(comm, 0))*12 desc;

Q> order by절에 select절의 별칭으로  정렬 가능?  ---O
select  deptno, sal, ename, (sal+nvl(comm, 0))*12 "Annual Salary"
from emp
order by "Annual Salary" desc;

Q> order by절에 select절에 선언된 컬럼 position(offset)으로 정렬 가능? --O
select  deptno, sal, ename, (sal+nvl(comm, 0))*12 "Annual Salary"
from emp
order by 3 ;

Q16>
select reserv_date, visitor_cnt             ---3
from reservation                          ---1
where visitor_cnt > =5                     ---2 (filter조건)
order by reserv_date, visitor_cnt desc ;     ----4

함수------------------------------------------------------------
기능을 재사용 가능하도록 모듈화

--DB 함수는 반드시 하나의 값을 리턴합니다.

--단일행함수
--그룹함수, 집계함수, multiple row function
--분석함수 , window함수

단일행 함수 - character function
              number function
              date function
              conversion function
              null처리 함수와 조건처리함수

select length('korea'), length('대한민국'), lengthb('korea'), lengthb('대한민국')  
from dual;

--substr(문자열column, start offset, length)
select  substr('hello world' , 7, 3 ), substr('hello world' , 7  )
        , substr('hello world' , -5, 3 )
from dual;

--replace(문자열column, source문자열, target문자열)
--translate(문자열column, source문자열, 한문자단위로매핑해서변환문자열)
select  replace('hello world' , 'world', 'sql' )
from dual;

select  translate('hello world' , 'world', 'sql73' )
from dual;

--instr(문자열column, search문자열, start offset, n번째 위치)
SELECT instr('CORPORATE FLOOR','OR') , instr('CORPORATE FLOOR','OR', 1, 1) 
       , instr('CORPORATE FLOOR','OR', 3, 2)
       , instr('CORPORATE FLOOR','or') 
FROM DUAL;

select ename, sal, lpad(ename, 10, '*'), rpad(sal, 10, '$')
from emp;

#단일행함수는 nested할 수 있습니다.
--rtrim, ltrim, trim : 공백제거 또는 왼쪽 , 오른쪽에 문자를 제거
SELECT length('  hello  world  '), length( trim('  hello  world  '))
from dual;  --공백 제거 (왼쪽, 오른쪽)

SELECT  ltrim('hello  world', 'he'), rtrim('hello  world', 'rld')
from dual;

SELECT  trim(leading 'h' from 'hello  world'  ), trim( trailing 'd' from 'hello  world')
from dual;

SELECT REGEXP_COUNT('apple, banana, mango, melon', '[a-z]+') AS word_count
FROM dual;

SELECT REGEXP_COUNT('My ID is 1234 and my code is 9876', '\d+') AS number_count
FROM dual;

SELECT REGEXP_INSTR('abc123def456', '\d+') AS first_digit_pos
FROM dual;

SELECT REGEXP_REPLACE('abc123def456', '\d+', '###') AS replaced_text
FROM dual;

-------------------------------------------------------------------------
SELECT POWER( 2, 10) 결과1, CEIL (3.7) 결과2, FLOOR (3.7) 결과3
from dual;

select  sign(1000), sign(-2000), sign(0)
from dual; 

SELECT round(123.4567, 2) , round(123.4567, 0) , round(123.4567), round(12345.67 , -2)
from dual;

SELECT trunc(123.4567, 2) , trunc(123.4567, 0) , trunc(123.4567), trunc(12345.67 , -2)
from dual;

--함수는 select절, where절 사용 가능
select mod(3, 2) 
from dual;

Q> 사원번호가 홀수인 사원 정보만 검색 (emp)
select  empno, ename
from emp
where mod(empno, 2) = 1;

datetime function -----------------------------------------------------------------
alter session set nls_date_format='YYYY/MM/DD HH24:MI:SS';

select  sysdate, systimestamp, current_date, current_timestamp, sessiontimezone
from dual;
-- current_xxx는 세션에 설정된 timezone기반의 현재 시간을 반환
-- sysdate, systimestamp 는 OS에 설정된 현재 시간 반환

alter session set time_zone='+3:00';

select  sysdate, systimestamp, current_date, current_timestamp, sessiontimezone
from dual;

alter session set time_zone='+9:00'; --'Asia/Seoul'

날짜 데이터로부터 년도, 월, 일, 요일, 시간 ...등 추출 (문자 데이터로 추출, 수치 데이터 추출) : to_char(), extract()

select  to_char(sysdate, 'YYYY'),  extract(year from sysdate)
from dual;

alter session set nls_language=american; 

select  to_char(sysdate, 'MONTH'),  extract(month from sysdate)
       , to_char(sysdate, 'MON'), to_char(sysdate, 'MM')
from dual;

select  to_char(sysdate, 'DAY'),  extract(DAY from sysdate)
       , to_char(sysdate, 'D'), to_char(sysdate, 'DDD') , to_char(sysdate, 'DY')
from dual;

select  round(to_date('24/06/15', 'RR/MM/DD'), 'MONTH')
        , round(to_date('24/06/16', 'RR/MM/DD'), 'MONTH')
        , round(to_date('24/06/15', 'RR/MM/DD'), 'YEAR')
        , round(to_date('24/07/15', 'RR/MM/DD'), 'YEAR')
from dual;

select  trunc(to_date('24/06/30', 'RR/MM/DD'), 'MONTH')
        , trunc(to_date('24/06/16', 'RR/MM/DD'), 'MONTH')
        , trunc(to_date('24/12/01', 'RR/MM/DD'), 'YEAR')
        , trunc(to_date('24/07/15', 'RR/MM/DD'), 'YEAR')
from dual;

select  months_between(sysdate, hiredate), add_months(sysdate, 6)
from emp;

select  last_day(to_date('2012/02/01'))
       , last_day(to_date('2100/02/01'))
       , last_day(sysdate)
from emp; 

4.
  selec *
  from emp
  where  substr(ename, 1, 1) >'K' and substr(ename, 1, 1) <'Y';
6. 
  select  ename, instr(ename, 'L', 1)
  from emp;
7.
  select job , ltrim(job, 'A'), trim(leading 'A'  from job),
         sal , ltrim(sal, '1'),  trim (leading '1'  from sal)
   from emp
   where deptno = 10;

```