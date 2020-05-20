```sql                
-- 2) student3 테이블을 만들고 비만여부를 나타내는 컬럼을
-- 새로 추가하고, 각 학생들의 비만정보를 update 하여라.
-- 비만여부는 체중이 표준체중보다 크면 과체중, 
-- 작으면 저체중, 같으면 표준으로 분류하여라.
-- *표준체중 = (키-100)*0.9

create table STUDENT3
as select * from STUDENT;

alter table STUDENT3 add (비만여부 varchar(6)); --varchar(6) 뒤에 not null 써서 선언 가능

update STUDENT3  
set 비만여부=case when weight>(height-100)*0.9 then '과체중' 
                  when weight=(height-100)*0.9 then '표준'   
                  when weight<(height-100)*0.9 then '저체중' end;
-- update시, 명령이 수행되는 특정행에서 수행될 수 있기 때문에 위에서 select가 구지 필요없음

alter table STUDENT3 modify(비만여부 varchar2(6)); -- modify를 통해 형변환 가능
desc STUDENT3;

-------------------------------------------------------------------------------------------------

-- 4) emp_back2 테이블을 만들고 각 직원의 연봉을 
-- 직원과 각 직원의 상위관리자의 연봉의 평균으로 수정.
-- 단, 상위관리자가 없는 경우는 본인의 연봉의 
-- 10% 상승된 값을 상위관리자 연봉으로 취급

create table emp_back2
as select * from emp;

#sol1)
update emp_back2 e1
set e1.sal=case when e1.mgr is not null then (select(e2.sal+e1.sal)/2
         from emp_back2 e2
         where e1.mgr=e2.EMPNO)
         else 1.1*e1.sal end;
#sol2)
update emp_back2 e1
   set sal = nvl((select (e1.sal + e2.sal)/2
                    from emp_back2 e2
                   where e1.mgr = e2.EMPNO),
                  (e1.sal + e1.sal*1.1)/2);
```
----------------------------------------------------------------------------------------------

### 데이터 딕셔너리
-  (데이터베이스 내부적으로 발생하는 모든 구조적변경과 시스템 성능과 관련된 자료를 수집 및 저장하는 공간)

-- base table + data dictionary view
-- (raw data) + (raw data 접근을 대신 해주는 대상)
- basetable을 여러 개 묶어 뷰(테이블 아닌데 테이블인 것처럼 만들어 사용자가 보도록) 만들어 놈, basetale은 원본으로 접근이 불가능

#### 1. base table
-- data dictionary view를 구성하는 원본 데이터 ex)sys.ts$, sys.seg\$
-- dba권한을 갖더라도 접근 불가

#### 2. data dictionary view
##### 1) static dictionary view: 구조정보와 관련
--dba_xxxx: dba권한을 가진 자 조회 가능 (dba권한 필요)
```select count(*) from dba_tables;``` --system에서 조회가능

--all_xxxx: 현재 유저에 권한이 있는 객체 조회 가능
```select * from all_tables;```

--user_xxxx: 본인 소유의 객체만 조회 가능
```select * from user_tables;```

***중요 오브젝트 관련 뷰**
```sql
select * from user_tables;    --테이블 정보
select * from user_tab_columns; -- 테이블내 컬럼 정보
select * from user_views; --뷰 정보
select * from user_tab_constraints; --제약조건 정보
select * from user_tab_indexes; --인덱스 정보
```
##### 2) dynamic performance view: 성능 정보와 관련
```select * from v$session;```
##### 뷰 만들기
```sql
create view abcd                                     
as
select * from SCOTT.emp e, SCOTT.dept d where e.DEPTNO = d.DEPTNO;
```
- 테이블을 만들 수는 없고 맨날 조회해 사용해야해서 뷰를 만듦, 뷰는 저장공간을 갖지않음 

-------------------------------------------------------------------------------------------

### 제약조건(constraint)
- (데이터가 잘못 입력, 수정, 삭제 되지 못하도록 (데이터 부결성) 각 컬럼별 걸어놓은 제약을 의미) 

#### 1. 종류
##### 1) not null: null을 허용하지 않는 제약
##### 2) unique: 중복된 값을 허용하지 않는 제약
##### 3) primary key: 각 행의 유일한 식별자, not null+unique, 하나만 생성 가능, 어떤 정보를 자주   찾기 위해 사용되는 컬럼으로 주민보다는 학번으로 설정함
##### 4) foriegn key: 다른 테이블(부모)를 참조해 부모테이블의 제약을 받도록 함
- emp(deptno) - dept(deptno), 부모 테이블로 dept로 정의, emp가 항상 dept를 참조해야 한다

##### 5) check: 사용자 지정 데이터만 들어오도록 하는 제약, grade에 문자 안되게 제약
--

#### 2. 생성방법(테이블 생성 시 기술)
```sql
--예제) jumun테이블과 cafe_prod 테이블을 만들고 jumun테이블의 product_no 컬럼이 cafe_prod테이블
--      의 상품번호(no)를 참조하도록 테이블을 설계
1) 테이블 생성
drop table cafe_prod;

create table cafe_prod(
no     number(4) primary key,
name   varchar(10) not null,
price  number not null);

create table cafe_jumun(
no     number(4) primary key,
qty    number not null,
product_no  number references cafe_prod(no));


2) 같은 이름의 no 생성 불가
insert into cafe_prod values(1, 'americano',1500);
insert into cafe_prod values(2, 'latte',2500);
insert into cafe_prod values(1, 'mocha',3500); -- => 무결성 제약 조건(SCOTT.SYS_C0011120)에 위배됩니다.
commit;


3) 자식에 부모의 정의되지 않은 데이터 삽입불가
insert into cafe_jumun values(1,5,1);
insert into cafe_jumun values(2,1,2);
insert into cafe_jumun values(3,2,3); **
commit;


4) 자식데이터가 있을 때 부모데이터 삭제 불가
delete from cafe_prod where no=2; -- 이미 주문이 들어가 있는 상태에서 지우려함 
-- 해당하는 자식데이터(jumun)가 있으므로 delete불가능, 자식데이터를 먼저 지우고 부모를 지워야함

2) 제약조건의 이름 부여 및 생성(어떤 컬럼 떄문에 에러가 발생했는지 알 수 있음)
create table cafe_prod2(
no     number(4) constraint cafe_no_pk primary key,
name   varchar(10) constraint cafe_name_nn not null,
price  number constraint cafe_price_nn not null);

insert into cafe_prod2 values(1, 'americano',1500);
insert into cafe_prod2 values(2, 'latte',2500);
insert into cafe_prod2 values(1, 'mocha',3500); 
-- => 무결성 제약 조건(SCOTT.CAFE_NO_PK)에 위배됩니다.
-- 기존 메세지 :  무결성 제약 조건(SCOTT.SYS_C0011120)에 위배됩니다. 에서 바뀜
```

#### 3. 제약조건 추가
```sql
1) 문법
alter table 테이블명
      add constraint 제약조건이름 종류(컬럼);


2) primary key 적용
alter table dept_back 
      add constraint deptbk_deptno_pk primary key(deptno); 
-- default 설정과 달리 원래데이터 검색해 무결성 만족하는지 검사함
-- primary키로 설정하기 위해 검사하는 시간동안은 lock걸려서 해당 테이블 사용못함


3) foreign key 적용 및 foreign key 생성을 위한 전제조건
-- 부모테이블의 reference key는 유일해야 한다
-- 따라서 primary 키 또는 unique제약조건이 먼저 생성되어 있어야한다
alter table dept_back
      add constraint deptbk_deptno_pk primary key(deptno);

alter table emp_backup
      add constraint empbk_deptno_pk foreign key(deptno) references dept_back(deptno);
```

#### 4. 제약조건 확인
```sql        
1) 테이블마다의 제약조건 확인 뷰
select *
from user_constraints; 


2) column 정보 확인
select * from user_cons_columns; 
```
----------------------------------------------------------------------------------------------
#### [참고 - 테이블 및 컬럼 comment 확인]
- comment : 각 테이블, 컬럼에 대한 상세정보 
```sql
1) comment 확인
select * from user_tab_comments; -- 테이블 정보
select * from dba_col_comments; -- 컬럼 정보


2) comment 적기
COMMENT ON table 테이블명 IS '설명'; 
COMMENT ON COLUMN 테이블명.컬럼명 IS '설명';
COMMENT ON COLUMN SYS.DBA_COL_COMMENTS.COMMENTS IS 'Comment on the object';