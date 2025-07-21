# 3강

# 복습

# 인덱스 사용

3가지 경우를 제외하고는 unique index가 만들어지지 않는다

## 세팅

```sql
alter session set statistics_level = all;

-- SQL 문장

select * from table(dbms_xplan.display_cursor(null,null,'allstats last'));
```

- 실행계획을 라이브러리 캐시에 저장하는 것들을 parameter라고 하는데, 그걸 조절할 수 있음

## FULL SCAN vs INDEX RANGE SCAN

```sql
select *
from customers
where cust_id between 70 and 800000;

```

- 실제 블록 위치의 퍼짐 정도에 따라서 buffer 수가 달라짐.
- 인덱스 검색이 항상 좋은 것은 아니다.
- 책 예시(P.76) 참고

### FULL TABLE SCAN이 요즘에는 BUFFER CACHE를 거치지 않고, 바로 PGA에서 가져오다보니, 상당히 빨라짐

## Clustering Factor

```sql
SELECT index_name, blevel, leaf_blocks, distinct_keys, clustering_factor
FROM user_indexes
where table_name = 'CUSTOMERS'
order by 1;

mehtod.opt => fro all colummns size auto
-- 전부 다 필요는 없다.

```

- clustering_factor가 높을 수록 데이터가 여러 블록에 퍼져 있다는 것 → 많이 뭉쳐 있다? 그 반대
- unique하면 clustering factor가 높다?
- 힌트를 주는데, 알리아스가 있으면 반드시 알리아스를 써야한다. 풀 네임 쓰면 안됨

## 인덱스를 사용하는 SQL 작성

### 인덱스를 사용하지 못하는 경우 1) 조건식에 WHERE이 없는 경우

```sql
select first_name, salary
from employees
where department_id = 20;

select department_id, count(*)
from employees
group by department_id;

select department_id, count(*)
from employees
group by department_id
having avg(salary) >= 3000;

select department_id, count(*)
from employees
where salary >= 10000
group by department_id;

select department_id, count(*)
from employees
group by department_id
having salary >= 10000; -- error

```

- select에 특정 행이 없어도, 전부 스캔해야함
- count(*)이 5보다 큰 것만 모으려면: where 절에 못 씀(실행 순서 떄문에)

→ having 절에는 그룹 함수만 오도록

→그럼 select 절에 안쓰인 것도 올 수 있나? 가능

→ 일반 칼럼도 가능은 하지만 비 권장

→ 그러나 아무것은 안되고, select에서 쓴 것만 having에서 사용 가능

```sql
select cust_city, avg(cust_credit_limit)
from customers
where cust_city in ('Lubin', 'Emmen')
group by cust_city;

select cust_city, avg(cust_credit_limit)
from customers
group by cust_city
having cust_city in ('Lubin', 'Emmen');
```

- having 절은 index를 태울 수 없다.
- group by 할 필요가 없는 것 까지 grouping 해버린다.

→ 즉, 일반 칼럼에 대한 조건은 where에 하고

→ having 절에는 집계 함수만 하라고.

## 테이블 변경

```sql

update employees
set hire_date = sysdate - 8440
where employee_id >= 200;

commit;

-- 안나옴
select employee_id, first_name, hire_date
from employees
where hire_date = to_date('2002/06/07','yyyy/mm/dd');

-- 방법 1. 범위 비교
SELECT employee_id, first_name, hire_date
FROM employees
WHERE hire_date >= TO_DATE('2002/06/07','yyyy/mm/dd')
  AND hire_date <  TO_DATE('2002/06/08','yyyy/mm/dd');
  
-- 방법 2. TRUNC
SELECT employee_id, first_name, hire_date
FROM employees
WHERE trunc(hire_date, 'dd') = TO_DATE('2002/06/07', 'yyyy/mm/dd');

-- 방법 3. to_char
SELECT employee_id, first_name, hire_date
FROM employees
WHERE to_char(hire_date, 'yyyymmdd') = '20020607';  
  
SELECT employee_id, first_name, to_char(hire_date, 'yyyy/mm/dd hh24:mi:ss')
FROM employees
```

- 시분초가 00시 00분이 아니라서 범위로 해줘야 함!!!
- 왼쪽 인자인 ‘2002/06/07’은 00:00:00으로 되어 있기 때
- trunc 2번째 인자 : 거기까지 나오게 하라.

### 인덱스를 만든다면?

```sql
create index empl_hiredate_ix on employees(hire_date);

select cust_id, cust_first_name, cust_city
from customers
where cust_last_name = 'king' -- 안나옴. 대소문자 구분
```

- 이거 해놓고 위에꺼 해놓으면

→ 1번만 index 탄다

→ 조건에서 인덱스에 변형을 주면 작동 안함!!

- 인덱스는 대소문자가 구분되어있는데, 이거 맞춰주려고 upper하고 그러면 인덱스 활용이 안된다.

→ 함수가 적용된 값 자체를 인덱스로 만들 수도 있다!

```sql
select *
from customers
where cust_city = 'Lublin';

select *
from customers
where cust_city = 'LUBLIN';

select *
from customers
where upper(cust_city) = upper('lublin');

create index f_custs_city_ix on customers(upper(cust_city));

select *
from customers
where upper(cust_city) = upper('lublin');
```

- 이렇게 Function_Based_Index를 쓰면 Index Search를 수행한다.
- 따라서 해결 방법은 두 가지. 1.  범위 서치 2. Function based index를 쓴다.

## IS NULL 비교

### 세팅

```sql
select count(*)
from customers
where cust_email is null;

update customers
set cust_email = null
where cust_id in (3228, 4117, 6784);

commit;

create index cust_email_ix on customers(cust_email);

select *
from customers
where cust_email is null; -- full access
```

- 몇몇 email 값을 null로 만들어주자
- distinct 값이 많으므로 인덱스를 만들어주자.
- B* tree에는 null이 저장이 되지 않는다..

→ 해결을 위해 2가지 방법이 있다.

1. 업무적 협의
2. Function Based Index

→ Funtion Based Index 방법으로 nvl(cust_email, ‘????’)을 인덱스로 만들고, 찾을 때 ‘????’로 찾으면 된다.

```sql
create index f_custs_email_ix on customers( nvl(cust_email, '????'));

select *
from customers
where nvl(cust_email,'????') = '????';

```

## Bitmap Index

전체 비율 중에 ~~조건이 몇 퍼센트냐(조건부 확률)에서 많이 씀

튜닝이 가능한 PGA → Hash join

남자라고 대답한 사람들로 비트맵 만듬 1 0 1 0 0 0 … (에다가 시작 rowid, 종료 rowid?)

흰 색깔에 대해서, 은색에 대해서..

and or로 연산하면 됨.

운영계에서는 사용 안하고, 분석계에서는 사용함

→ Index 쪽의 dml 때문에..

→ index는 update의 개념이 없고, delete와 insert라서 비트맵 새로 짜야함

→ 의사결정을 위한 데이터베이스에서만 사용된다.

function based index가 내부적으로 bitmap index로 전환되서 처리될 수 있음 → 지피티가 아니래

## WHERE절에서의 묵시적(암시적) 변환

```sql
select *
from tid
where lnid = 11100; -- full access

select *
from tid
where lnid = '11100'; -- index access

select *
from emp
where hiredate='81/02/20';
```

- 기본적으로 char → number, date로 바꾸려고 함.. 그래서 인덱스에 to_number가 적용되어버려

→ 해결 방법 : 선제적으로 문자 데이터로 바꿔주기 

→ || (연결 연산자), LIKE, 문자 함수

- 왼쪽은 가급적으로 변형되지 않는 형태로 가자.

→ 주민 번호의 경우 숫자? 패턴 매칭을 한다면 문자가 더 편할 수도 있다.

→ 이러면 문제가 생길 수도 있다.

### 묵시적 변환, % 잠깐

```sql
select *
from emp
where hiredate like '1980%';

select *
from emp
where empno like '778%';
```

- 묵시적 변환 때문에 full access

## is not null 비교

```sql
select *
from customers
where cust_email is not null;

select count(*)
from customers
where cust_email is not null;

```

- 자료 수가 많아서 full access가 기본적으로 유리함

→ 그런데 count(*)라면?

→ FAST FULL SCAN을 하게 됨(leaf block만 읽으면 되는 경우에, 순차적이 아니라 그냥 원하는 위치부터 한번에 읽어버림)

- RBO는 인덱스를 못타고, CBO는 탈수도 있다? 부정형은 인덱스를 자 못탄다?

```sql
select *
from customers
where cust_id = 100866; --index scan

select *
from customers
where cust_id != 100866; --full scan

-- 샘플 테이블 만들기
create table sample(no number, name varchar2(30));

-- PL/SQL까지
begin
    for i in 1..200000 loop
        insert into sample
        values(1, to_char(i));
    and loop;
    commit;
end;
/

begin
    for i in 2..1000 loop
        insert into sample
        values(i, to_char(i));
    end loop;
    commit;
end;
/

-- 통계 정보 만들기
exec dbms_stats.gather_table_stats('sqlt01', 'sample');

-- no에 대해서 index 만들기
create index samp_no_ix on sample(no);

select *
from sample
where no = 1;

exec dbms_stats.gather_table_stats('sqlt01', 'sample', method_opt=>'for columns size auto no');

select *
from sample
where no = 1;

select *
from sample
where no != 1;

select *
from sample
where no >= 2;

```

- 여기서 부정형은 너무 많아서 인덱스를 안탄건지? 탈 수 없어서 안 탄 건지?
- 1이 대부분인데, index scan을 한다? full scan이 유리한데.. 통계 정보를 만들었는데, 그게 정확하지 않아서 그런가?

→ 다시 통계 정보를 만들면 Full Scan 한다.

→ 그렇다면 부정형을 테스트 해본다면? 극소수니까 index 타는게 맞아야하는데, 만약 아니라면(full scan이라면) 부정형이라는게 인덱스를 못타는 구조인 것이다.

→ Full scan 뜸;;

→ 부정이 들어오면 거의 index를 못 탄다. ≥2로 바꿔주면 됨.

- 힌트는 보통 cost based optimizer로 돌게 되는데, /*+ rule */ 하면 rule로 돌게 되고, rule로 돌게 되면, index를 탈 수 없게 된다.
- rule 태그를 빼게 되면, index full일 수도, fast full일수도 있다. not null인 것들은 전부 leaf에 있을 것이니까

## 와일드 카드

- 앞을 알 수 있으면 index searching이 가능한데, 앞이 가려져있으면 근본적으로는 불가능

→ 그런데 function based로 하면 가능

### Case by Case

1. 16~~
2. ~~508
3. xx바xxxx
4. ~~88~~

reverse, substr 등등 활용 가능. 4번은 어려움

2,3번 혼합도 가능한데, 하나의 칼럼당 function index 최대 2개?

```sql
select *
from customers
where cust_first_name like 'Ab%'; -- index scan

select *
from customers
where cust_first_name like '%enn'; -- full scan

select *
from customers
where substr(cust_first_name, -3,3) = 'enn'; -- 아직은 full scan

create index f_custs_first_name_ix on customers(reverse(cust_first_name));

select *
from customers
where reverse(cust_first_name) like 'nne%'; -- index scan
```

## 정리

- 조건식은 having이 아니라, where?
- 좌측에 맞게 function 조절
- is null은 죽어도 index에 저장이 불가능하므로, 형태를 바꿔서 저장
- 암시적 묵시적 데이터 변환은 약속을 잘하거나 function으로 해결
- is not null도 function으로 해결

## 결합 인덱스

sales에서 time_id, prod_id 결합 인덱스의 비밀..

time_id가 더 distinct한데.. 왜 prod_id를 앞에 두는게 빠른가?

→ time_id가 범위 서치이기 때문

→ 25/07/07이라고 하면… 25/07/08 사이에는 수 많은 칼럼들이 있다. 그래서 prod_id가 A라고 하면 00시00분에서 다 찾았다고 00시01분에 A가 없다고 할 수 없음. 결국 다 서치하게 됨.

→ 선행 칼럼은 가급적 =으로 비교 되는 것으로 할 것

# SQL Workarea 최적화

정렬은 PGA에서 수행, 메모리 크기에 크게 영향을 받는다.→ 튜닝이 가능/불가능한 메모리 영역?

distinct , group by, order by, 조인 작업, sort aggregate(이건 실제 정렬은 아님)

### Group by

```sql
select department_id, count(*)
from employees
group by department_id
order by department_id;
```

- 예전에는 결과값을 정렬해서 리턴해줬는데, 요즘엔 그렇지 않다.

→ 예전엔 먼저 정렬해놓고, distinct만 남기면서 count 값 증가시키는 방식이었다.

→ HASH GROUP BY 라는 Hidden Parameter가 있는데, 예전에는 False였는데, True로 바뀜.

→ ORDER BY 요즘엔 써야

→ 실행 계획을 보면 SORT GROUP BY로 바뀐다.(예전 기법, 느림)

```sql
select cust_id, sum(amount_sold)
from sales
group by cust_id
order by cust_id;

select *
from	(select /*+ no_merge */ cust_id, sum(amount_sold) sum_amt
	from sales
	group by cust_id)
order by cust_id;
```

- SORT GROUP BY로 수행되는데, HASH GROUP BY로 처리되도록 조작해줄 것.
- 일단은 성능상 차이를 보이지 않는데.. 흐음..

```sql
select cust_id, cust_credit_limit, cust_street_address
from customers
order by cust_id;

select /*+ first_rows */
cust_id, cust_credit_limit, cust_street_address
from customers
order by cust_id;

select /*+ index(customers custs_id_ix) */
cust_id, cust_credit_limit, cust_street_address
from customers
order by cust_id;

select /*+ first_rows */
cust_id, cust_credit_limit, cust_street_address
from customers
order by cust_credit_limit; -- full로 탄다

select /*+ first_rows index(customers CUSTS_CREDIT_IX)*/
cust_id, cust_credit_limit, cust_street_address
from customers
order by cust_credit_limit; -- 역시나 full

```

select *

from tab1

where a = ‘텀블러’

order by b;

라고 한다면.. a + b의 인덱스를 만들자. sort로 하면 all_rows로 해야해서 너무 느리다.

→ index access를 통해서 대체할 수 있다.

→ 사실 굳이 따지면 정렬해서 후루룩 넘겨주는게 더 빠를 수 있음

→ 다만, 온라인 서비스에서는 all_rows로 하면 안되고, first_row 방식으로 실행 계획을 짜야 한다!!

→ 위 두 커리를 보면 전체적으로는 첫번째가 더 나은 것을 알 수 있음. 하지만 first_rows를 할 수 있다는 장점!!

→ batch 성 업무라면 첫번째 방식으로 가야한다..!

→ index 힌트를 주는 경우에?

→ index를 쓰더라도 order by는 적어두는 것이 좋다.??

(당장은 없어도 작동하지만.. 인덱스 힌트 빠뜨리면 안되어서)

→ 왜 밑 두 케이스는 full로 되냐?

→ nullable이라서

→ 실제로 null이 없더라도, 테이블에 대한 정보에서 null을 허용한다면 index searching을 할 수 없다.

→ 정렬을 index access로 대체하고 싶다면. where 절과 order by 절에 대해서 결합 인덱스를 만들어야 한다.

where a = 10

order by b, c;

1. a+b+c : 결합 index
2. 최소 1개 칼럼에는 not null 이어야.

order by에서 index를 활용하려면? first_rows와의 연관성?

## PGA

sort, hash, bitmap, index는 튜닝이 가능??

메모리 크기를 변경했을 때, 정렬이 어떻게 바뀔까??

→ 줄이니까 Used_Tmp, 즉 temporary 공간 활용

→ 정렬의 속도는 메모리 크기가 좌지우지 한다.

→ 그전에 메모리 영역을 개별 관리하는게 좋지 않다

→ 자동의 기법을 써야.. manual이 아니라 auto로, pga_aggragate_target = 500M

→ 처음에 적게 잡더라도, 자동으로 조정해준다.

지금까지 기반으로 내일부터 튜닝 실습..

PGA
├── 튜닝 불가능 영역
│   ├─ Session memory
│   ├─ Persistent area
│   └─ UGA (in dedicated server)
│
└── Workarea (튜닝 가능 영역)
├─ Sort area
├─ Hash area
├─ Bitmap merge area
└─ Sort aggregate area

| 구분 | 설명 |
| --- | --- |
| ① **Session Memory** | 각 세션을 위한 고정 메모리 (로그인 시 할당, 세션 유지 중 지속됨) |
| ② **Persistent Area** | 커서 및 SQL 실행을 위한 고정 메모리 (예: PL/SQL 패키지 변수 저장) |
| ③ **UGA in PGA (Dedicated Server)** | 세션 변수 등 사용자 메모리, 전통적으로는 SGA에 있던 게 dedicated server에서는 PGA로 |