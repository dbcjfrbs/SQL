```sql
-- 1) emp2 테이블에서 각 직원과 나이가 같으면서 
-- 취미가 같은 직원의 수를 직원의 이름, 부서이름, 취미, 
-- pay와 함께 출력하여라.
select * from emp2;
select * from DEPT2;
alter session set nls_date_format='YYYY/MM/DD';

select e1.name, d.dname, e1.hobby, count(i.empno),e1.pay  
from emp2 e1, DEPT2 d, (select substr(birthday,1,4) 나,hobby, empno
                                       from emp2 e2) i
where substr(e1.birthday,1,4)=i.나(+) ---where절에 함수 못오기 떄문에 alias.
      and e1.HOBBY=i.hobby(+)
      and e1.DEPTNO=d.DCODE(+)
      and e1.empno!=i.empno(+)
group by e1.name, d.dname, e1.hobby, e1.pay;

--------밑의 코드가 일관성이 있어 더 가독성이 좋고 간단해 보인다.-----------

select e1.NAME, e1.hobby, e1.pay, d.DNAME,
       count(e2.EMPNO)
  from emp2 e1, EMP2 e2, DEPT2 d
 where to_char(e1.BIRTHDAY,'YYYY') = 
       to_char(e2.BIRTHDAY(+),'YYYY')
   and e1.HOBBY = e2.HOBBY(+)
   and e1.EMPNO != e2.EMPNO(+)
   and e1.DEPTNO = d.DCODE(+)
 group by e1.EMPNO, e1.NAME, e1.hobby, e1.pay, d.DNAME
 order by 1; -- select절의 첫번째 컬럼을 기준으로 정렬함을 의미
```
```sql
-- 2) STUDENT테이블에서 성별로 평균몸무게보다 
--    높은 학생의 이름, 학년, 몸무게, 평균몸무게 출력
-- 2-1) 인라인뷰
select * from STUDENT;

select name, grade, weight, avg_weight, 성별
from STUDENT s, (select substr(jumin,7,1) 성별, avg(weight) avg_weight
                 from STUDENT
                 group by substr(jumin,7,1)) i
where weight>i.avg_weight and substr(jumin,7,1)=성별;

-------------------------------------------------------------

-- 2-2) 상호연관

select s1.name, s1.grade, s1.weight, avg_weight 
from STUDENT s1 
where s1.weight>(select avg(s2.weight) avg_weight -- alias 이용불가
               from STUDENT s2
               where substr(s1.jumin,7,1)=substr(s2.jumin,7,1));

=> 출력불가, where절에 사용된 select절의 alias는 메인select절에 사용불가 
-----------------------------------밑이 풀이------------------
select s.NAME, s.GRADE, s.WEIGHT, 
       decode(substr(s.jumin,7,1),'1','남','여'),
       i.평균몸무게
  from STUDENT s, 
       (select substr(jumin,7,1) AS 성별,
               avg(weight) AS 평균몸무게
          from STUDENT
         group by substr(jumin,7,1)) i
 where s.WEIGHT > (select avg(weight) AS 평균몸무게
                     from STUDENT s2
                    where substr(s.jumin,7,1) = 
                          substr(s2.jumin,7,1))
   and substr(s.jumin,7,1) = i.성별;

-- **상호연관으로 하면 student를 3번이나 스캔하므로 인라인 뷰가 선호됨**
```
#### [참고 - 순환구조로 인한 이중 조인]
```sql
-- 4) STUDENT 테이블과 EXAM_01 테이블을 사용하여 각 
-- 학생보다 같은 학년 내에 시험성적이 높은 친구의 수를
-- 출력하되, 이름, 학년, 학과번호, 시험성적과 함께 출력.
**************************************************
각 학생과 점수 연결로 인한 조인 필요  s1-e1, s2-e2 
학년 비교로 인한 s1-s2 조인 필요 
점수비교로 인한 e1-e2 조인 필요
s1 - e1
|    |
s2 - e2  형태의 연결구조 형성
순환구조가 없는 형태로 만들 필요가 있음
(s1 - e1) - (s2 - e2) 형태로 조인 가능
**************************************************
select * from STUDENT;
select * from exam_01;

select r.name, r.grade, r.deptno1, r.total, count(r.name)
from (select name, grade, total, deptno1 
     from STUDENT s1, EXAM_01 ex1
     where s1.STUDNO=ex1.STUDNO) r,
     (select name, grade, total, deptno1
      from STUDENT s2, EXAM_01 ex2
      where s2.STUDNO=ex2.STUDNO) i                            
where r.total<i.total(+) and r.grade=i.grade(+) 
group by r.name, r.grade, r.deptno1, r.total;
```




