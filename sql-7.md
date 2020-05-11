##### 1-1) 다중컬럼 서브쿼리
```sql
--예제) emp에서 부서별 최대 연봉자의 ename, deptno, sal 출력

select ename, deptno, sal
  from EMP
 where (deptno,
        sal) in (select deptno, # 소괄호 내 조합이 모두 맞는 조건
               max(sal)
          from emp
         group by deptno);
=>
ENAME          DEPTNO        SAL
---------- ---------- ----------
BLAKE              30       2850
SCOTT              20       3000
KING               10       5000
FORD               20       3000
```

##### 1-2) 인라인 뷰 사용
```sql
select ename, deptno, sal 
  from EMP e,
       (select deptno,
               max(sal) as max_sal
          from emp
         group by deptno) i
 where e.DEPTNO=i.deptno
   and e.sal=max_sal;  --where절에 그룹함수 불가해서 alias로 사용해 대입
```
- 인라인 뷰는 다중컬럼이 안되는 대소비교가 되기 때문에 인라인 뷰를 선호함 

##### 1-3) 상호연관 서브쿼리**

```sql
1) 다중컬럼 서브쿼리로
--예제) PROFESSOR에서 소속학과별 최대 연봉을 받는 교수 출력
select *
  from PROFESSOR
 where (deptno,
               pay) in(select deptno,
               max(pay)
          from PROFESSOR
         group by deptno);
 
---------------------------------------------------
2) 상호연관 서브쿼리로
select *
  from PROFESSOR p1
 where p1.pay in (select max(pay)
          from PROFESSOR p2
         where p1.DEPTNO=p2.DEPTNO
         group by deptno);

**[중요] 상호연관 커리 실행 순서!!!**
NAME                        PAY     DEPTNO
-------------------- ---------- ----------
조인형                      550        101
박승곤                      380        101
송도권                      270        101
양선희                      250        102
김영조                      350        102
주승재                      490        102
김도형                      530        103
나한열                      330        103
김현정                      290        103
심슨                        570        201
최슬기                      330        201
...

1) 조인형의 pay확인(550) -- where p1.pay 

2) 서브쿼리에서 조인형의 deptno(101) -- where p1.DEPTNO=p2.DEPTNO

3) 서브쿼리에서 101번 학과의 max(pay) 구함(550) -- select max(pay)

4) 550=550 조건일치로 첫번째 행 리턴 -- where p1.pay in (select절)

5) 나머지 행에 대해 1~4과정 반복

[참고] 
- 서브쿼리 내 group by 생략가능 
이유: 서브쿼리 조건절 where p1.DEPTNO=p2.DEPTNO 가 조인형 deptno
      인 101에 대해 max(pay)을 계산하기 때문!!
- where절은 행마다 쿼리를 반복함
```
##### 1-4) 스칼라 서브쿼리
```sql
예제) emp에서 사원이름, 부서명, 출력

select ename,
       (select dname -- 결과가 무조건 하나 출력되어야 함
          from dept d
         where e.deptno=d.deptno) 
  from emp e;

-- 행 연산이 된다는 접근으로 상호연관쿼리 메커니즘터럼 해석하면 이해쉬움!!

연습문제) STUDENT, PROFESSOR에서 학생이름, 교수이름 출력
select name 학생이름,
       nvl((select name --그 자체가 컬럼이기 때문에 함수는 밖에다가 사용!!
                  from PROFESSOR p
                 where s.PROFNO=p.PROFNO), 'a') 교수이름
  from STUDENT s;
  
--조인을 사용하면 outer를 해야하는데 여기서는 생략없이 다나옴
