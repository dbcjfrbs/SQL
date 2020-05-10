#### outer join: 조인조건을 만족하지 않는 대상까지 추출
- 기준이 되는 테이블에 따라 left outer join, right outer join, full outer join으로 구분
```sql
--예제) student, professor 테이블을 사용해 각 
--      학생의 이름, 학년, 지도교수명 출력
--      교수가 없는 애들도 있음

select s.name, grade, p.name 
from STUDENT s, PROFESSOR p
where s.PROFNO=p.PROFNO; 

=> 조건절에서 null값은 제외되기 때문에 지도 교수 
   없는애들은 생략되어 출력

##################################################

select s.name as 학생, grade, p.name as 교수 
from STUDENT s, PROFESSOR p 
where s.PROFNO=p.PROFNO(+);    
=> 지도 교수 없는애들을 포함한 출력을 원할 때 기준이 
   student테이블로서 outer join 사용
   위 코드에서는 left outer join으로 사용되었다고 표현함 

학생                      GRADE 교수
-------------------- ---------- --------------------
서진수                        4 조인형
일지매                        2 박승곤
김신영                        3 박승곤
김진욱                        2 양선희
서재수                        4 양선희
신은경                        3 김영조
이미경                        4 나한열
임세현                        3 심슨
김재수                        4 심슨
안광훈                        2 최슬기
김문호                        2 박원범

학생                      GRADE 교수
-------------------- ---------- --------------------
오나라                        3 박원범
박동호                        4 박원범
노정호                        2 허은
구유미                        3 허은
허우                          1
김주현                        1
인영민                        1
안은수                        1
이윤나                        1
20 개의 행이 선택되었습니다.   
```
- 생략되지 않기를 원하는 테이블 기준 반대쪽에 +기호를 붙여준다고 생각한다!
- where절에서 명시하는 테이블의 순서는 상관x
- 기준 반대쪽 컬럼이 여러개 제시되어 있다면 모두 (+)붙여줌
##### [참고 - 조인 시, 일 대 다수 매칭]
```sql
--연습문제) STUDENT와 PROFESSOR테이블을 사용해 
--          교수이름, pay, 각 교수의 지도학생이름 출력
--          단, 지도학생이 없는 교수이름도 출력
select p.name, pay, s.name
from STUDENT s, PROFESSOR p       
where p.PROFNO=s.PROFNO(+) 
order by 1;

=>
-- 같은 지도교수를 가질 수 있으므로 1대 다수 매칭이 된다 

NAME                        PAY NAME
-------------------- ---------- --------------------
김도형                      530
김영조                      350 신은경
김현정                      290
나한열                      330 이미경
바비                        500
박승곤                      380 일지매
박승곤                      380 김신영
박원범                      310 김문호
박원범                      310 오나라
박원범                      310 박동호
송도권                      270

NAME                        PAY NAME
-------------------- ---------- --------------------
심슨                        570 임세현
심슨                        570 김재수
양선희                      250 김진욱
양선희                      250 서재수
전민                        220
조인형                      550 서진수
주승재                      490
차범철                      260
최슬기                      330 안광훈
허은                        290 구유미
허은                        290 노정호
```
##### [참고] full outer join
```sql
1) ansi표준
select s.name as 학생이름,
       p.name as 교수이름
  from STUDENT s full outer join PROFESSOR P
  on s.PROFNO=p.profno;

=>
학생이름             교수이름
-------------------- --------------------
김진욱               양선희
안광훈               최슬기
김문호               박원범
노정호               허은
이윤나
안은수
인영민
김주현
허우
                     김도형
                     전민

학생이름             교수이름
-------------------- --------------------
                     주승재
                     바비
                     김현정
                     송도권
                     차범철

2)오라클 표준
select s.name as 학생이름,
       p.name as 교수이름
  from STUDENT s,PROFESSOR P
 where s.PROFNO(+)=p.profno(+); 
 
=> 출력 불가
-- 성능 상 좋지 않아서 오라클은 full outer join을
-- 막아놈 union을 사용해 해결!!
```
#### [참고 - 여러 개 테이블이 조인 될 때 outer join]
```sql       
--연습문제) STUDENT,PROFESSOR, DEPARTMENT테이블을
--         사용해 각 학생의 이름, 지도교수명
--         지도교수 학과명 출력 단, 지도교수가 없는 
--         학생정보도 출력 
select * from student;
select * from PROFESSOR;
select * from DEPARTMENT;

select s.name 학생이름, p.name 지도교수, d.dname 교수학과
from STUDENT s, PROFESSOR p, DEPARTMENT d
where s.PROFNO=p.PROFNO(+) and p.DEPTNO=d.DEPTNO(+);
```
- 연결된 테이블 모두에 (+)붙여준다, s테이블은 p로 인해 d와도 연결됨 [s - p(+) - d(+)]
-------------------------------------------------
#### self join: 하나의 테이블을 여러 번 join
한번의 스캔으로 표현할 수 없는 정보를 똑같은 테이블을 재사용(scan)해야만 출력 가능한 경우 사용
```sql
--예제) emp 테이블에서 각 직원의 이름, 연봉과 해당
--      직원의 매니저의 이름 및 연봉을 출력하되
--      매니저보다 높은 연봉을 받는 직원 출력

select e1.ename, e1.sal, e2.ename 해당매니저,
       e2.sal 매니저연봉
from emp e1, emp e2 --서로다른 테이블인 것처럼 생각해서 join
where e1.mgr=e2.empno(+) and e1.sal>e2.sal; 
----------------------------------------------------
--연습문제) PROFESSOR에서 교수의 번호, 교수이름,
--         입사일, 자신보다 입사일 빠른 사람 인원수 출력

select p1.profno, p1.name, p1.hiredate, count(p1.name)
from PROFESSOR p1, PROFESSOR p2
where p1.HIREDATE>p2.HIREDATE(+)
group by p1.profno, p1.name, p1.hiredate; 
```
--------------------------------------------------

### sub query - 쿼리문 안에 쿼리문
- 서브 쿼리 종류 및 목적
1) select ... --> 스칼라 서브쿼리, 정해진 것이 아니고 select의 결과를 컬럼처럼 사용하기 위헤서
2) from... --> 인라인 뷰, 정해진 것이 아니고 select의 결과를 테이블화 시키기 위헤서
3) where... --> 일반 서브쿼리, 정해진 것이 아니고 select의 결과를 상수로 대체시키기 위헤서
```sql
3) where 절에 적용
--예제) emp 테이블에서 전체 직원의 평균연봉보다 낮은
--     연봉을 받는 직원의 이름, 연봉, 입사일 
select ename, sal, hiredate 
from EMP
where avg(sal)<sal; 

=> 출력불가
-- avg, max, count 같은 group함수는 where절 사용 불가 
-- why?? 찾아보고 수정하겠음다ㅠ 
-- select절에만 이용가능
-----------------------------------------------
select ename, sal, hiredate
from EMP
where (select avg(sal) from emp)>sal;
```
```sql
--예제) scott과 같은 부서에 있는 직원정보 출력
select deptno
from emp
where ename='SCOTT'; -- 이것이 서브쿼리로 들어감
--                      단일 행을 출력해야만 함

select *
from emp
where deptno=(select deptno
from emp
where ename='SCOTT');
```
------------------------------------------------
##### 2) 다중행 서브쿼리: 결과가 여러 행으로 in,   (max,min)을 출력하는 any, all연산자랑 어울림
```sql
--예제) 이름이 'A'로 시작하는 직원과 같은 부서에 
--      있는 직원정보 출력
select *
from emp
where deptno in (select deptno
              from emp
              where ename LIKE 'A%');
```
- in 연산자는 뒤에 오는 컬럼에 속하기만 하면 조건 만족
or랑 비슷한 연산자
```sql              
--예제) 이름이 'A'로 시작하는 직원들의 연봉보다 
--      높은 연봉을 갖는 직원 정보 출력
select sal  
from emp    
where ename LIKE 'A%'
=> 
       SAL
----------
      1600
      1100
----------------------------where절에 적용
select *
from emp
where sal> all(select sal  
               from emp    
               where ename LIKE 'A%'); 
=>
ENAME             SAL
---------- ----------
CLARK            2450
BLAKE            2850
JONES            2975
SCOTT            3000
FORD             3000
KING             5000
```
- all연산자는 대소 비교시 가장 적은 범위를 나타내도록 안의 값 반환
- any 연산자는 정확히 반대 개념