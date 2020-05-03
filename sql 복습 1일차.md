```SQL
select * from emp;
```
#### 데이터 조회 가능
- sql작업이며 누가 짜는냐에 따라 성능이 달라짐
다른 언어는 성능이 크게 좌우되지 않지만 sql은 데이터 처리 시간이 달라짐 

---

```SQL
desc emp;
```
#### 데이터 타입 확인 가능

---
```SQL
select empno, ename, SAL, sal*1.1, 1111, 'abcd'
from emp;
```
#### 표현식: column명을 제외한 나머지 

- 1111,'abcd'는 표현식으로써 규격에 맞게 모두 출력됨

- 실존데이터는 대명사로 취급해 싱글따옴표 필요 없음, 사용자 입력값(문자?)만 싱글따옴표 취함

----
```
## hr 계정에서만 다음 쿼리 가능
select *  from emp;
```
- hr에 속한 employees 테이블은 scott 계정에서 조회가 안됨

- 계정이 소유하고 있는 모든 대상을 **schema**라고 함

---

```SQL
select deptno (as)부서, dname (as)부서명, loc (as)위치 
from dept;
---------------
select empno (as)"사원 번호", ename (as)"사원이름!", SAL (as)"salary" 
from emp;
```
#### alias : 컬럼 이름 출력 시 변경하여 사용
  - 출력 시에만 바뀌고 원본은 바뀌지 않음 뛰어쓰기 허락 안함
  - 더블 따옴표로 띄어쓰기, 특수기호 소문자 표현 가능
------

```SQL
select empno||ename||sal
from emp;
```
#### 연결 연산자(||): 컬럼과 컬럼, 컬럼과 표현식을 연결

---
**[참고]** 아예 변경을 SQL에서 하고 추가적으로 R, 파이썬에서 정제하는 것이 성능 상 좋다.

---

```SQL
select distinct deptno, ename 
from emp;
```
#### distinct : 중복행 제거
- select 뒤에 바로 와야함
- deptno와 ename의 조합을 대상으로 했을 때 유일한 값

---

```SQL
select * 
from student
where grade=4 and studno=9411; 
```
#### where절 구문 : 행을 선택하기 위한 조건 나열

- where name='서재수' 
  - 서재수가 싱글따옴표가 없다면 컬럼명으로 해석됨, 문자나 날짜 상수를 표현할 떄 '  ' 사용

- where ename='SMITH' 
  - 문자상수는 대소문자를 구별함, 문법은 대소문자를 구분하지 않음

---
```SQL
select * 
from EMP
where sal between 2000 and 3000;
```
#### between A and B: A 이상 B 이하 범위 선택
- between은 제한조건을 포함!

---
```SQL
select *
from student
where grade in(1,2);
```
#### in연산자: A 또는 B 또는 C ...

---
```SQL
select * 
from EMP
where ename like 'S%';
```
#### like 연산자: 패턴연산자
- where ename like '_A%'
  - 두번째 시작이 A
- where ename like '_ A _ _' 
  - 두번째 시작 A이고 총 4자리

---

#### not연산자: 부정 연산자
- *not between A and B*: A 미만 B 초과
- *not in('A','B')*: A와 B를 제외한 나머지 
- *not like 'A%'*: A로 시작하지 않는

---
```SQL
select *
from EMP
where comm is not null; -- where comm is null; 
```
- ' '는 문자임
- null은 공간을 차지하지 않는 데이터로 size가 없음, 아직 입력되지 않은 데이터

---
#### 참고
```SQL
select *
from EMP;
---
select * from EMP;
```
- DB는 문법을 해석할 때 띄어쓰기 한 sql은 다른 sql로 인지함, 문법을 해석하는 옵티마이저가 있는데 순서대로 해석하기 때문

- 네비게이션에 즐겨찾기하면 다시 찍을 필요가 없고 길을 빨리 찾을 수 있듯이 한번 수행된 sql을 저장해두기 때문에 똑같은 sql을 수행하면 빠른 처리가 가능함 
- 반면 같은 결과가 수행되는 sql을 다른 포맷으로 입력하면 메모리 용량이 쓸데 없이 차게되 성능 낮아짐 **결국 쿼리를 같은 포맷으로 정리해둘 필요가 있음**





