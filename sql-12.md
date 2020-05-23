```sql
--예제1) scott ->hr -> kic에게 STUDENT테이블 조회권한 부여
-- 1. 유저 생성
create user kic identified by oracle;

-- 2. kic유저에 접속권한 부여
grant create session to kic;

-- 3. scott계정에서 hr에 조회권한 및 권한부여
grant select on SCOTT.STUDENT to hr with grant option;

-- 4. hr계정에서 kic계정에 STUDENT테이블 권한부여(이외 테이블도 권한부여 시도)
grant select on SCOTT.STUDENT to kic;

-- 5. scott계정에서 hr에 부여된 권한 회수
revoke select on SCOTT.STUDENT from hr;

-- 6. kic유저는 STUDENT테이블을 조회할 수 있는가? 없다!!
select * from SCOTT.STUDENT;

** 중간관리자의 권한이 회수되면 그 권한에서 파생된 권한들도 회수되어진다.


--예제2)
--1) 다음을 수행하여라
create table emp_test2 as select * from emp;

--2) 입력 가능한 데이터 두 건 입력

--3) co1 컬럼 추가(varchar2(10))
alter table emp_test2 add (co1 varchar2(10)); 

--4) co1에 default값 'a' 지정
alter table emp_test2 modify (co1 varchar2(10) default 'a'); 
- 수정하면 이후에만 default적용!

select * from emp_test2;
--5) 다음을 수행
--(col1에 입력되는 값 차이 확인--sqld 기출)
insert into emp_test2(empno,ename,co1) values(1234,'hong',null);
-- => 입력한대로 들어감
insert into emp_test2(empno,ename) values(1111,'hong2');
-- => col에 default값인 'a'가 들어감

** default는 값이 들어오지 않을 때 들어가는 값이다!!


-- 예제3)
-- 1) rl_selscott 롤을 만들고 scott계정의 모든 테이블에 
--    대한 select 권한을 부여하여라
create role r1_selscott; 
--scott에 dba권한을 부여했기 때문에 scott에서 실행가능

select 'grant select on SCOTT.'||TABLE_NAME||' to r1_selscott;' 
from user_tables;

-- 2) 위에서 만든 롤을 kic 계정에 부여 
grant r1_selscott to kic; 
-- 롤을 통한 권한받은 자는 재접속이 필요하다!


--예제4) 아래와 같은 형식으로 조회되도록 쿼리작성

--테이블명     pk이름          pk컬럼     시노님수
--EMP2	    SYS_C0011047	   DEPTNO	        1
--STUDENT	SYS_C0011043	   GRADE	        1
--TEST       .                  .               0

-- use_constraint, user_cons_columns에는 제약조건 없는 테이블은 보이지 않음 

select t.table_name,
       i.constraint_name as "pk이름",
       i.column_name as "pk컬럼",
       count(s.synonym_name)
  from user_tables t, all_synonyms s,
      
      (select c1.table_name,
               c1.constraint_name, 
               c1.column_name
          from user_cons_columns c1, 
               user_constraints c2
         where c1.constraint_name = c2.constraint_name(+)
           and c2.constraint_type ='P') i
           
 where t.table_name = i.table_name(+)
   and t.table_name = s.table_name(+)
 group by t.table_name, i.constraint_name, i.column_name;
 
SELECT * FROM all_synonyms;
-- 관계 : s(+) - t - (c1 - c2)(+)
