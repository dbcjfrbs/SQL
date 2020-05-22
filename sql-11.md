#### * 제약조건정보를 담고 있는 테이블 
##### user_constraints / user_cons_columns
-- (제약조건이 있는 테이블만 보여짐)
```sql
select u1.TABLE_NAME, u2.COLUMN_NAME, u2.CONSTRAINT_NAME, u1.CONSTRAINT_TYPE,
       u1. delete rule -- delete cascade 여부 보여줌(밑에나옴)
from user_constraints u1, user_cons_columns u2
where u1.CONSTRAINT_NAME=u2.CONSTRAINT_NAME; 
```
- user_constraints : 테이블이름, 제약조건타입, 제약조건이름 존재
- user_cons_columns : 컬럼이름, 제약조건이름 존재
--------------------------------------------------------
---------------------------------------------------------


#### * foreign key 존재할 때 primary key 컬럼 삭제
##### 1. cascade constraint
```sql
1) 테이블 복사
create table emp_t1 as select * from emp;
create table dept_t1 as select * from dept;


2) 제약조건 넣기
alter table dept_t1 add constraints deptt1_deptno_pk 
            primary key(deptno);

alter table emp_t1 add constraints empt1_deptno_fk
            foreign key(deptno) references dept_t1(deptno);
            

3-1)제약조건 삭제불가 - 부모키를 먼저 삭제할 수 없음
alter table dept_t1 drop column deptno; 

3-2) cascade constraint구문 : 제약조건 걸린 컬럼과 연결된 제약조건 모두 삭제           
alter table dept_t1 drop column deptno 
      cascade constraint; 


4) 제거확인
select * 
from user_constraints
where table_name='dept_t1';
```

##### 2. on delete cascade
```sql
1) 테이블 복사
create table emp_t2 as select * from emp;
create table dept_t2 as select * from dept;

2) on delete cascade를 포함한 제약조건 넣기
* on delete cascade: reference key 데이터 지울 때 foreign key 데이터도 함께 삭제

alter table dept_t2 add constraints deptt2_deptno_pk 
            primary key(deptno);

alter table emp_t2 add constraints empt2_deptno_pk 
            foreign key(deptno) references dept_t2(deptno)
            on delete cascade;


3) deptno=10 데이터 삭제 시 부모, 자식 테이블 모두 해당 데이터 삭제됨
delete from dept_t2 where deptno=10;
select * from emp_t2;
```

##### 3. on delete set null 
```sql
1) 테이블 복사
create table emp_t2 as select * from emp;
create table dept_t2 as select * from dept;

2) on delete set null 를 포함한 제약조건 넣기
* on delete set null : reference key 데이터 지울 때 foreign key 데이터 널 처리

alter table dept_t2 add constraints deptt2_deptno_pk 
            primary key(deptno);
                  
alter table emp_t2 add constraints empt2_deptno_pk 
            foreign key(deptno) references dept_t2(deptno)
            on delete set null;


3) deptno=20 삭제 시 부모테이블 삭제, 자식테이블 null처리 
delete from dept_t2 where deptno=20;
select * from emp_t2;
```
----------------------------------------------------
---------------------------------------------------
#### 제약조건 관리
##### (데이터 insert 시 제약조건이 있으면 속도저하)

##### 1. disable : 제약조건 삭제 대신 기능 유지
```sql
* 문법
alter table 테이블명 disable/enable [옵션]
                    constraint 제약조건명;
* 옵션
validate : disable 후에도 무결성 체크 
novalidate(기본) : 무결성 체크하지 않음

1) 테이블 생성
create table emp_t3 as select * from emp;
alter table emp_t3 add constraints empt3_empno_pk
            primary key(empno);


2) 무결성위배로 인한 실행불가(같은 empno존재)
insert into emp_t3
select * from emp where empno=7369; 


3) disable 을 통한 제약조건 비활성화
alter table emp_t3 disable constraint empt3_empno_pk;


4) 같은 empno 입력 가능 
insert into emp_t3
select * from emp where empno=7369; 
```

##### 2. enable: 제약조건 기능 활성화
```sql
* 옵션
validate(기본) : enable 시 기존 데이터 무결성 체크
novalidate : 무결성 체크하지 않음

5-1) enable 을 통한 기존 데이터 무결성 체크(validate 옵션이 default이기 때문)
alter table emp_t3 enable constraint empt3_empno_pk; 
=> 실행불가

5-2) novalidate 옵션을 통한 무결성 체크 안함, 그런데 실행불가
alter table emp_t3 enable novalidate constraint empt3_empno_pk;
=> pk나 unique 만들면 색인하기 좋다고 인지해 자동으로 index가 생성되는데 index에서도
   검사해서 오류나기 때문??


6) 제약조건 활성화여부 확인
select TABLE_NAME,
       CONSTRAINT_NAME,
       STATUS,          --disable 여부
       VALIDATED        --validate여부
from user_constraints
where CONSTRAINT_NAME='EMPT3_EMPNO_PK';
```
##### ** not null 제약조건에 대한 enable novalidate 
** (위의 primary key 제약조건에 대해서는 enable novalidate에 대한 실행이 안되었음) 
```sql
7) 테이블 제거 및 not null 제약조건 테이블 생성
drop table emp_t3;
create table emp_t3 as select * from emp;

alter table emp_t3 modify ename
            constraint empt3_nn not null; 


8) ename 컬럼에 데이터 삽입불가 
insert into emp_t3(empno, ename)
            values(9999,null); 
=> 실행불가


9) disable 옵션을 통한 데이터 삽입
alter table emp_t3 disable constraint empt3_nn;

insert into emp_t3(empno, ename)
            values(9999,null);


10) enable 시 기존데이터 체크 시도해서 오류
alter table emp_t3 enable constraint empt3_nn; -- 실행불가  


11) novalidate 를 통해 이후에는 제약조건에 해당하는 데이터 삽입 불가
alter table emp_t3 enable novalidate constraint empt3_nn; 
-- 기존 데이터 체크시도 안함, 이후 데이터만 체크시도
insert into emp_t3(empno, ename)
            values(9998,null);
             --입력불가, 이후 데이터는 무결성 체크하므로
```
-------------------------------------------------------
------------------------------------------------------
## **기타 오브젝트
### 1. 뷰(VIEW)
- 실제 테이블은 아니지만 테이블처럼 활용가능
- 저장공간 할당 X
- 뷰를 생성하면 뷰에 연결된 쿼리가 그때마다 실행, 테이블 대신 만든다
- 뷰를 통한 dml작업도 가능
- 뷰 생성하려면 create view 권한 필요
- 단순뷰, 복합뷰, 인라인뷰로 분류
```sql
* 문법
create [or replace] view 뷰이름
as
subquery;
* [or replace 옵션]
이미 뷰가 존재한다면 덮어쓰기

* 뷰 삭제
drop view 뷰이름

* 뷰 조회
select * from user_views='뷰이름';

[뷰 생성/조회/수정]

1. scott 계정에 뷰 생성 권한 부여 - (system)계정에서 수행
grant dba to SCOTT; -- dba는 DB의 모든 권한의 묶음을 의미


2. 뷰 생성
create view emp_view1
as
select empno, ename
  from emp;

* or replace 옵션을 사용한 덮어쓰기
create or replace view emp_view1 
as
select empno, ename, sal
  from emp;


3. 뷰를 통한 조회
select * from emp_view1;


4. 뷰를 통한 수정

- 뷰를 통해 실제 테이블을 삭제하게 된다!
- 뷰는 고정된 객체가 아니기 때문에 원본이 바뀌면 따라 바뀐다!
- 뷰를 불러올 때마다 뷰 생성 쿼리문이 수행되기 때문에 따라 바뀜
- 저장된 데이터를 가져오는 것이 절대 아니다!

delete from emp_view 
where empno=7369; 

select * from emp
where empno=7369; --삭제됨


5. 생성된 뷰 확인
select * from user_views where view_name='EMP_VIEW1';

- 조회할때에는 대문자로 무조건 조회
- sql에서는 컬럼, 테이블이 모두 대문자로 저장되기 때문 
```
```sql
--[연습문제]
--학생의 이름, 학번, 학년, 학과이름, 시험성적을 한 테이블처럼 조회가능하도록 뷰 생성
create or replace view STUDENT_view
as
select name, grade, dname, total 
from STUDENT s, DEPARTMENT d, EXAM_01 e
where s.deptno1=d.deptno and s.studno=e.studno;
```
---------------------------------------------------------
----------------------------------------------------------
## **시퀀스(sequence)
**( 채번 오브젝트 ex) 주문거래번호 )**
- 연속된 숫자를 자동으로 부여하기 위한 객체
- 동시접속자가 많을 때 cache 사용하면 채번부여 쉬움 
```sql
1. 시퀀스 생성
create sequence jumun_seq1
increment by 1
start with 1
maxvalue 10;

create sequence jumun_seq2
increment by 1
start with 1
maxvalue 10
minvalue 1
cycle
cache 2;

*[cache 옵션]
- 채권 획득 세션이 여러개이면 cache는 속도를 좀 더 빠르게 해줌


2. 테이블 생성 
create table jumun(
no  number,
name varchar2(10));

3. 시퀀스를 통한 번호 부여
insert into jumun values(jumun_seq1.nextval,'a');
insert into jumun values(jumun_seq1.nextval,'b');
insert into jumun values(jumun_seq1.nextval,'c');

4. 10회 insert 후에는 에러발생
select * from jumun; 


5. jumun_seq2 은 계속 cycle로 계속 insert 가능
delete from jumun;

insert into jumun
 values(jumun_seq2.nextval,'a');

select * from jumun;
=>
        NO NAME
---------- ----------
         1 a
         2 a
         3 a
         4 a
         5 a
         6 a
         7 a
         8 a
         9 a
        10 a
         1 a
...
```
----------------------------------------------------------
#### *[참고]
- 구버전에서는 테이블 만들면 메모리 최소공간(segment) 할당되었음 
- 신버전부터 빈 테이블이 생기면 데이터가 들어올 때까지 segment 할당x
- 따라서 처음 insert문이 수행되면 segment가 그제서야 생성되며 데이터는 입력되지 않고 2번째 insert문부터 인식되며 출력됨
- 한 번 증가된 sequence값은 delete되어도 리셋되지 않고 다음 번호가 계속 출력됨
--------------------------------------------------------
```sql
1. 시퀀스 생성 확인
select * from user_sequences;

=>
SEQUENCE_NAME                   MIN_VALUE  MAX_VALUE INCREMENT_BY C O CACHE_SIZE
------------------------------ ---------- ---------- ------------ - - ----------
LAST_NUMBER
-----------
JUMUN_SEQ1                              1         10            1 N N         20
         11

JUMUN_SEQ2                              1         10            1 Y N          2
          3
---------------------------------------------------------
*[LAST_NUMBER]  
- 절대 sequence가 가지는 마지막 값이 아님 
- CACHE_SIZE 만큼 미리 메모리에 올려놓은 시퀀스의 마지막 넘버
 다쓰면 다시 미리 설정
- 내부에서 미리 메모리에 할당해 놓은 최종넘버라고 생각!


2. 시퀀스 넘버 확인
1) nextval
SQL> select jumun_seq2.nextval from dual;
   NEXTVAL
----------
         4
SQL> select jumun_seq2.nextval from dual;
   NEXTVAL
----------
         5
.....
- 조회만으로도 넘버가 올라간다!
- 명령전달시 번호 카운팅 되므로 현재 시퀀스 조화할때 사용금지!

2) currval
SQL> select jumun_seq2.currval from dual;
   CURRVAL
----------
         6
SQL> select jumun_seq2.currval from dual;
   CURRVAL
----------
         6
- 시퀀스의 현재 넘버 조회 


--3. 시퀀스 삭제
drop sequence jumun_seq3;
- 한번 늘어난 sequence는 돌아갈 수 없으며 재생성해야 함
```
---------------------------------------------------------
--------------------------------------------------------
## synonym 
(테이블에 대한 별칭을 고정시키는 행위)
- 원래 테이블은 SCOTT.emp처럼 소유자명이 붙음
```sql
1. 생성
create [or replace][public] synonym 별칭
                            for 원본테이블;

- or replace : 재생성 옵션
- public : 시노님을 생성한 유저 외에도 사용가능하게 한다는 의미
- public 사용하지 않으면 본인계정에서만 사용한다는 의미


2. 삭제
drop [public] synonym 시노님명;


3. 조회 
select * 
from all_synonyms
where synonym_name='STUDENT';

- public synonym 생성하게 되면 그 synonym은 공용소유로 바뀜!!
```
------------------------------------------------------
##### [scott 계정의 STUDENT테이블을 hr계정에서 조회하기] 
```sql
1) 권한부여
grant select on scott.STUDENT to hr;

2) hr계정에서 테이블 조회
select * from scott.STUDENT; 

3) 시노님 생성(SCOTT계정에서 수행) 
create public synonym STUDENT
for SCOTT.STUDENT;

4) hr계정에서 테이블 조회
select * from STUDENT;
```
------------------------------------------------------
##### [DB link: 한 DB에서 다른DB와의 연결]
- 타 DB에도 synonym 생성가능
---------------------------
#### * 권한
```sql

1. 권한부여
1) 특정 오브젝트에 대한 오브젝트 권한
grant select, update, delete, insert on 테이블명
      to 유저명;
      
2) 시스템 권한
- 작업과 관련된 권한

grant 권한명 to 유저명
grant create public synonym to hr;

2. 회수 
revoke 권한명 from 유저명;
revoke select on scott. STUDENT from hr;
revoke create public synonym from hr;
```
##### [hr계정에서 scott계정의 모든 테이블 조회]
```sql
1. hr계정에서 scott계정의 모든 테이블을 조회할 수 있도록 권한 부여
select 'grant select on '||TABLE_NAME||' to hr;'
from user_tables;

2. scott 계정의 모든 테이블에 public synonym 생성
create or replace public synonym emp for scott.emp;

select 'create or replace public synonym '||TABLE_NAME||' for scott.'||TABLE_NAME||';'
from user_tables;

3. hr계정에서 테이블 조회
```
#### * role 
(권한의 묶음)
- 제 계정에서 테이블 보여줄 수 있게 해주세요 담당자에게 요청하면 위에처럼 한다. 
- 이제는 b가 똑같은 요청하면 이제 그냥 1000개의 권한을 묶어 1번의 명령어로 1000개의 권한을 동시에 전달할 수 있게 함
```sql
1. role 생성 
- 일단 장바구니를 만들어놔야 함

create role r1_select;

2. role에 권한을 부여, 담는다는 의미
grant select on emp to r1_select;
- to 이후에 사람계정일 수 있지만 role일수도 있다

3. role을 통한 권한 부여 
grant r1_select to hr;
select * from SCOTT.emp; 

- with grant option 옵션
- 권한 부여 시 사용, 권한을 부여받은 자가 권한을 행사할 권리를 주는 것