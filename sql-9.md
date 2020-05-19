```sql
--3) 교수에 대한 전체 자료를 다음과 같은 형식으로 출력 
--(단, 모든 교수들에 대해 출력되도록 한다)
--이름	학생수 학생_성적평균 A_학점자수	B_학점자수	C_학점자수	D_학점자수
--심슨     	2	    79	         1	        0	        0	        1
--허은	    2	    83	         0	        1	        1	        0
--조인형    1	    97	         1	        0	        0	        0


select p.NAME AS 이름, count(s.NAME) AS 학생수, 
       avg(nvl(e.TOTAL,0)) AS 학생_성적평균,
       count(decode(substr(h.GRADE,1,1),'A',1)) AS A, -- 학생 수를 세기 위해 임의의 조건을 설정해 count
       count(decode(substr(h.GRADE,1,1),'B',1)) AS B, 
       count(decode(substr(h.GRADE,1,1),'C',1)) AS C,
       count(decode(substr(h.GRADE,1,1),'D',1)) AS D
  from PROFESSOR p, STUDENT s,
       EXAM_01 e, HAKJUM h
 where p.PROFNO = s.PROFNO(+)
   and s.STUDNO = e.STUDNO(+)
   and e.TOTAL between h.MIN_POINT(+) and h.MAX_POINT(+)
 group by p.NAME;
```

-------------------------------------------------------------------------------------------

## 데이터 베이스 관리
#### 1. 구조 변경(DDL): auto commit 하는 순간 즉시 반영(위험) 

##### 1) 테이블 생성 - create
```sql
create table test(
no1 number(10), -- 10자리 가능
no2 number, --자리수 더 많이 할당
col1 varchar2(10), 
col2 char(10),
col3 date);

** varchar2와 char 타입 비교
varchar2(10), char(10)으로 만들어진 각 컬럼에 'a'입력하면 각각 'a','a         ' 이렇게 저장됨
```

##### 2) 테이블 복사(CTAS): 백업목적으로 사용, 제약조건은 복제 안됨
```sql
create table emp_back
as
select * from emp ; -- as뒤에 바로 select절 적용

create table emp_back2 
as
select ename, sal from emp;  -- 명시된 컬럼만 백업테이블로 만들기

create table emp_back3
as
select ename, sal from emp where deptno=10; -- where절 사용가능

create table emp_back4
as
select ename, sal from emp where 1=2; -- 데이터 없이 구조만 복사

select * from emp_back; 
```
##### 3) 변경 - alter 
```sql
--1) 컬럼 추가
alter table emp_back2 add (no1 number); --null로 채워짐
alter table emp_back2 add (no2 number default 10); -- default 값으로 모두 채워짐

--2) 컬럼 이름 변경
alter table emp_back2 rename column no2 to no22; --alias와는 달리 영원히 변경 

--3) 테이블(객체) 이름 변경
rename emp_back2 to emp_back22;

--4) 테이블 컬럼 속성(데이터타입, 사이즈, default값) 변경
desc emp_back22;

alter table emp_back22 modify (ename varchar(20));

alter table emp_back22 modify (ename varchar(8)); --데이터의 손실이 없게끔 변경이 가능

alter table emp_back22 modify (no22 varchar2(0)); --6:00

alter table emp_back22 modify (no2 varchar2(0));

* 데이터가 있다면 형변환 불가

--[참고] : 컬럼 속성 확인
select *
from user_tab_columns
where table_name='EMP_BACK22'; 
--DEFAULT값 의미: (컬럼 생성시에도 가능) 데이터 값이 들어오지 않는 경우 자동으로 부여되는 값
```
##### 4) 삭제 - drop, truncate
```sql
--5) 컬럼 삭제 
alter table emp_back22 drop column no22;
--뒤에 cascade constraints를 쓰면 제약조건 때문에 삭제가 안될 때 제약조건 포함해 모두 삭제한다
--그럼에도 불구하고 사용하는 이유는?
--delete는 한건한건 지우면서 기록을 하기 때문에 속도가 느리다 하지만 truncate는 기록하지 않기
--때문에 빨리 날라감

--6) truncate: 구조는 남고 전체 데이터만 날라감, 완전 날라가서 제일 위험함
-- (일부행을 지우고 싶다면 delete, 이건 dml로서 commit 전에 발견하면 롤백가능)

--7) drop: 테이블 자체가 모두 없어짐 단, 휴지통으로 가기 때문에 복구는 됨
drop table emp_back22;
select * from user_recyclebin;

--[참고] 휴지통에서 빼오기 
flashback table "BIN$p2xAH6tPQhippX13egyJzA==$0"
to before drop;
 
-- truncate는 구조변경은 아니더라도 auto commit이 되기 때문에 DDL로 분류
```
------------------------------------
#### [참고 - 아카이브 모드] 
-- 대체적으로 DB자체는 백업을 해놓는다 데이터베이스는 복구모드에서, 일어나는 모든 일들을 기록하는 시스템이 있어 사용자가 사용한 명령어들을 다 기록해 놓음 
-- 만약 사용자가 12:12에 truncate를 날렸다면 그날 백업된 db에 덧대어 12:11까지 변경되었던 모든 내용들을 다시 복구할 수 있지만 시간이 좀 걸린다. 
-- 이것을 아카이브 로그모드라고 한다. 즉, 복구에 대한 준비가 잘되어있다는 것이다.

------------------------------------------------------------------------------------

#### 2. 데이터 변경(DML): 수동적으로 commit을 해줘야 된다 안그러면 롤백됨
- 삽입 - insert, 수정 - update, 삭제 - delete, 병합 - merge

##### 1. insert
```sql
CREATE TABLE DEPT_BACK AS SELECT * FROM DEPT;

SELECT * FROM DEPT_BACK;

--1)전체 컬럼에 데이터 삽입: 컬럼 명시x
insert into dept_back values(50,'tset','seoul');

--2)일부 컬럼에 데이터 삽입: 컬럼 명시
insert into dept_back(deptno, dname) values(50,'tset1');--마지막 열은 null임
--명시하지 않은 컬럼이 not null 특성이 존재할 경우 명령어 실행 x

--3) 테이블 데이터를 입력하는 경우
insert into dept_back 
select * from dept where deptno=10;

insert into dept_back(deptno, dname) 
select deptno, dname from dept where deptno=10; 

insert into emp_back22(ename, sal)
     values('abcd',2000); -- 나머지 컬럼은 null값 또는 default값 할당
--default값의 반영은 수정되는 시점부터 적용
```
##### 2. delete: 행단위 삭제, 내부적 로딩에 의해 일부 복구가 가능하나 삭제작업 오래 걸림
```sql
--delete from 테이블명
--where 조건(서브쿼리 가능);

--예제) STUDENT에 대한 백업 생성 후 서진수와 같은 학년 학생 정보 삭제
create table STUDENT_back
as
select * from STUDENT;

delete from STUDENT_back
where grade=(select grade
              from STUDENT_back
              where name='서진수');
              
rollback; -- 취소됨

--연습문제) 학년별 평균키보다 큰 학생 삭제
delete from STUDENT_back s1
where height>(select avg(height)
                from STUDENT_back s2
                where s1.GRADE=s2.GRADE);
```

-----------------------------------------------------------------------------------------------
#### [ 참고 : 데이터 복구 방법 ]

```sql
delete from STUDENT where grade = 4;
commit;
select * from STUDENT;

--1. 시점 데이터 조회
select *
  from STUDENT 
  as of timestamp(systimestamp - interval '10' hour);

--2. 원래 테이블에 데이터 입력
insert into STUDENT
select *
  from STUDENT 
  as of timestamp(systimestamp - interval '10' hour)
 where grade = 4;
 
commit;
```
------------------------------------------------------------------------------------------------

#### [참고: 백업테이블 만들기]
```sql
select * from tab; 
=>
TNAME                          TABTYPE  CLUSTERID
------------------------------ ------- ----------
BIN$1K3NzFXhTxW/qVVQntqbXA==$0 TABLE
BIN$GOuoXr36RMaMXEdNrn5VEg==$0 TABLE
BIN$KPGmOYd+RJOvY3xvaeS7Zg==$0 TABLE
BIN$PJOP4nVCTIOsTPTW8EWlHQ==$0 TABLE
BIN$X646dPr1ShGuiniaxZmpWg==$0 TABLE
BIN$zUyBxGqtRBioet69cPrW4A==$0 TABLE
BONUS                          TABLE
CAFE_JUMUN                     TABLE
CAFE_PROD                      TABLE
CAFE_PROD2                     TABLE
...
---------------------------------------------------

select 'creat table'||tname||'_bac as select * from '||tname||';'
from tab
where tname not like 'bin%';
```
---------------------------------------------------------------------------------------------
#### [ 참고 - transaction]
500만원 상대방에게 보내는 업무를 생각하면...
**내 계좌에서 500만원 있는지 조회 > 상대방 계좌 있는지 확인 > 내 돈 뺴서 입금 > 계좌 정보 업데이트**
위와 같은 하나의 업무단위를 transaction이라 함. 종료언어로서 commit과 rollback이 있음

***rollback 지점 설정**
```sql
insert into...
savepoint a;

update...
savepoint b;

delete...
savepoint c;
-----여기까지 업무를 수행하다가 다음과 같은 명령어를 실행시킨다면..?
alter table ... -- DDL언어는 auto commit이 되므로 그 전에 했던 명령어들도 commit 되어버림
                -- 따라서 그러한 상황을 방지하고 원활한 업무를 위해 각 지점마다 savepoint 설정

rollback to savepoint b; -- b 포인트로 돌아감
```
--------------------------------------------

##### 3. update: 특정 데이터의 특정컬럼 단위 수정
```sql

 --연습문제) STUDENT에서 원본과 동일하게 생성 후 성별컬럼을 생성한 뒤 각 학생의 성별을 남자,
 --          여자로 수정
 create table STUDENT2
 as select * from STUDENT;
 
alter table STUDENT2 
        add (성별 varchar(2));
 
update STUDENT2
   set 성별=decode(substr(jumin,7,1),'1','남','여');

commit;