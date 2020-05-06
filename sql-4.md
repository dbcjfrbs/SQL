```sql
# deptno가 10인 직원중 job이 clerk인 직원은 A부서 
  나머지는 B부서 20인 직원은 C부서, 30인 직원은 D부서로 이동  

select ename, job, deptno, 
       decode(deptno,10,decode(job,'CLERK','A','B'),
              20,'C',
              30,'D')
  FROM emp; 
```
#### decode 심화
- 내부 decode 가능
- 조건은 등호가 성립하는 것만 가능하며 콤마(,)로 구분해 명시
-------------------------------------------------
```sql
1)
select deptno, case when deptno=10 then 'A' 
                    when deptno=20 then 'B' 
                    when deptno=30 then'C' end 새부서명1
from emp;
---------------------------------------
2)
select deptno, case deptno when 10 then 'A' --deptno와 10은 동일 타입 
                           when 20 then 'B'             
                           when 30 then 'C' end 새부서명1
from emp;

=> 같은 결과
```
#### case when 조건1 then 치환1 when 조건2 then 치환2... else 'else치환값' end
- 2에서 case와 when 바로 뒤의 데이터 타입이 서로 같아야 함  
- 성능은 case문이 decode보다 좋음  =>why? 찾아볼께요ㅠㅠ
- 조건에 등호가 아닌 대소비교도 가능 
------------------------------------------------
```sql
select count(*),count(empno),count(comm) --comm에 속한 null은 세지않음
from emp;

=>    14 | 14 | 4
```
#### count(그룹함수): 갯수
- emp 테이블이 가지는 행의 갯수가 14라는 단일 값 한 건으로 출력됨 
- *는 정확하지만 속도가 느려 한 컬럼에만 적용하는게 성능이 좋음
------------------------------------------------
```sql
select sum(sal),sum(comm) -- null값 제외하고 계산 
from emp;
```
#### sum: 합
-----------------------------------------------
```sql
select avg(comm),sum(comm)/count(*),avg(nvl(comm,0))
from emp;

=>  550 | 157.142857 | 157.142857
```
#### avg: 평균
- count, sum과 마찬가지로 null값 제외하고 계산함
------------------------------------------------
```sql
select min(sal), max(sal)  
from emp;
```
#### max/min: 최대/최소
------------------------------------------------
```sql
1)
select deptno, max(sal) 
from EMP 
group by deptno; 
------------------------------------
2)
select grade,
       case substr(jumin,7,1) when '1' then '남' else '여' end,
       avg(height)
  from student
group by grade, 
         case substr(jumin,7,1) when '1' then '남' else '여' end; 
```
#### group by 절: 그룹별 그룹함수의 연산결과 출력
- group by절에 쓴 컬럼은 그룹함수를 적용한 것과 갯수가 일치해서 select절에 같이 쓸 수 있음 
즉, 컬럼을 이루는 갯수가 같지 않으면 select 절에 같이 들어갈 수 없음
- select 절 이전에 group by가 먼저 수행되는 query임
- 2코드처럼 2차적으로 grouping 가능
-----------------------------------------------
```sql
# query 수행 순서**

--select          --5 출력하고
--from            --1 테이블을 불러오고
--where           --2 조건을 걸어 거르고 
--group by        --3 그룹핑을 하고
--having          --4 그룹핑한 것에 대한 조건을 걸고
--order by        --6 출력한 것을 정렬하여 최종출력
--------------------------------------------------------------------
1)
select deptno, avg(sal)
from emp
where avg(sal)>=2500 
=> 출력불가, avg(sal)은 group by절 적용 후 생성되는 것이라서

2)
select deptno, avg(sal)
from emp
group by deptno
having avg(sal)>=2500;
=> 출력 가능
```
#### having 절: group by 연산 결과에 조건 추가 시 사용
**[참고]** 
```sql
1)
select deptno, avg(sal)
from EMP
where deptno!=10
group by deptno 
---------------
2)
select deptno, avg(sal)
from EMP
group by deptno
having deptno!=10

=> 둘 다 같은 결과 출력
```
- having 절에도 쓸 수 있지만 group by절이 먼저 수행되기 때문에 여기서 미리 제거하는게 당연히 성능 상 좋음 
