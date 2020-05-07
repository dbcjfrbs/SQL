
### 집합연산자
#### union/union all: 합집합
```SQL
--예제) emp 테이블에서 10번 부서원, 20번 부서원을 각각 출력, 집합연산자를 사용해서 하나의 집합
--      으로 합하여 출력
select *
from EMP
where deptno=10 
union all
select *
from emp
where deptno=20;
-------------------------------------------------------
[참고]
select name, sal -- 나열 순서대로 대응되어 합쳐지며 합칠 컬럼의 데이터타입이 서로 같아야 함 
from EMP         
where deptno=10 
union    
select sal, name -- 나열 순서대로 대응되어 합쳐지며 합칠 컬럼의 데이터타입이 서로 같아야 함
from emp
where deptno=20;

=> 실행 불가
```
- 두 개의 절 사이에 위치 
- 2 - 22:00 듣기 union 사용시 내부 정렬 수행됨 따라서 정렬할 필요가 없으며 중복될 데이터가 없는게 확실할 때 union all을 사용하는 것이 좋음
- intersect: 교집합 
- minus: 차집합, 위 집합에서 아래집합을 제거한 결과
----------------------------------------------------------------------------------
#### [참고 - 타 계정의 테이블 조회하기]
- scott계정에서 hr계정의 employees 테이블 조회가능하도록 작업하려면 system계정에서 아래 명령어 수행
```sql
create public synonym 별칭명 for hr.employees;
grant select on hr.EMPLOYEES to scott;
```
**설명**
-- scott과 hr 계정은 각각 서로의 땅들을 가지므로 서로의 것을 조회할 수 없음(권한이 분리되어 있음)
-- 하지만 다른 계정에 권한을 부여한다면 조회 가능
-- 원래 출력은 scott.emp 였지만 본인 계정 내에서 출력을 해왔으므로 scott. 은 생략했었음 
-- 다른 계정의 테이블을 조회할 때 계정명.을 꼭 입력해야 하나 synonym으로 다시 생략 가능

-------------------------------------------------------------------------------------
```sql
--연습문제) emp 테이블과 employees 테이블을 합쳐 사원번호, 이름, job, sal 정보를 출력
--          employees 에서 사원번호는 employee_id, 이름은 first_name, job은 job_id, sal은
--          salary 사용, 단 사원번호는 모두 4자리로 통일하도록 한다
-- 이때, emp와 employees는 다른 계정에 속한 테이블로서 위의 참고 글을 통해 코딩 가능

select to_char(empno), ename,job, SAL
from emp
union all
select lpad(employee_id,4,0), first_name, job_id, salary
from employees; -- to_number(lpad(employee_id,4,0))사용시 모든 언어에서 숫자는 앞의 0을 생략해 표시
                -- 컬럼명은 위 집합의 컬럼명으로 써진다 
```
--------------------------------------------------------------------------------------
### join: 서로 다른 테이블을 하나로 합침
```sql
select *
from emp, dept; -- 발생가능한 모든 교차 정보를 출력하게 됨 
                -- curtician 곱 발생
=> 56 개의 행이 선택(emp: 14개 행, dept: 4개의 행)

-------------------하나씩 대응시켜 join하려면 아래처럼 코딩----------------------

select empno, ename, dname, e.deptno
from emp e, dept d --from절에서도 alias사용 가능하며 사용시 모든에서 구문 alias로만 입력해야함
where e.deptno=d.deptno; -- 두 테이블을 연결하는 조건 입력, emp.depno 값을 dept에 던져 연결
```
- 집합연산자는 행의 추가, 조인은 컬럼의 추가로 이해!
------------------------------------------------------------------------------------
#### [참고 - select 절에 표현하기 위한 group by 절 조건]
```sql
--연습문제) emp 테이블에서 부서별 평균연봉을 구하고 부서명, 평균연봉 형태로 출력

select e.deptno, dname 부서명, avg(sal), loc 연봉형태 
from emp e, dept d
where e.deptno=d.DEPTNO
group by e.deptno,dname, loc;
```
- select 절에 출력할 컬럼은 모두 group by절에 넣아야 출력 가능
   이 때, group by에 넣는 컬럼들로 인해 원치 않는 세부적인 그룹핑이 되는지 주의 필요!! 
- 다양한 칼럼을 얻기 위해서 가능하면 될 수 있는 컬럼을 모두 group by절에 넣는다고 생각!
------------------------------------------------------------------------------------
#### non equi join: 등식이 아닌 조건 명시 
```sql
--연습문제) 위 테이블을 사용해 각 고객이 가져갈 수 있는 모든 샘플을 출력
select g1.GNAME 고객명, g1.POINT 포인트, g2.GNAME 상품명
from GOGAK g1, GIFT g2
where g1.POINT>=g2.G_END
order by 고객명;

=>
고객명                        포인트 상품명
------------------------- ---------- -------------------------
김설희                        542000 산악용자전거
김설희                        542000 주방용품세트
김설희                        542000 세차용품세트
김설희                        542000 샴푸세트
김설희                        542000 참치세트
김신영                        153000 참치세트
김지영                        770000 산악용자전거
김지영                        770000 참치세트
김지영                        770000 LCD모니터
김지영                        770000 노트북
김지영                        770000 세차용품세트
...                          ...     ...
68 개의 행이 선택되었습니다.
```
- `where g1.POINT>=g2.G_END` 조건을 만족하는 경우들을 모두 출력함
---------------------------------------------------------------------------------
#### [참고 - 조인 시 컬럼이름이 유일하다면 테이블 명시할 필요 없음]
```sql
--연습문제) STUDENT, EXAM_01, HAKJUM 테이블을 사용해 각 학생의 이름, 학년, 시험점수, 학점 출력

select s.name, s.grade, ex.total, h.grade
from STUDENT s, EXAM_01 ex, HAKJUM h
where s.studno=ex.studno and ex.total between MIN_POINT and MAX_POINT;
```
----------------------------------------------------------------------------------
#### [참고 - group by 절에 표현식(컬럼이 아닌 것) 사용 가능]
```sql
--예제) STUDENT, EXAM_01, HAKJUM 테이블을 사용해 각 학점별(A,B,C,D) 학생수, 평균점수, 
--      최대점수, 최소점수 출력

select substr(h.grade,1,1) 학점, count(s.studno) 학생수, avg(ex.total) 평균점수,
       min(ex.total) 최소점수, max(ex.total) 최대점수
from STUDENT s, EXAM_01 ex, HAKJUM h
where s.studno=ex.studno and
      ex.total between h.MIN_POINT and h.MAX_POINT
group by substr(h.grade,1,1)
order by 학점; 

=>
학점     학생수   평균점수   최소점수   최대점수
--  ---------- ---------- ---------- ----------
A           4      93.75         91         97
B          12         85         81         89
C           3         78         77         79
D           1         62         62         62
```