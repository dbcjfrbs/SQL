```
select sysdate,last_day(sysdate)
from dual;

=>  SYSDATE | LAST_DAY 
   20/05/05 | 20/05/31
```
#### last_day: 해당 날짜가 속해있는 월의 마지막 날짜
**[참고]**
```
select sysdate,
       round(sysdate),
       round(sysdate, 'month'),
       trunc(sysdate, 'month') 
from dual;

=> 20/05/05 | 20/05/06 | 20/05/01 | 20/05/01
```
- 일단위 반올림,월단위 반올림(15일 기준), 월단위 버림 가능
-------------------------------------------------
```
 select sysdate, --2020/05/05
        to_char(sysdate, 'YYYY'), -- 2020
        to_char(sysdate, 'month'), -- 5월
        to_char(sysdate, 'day'), -- 화요일
        to_char(sysdate, 'Year'), -- Twenty Twenty
        to_char(sysdate, 'ddth') -- 05th
from dual;

=> 2020/05/05 | 2020 | 5월 | 화요일 | Twenty Twenty | 05th  
   (앞으로 가독성을 위해 위처럼 코드 옆에 주석으로도 달겠음)
```
#### to_char(형 변환 함수): 숫자,날짜 >> 문자
- to_char에 의한 출력은 숫자로 보일지라도 문자임
```
select to_char(1234, '$9,999.99'), -- $1,234.00
       to_char(1234, '99999'), -- ' 1234'
       to_char(1234, '09999') -- 01234
from dual;
```
--------------------------------------------------
```
select '1'+1 
from dual;

=> 2
```
#### to_number: 문자 >> 숫자
- 묵시적 형변환 발생 <>=> to_number('1')) 
--------------------------------------------------
```
select to_date('89/12/05'), -- 0089/12/05
       to_date('89/12/05', 'YY/MM/DD'), -- 2089/12/05
       to_date('89/12/05', 'RR/MM/DD') -- 1989/12/05
  from dual;
  ------------------------------
select to_char(to_date('1980/12/17'), 'YYYY"년" MM"월" DD"일"')  
from dual; 

=> 1980년 12월 17일
```
#### to_date: 숫자,문자 >> 날짜
- 두번째 인자에 첫번째 인자의 포맷을 알려주는 정보 표시, 
변경이 아닌 해석의 의미(파싱이라고 함)
- YYYY년 MM월 DD일은 형식이 없어서 출력 안됨, " "로 넣어주면 방해받지 않고 삽입가능  

**[정리]**
- YYYY/MM/DD 나 YYYY-MM-DD 는 포맷으로 인정, "  "은 alias와 날짜 포맷에 삽입할 때 사용 
--------------------------------------------------
#### [참고]
```
select *
  from EMP
 where sal*1.1 >=3000;
-----------------밑에가 더 성능이 좋다------------------ 
select *
  from EMP
 where sal >=3000/1.1;
```
- 내부적으로 index를 사용해 쿼리 연산을 수행함
  index가 사용될 조건 중 하나, 컬럼의 변형이 일어나면 안됨
------------------------------------------------
```
alter session set nls_date_language='american';

where to_char(hiredate, 'Month DD, YYYY')='Fabruary 22, 1981';
--------
where hiredate=to_date(Fabruary 22 1981','month DD,'YYYY')
```
#### 영문 설정으로 변경 후 쿼리 작성
------------------------------------------------
```
select ename,
       sal,
       comm,
       sal+comm,  -- null을 포함한 모든 계산값은 null로 출력
       sal+nvl(comm, 0)  
  from emp;

** 현재 cmd 창으로 쿼리를 수행하고 있어 output을
   올리기가 번거로워 나중에 orange 툴을 이용이 
   가능하게 되면 올리도록 하겠습니다. 죄송해용ㅠㅠ
   그래도 sql을 공부해보신 적이 있다면 대충 이해가 되실겁니다!
   툴이 있으시다면 복붙해서 수행해주세요.. 
```
#### nvl(데이터, 데이터가 null일 경우 치환값)
```
select comm,
       nvl(comm, 'comm이 없다') --comm은 number타입임
  from emp;

=> 수치가 부적합 합니다
```
- 치환대상끼리 데이터타입이 일치해야 함 
하지만 문자컬럼에는 숫자삽입이 가능하다!!

--------------------------------------------
```
select nvl2(comm, comm*1.1, 300),
       nvl2(comm, 'comm이 있음', 'comm이 없음'), -- 가능
       nvl2(comm,0,'a') -- 불가, 두번째 데이터타입이 기준이기 때문임
  from emp;
```
#### nvl2(데이터,'널이아닐때치환','널일때치환')   
- 두번째 세번째 데이터타입만 같으면 출력 가능
----------------------------------------------
### 조건문
- sql에서는 if문이 없음 
변수를 생성, 삽입해 프로그래밍 하는 언어를 절차적 언어라고 함 
sql은 절차적 언어가 아니며 select 구문 하나로 처리함
반면, plsql라고 해서 절차적 언어로서 확장형 sql로 사용 가능
```
# clerk이면 A, salesman이면 B, manager이면 C, 그 외 D 출력

select job,
       decode(job, 'CLERK', 'A', 'SALESMAN', 'B', 
              'MANAGER', 'C', 'D')
  from emp;
```
#### decode(검사대상, 조건1,치환1[,치환1'],치환2,  치환2'...)

- 2개씩 쌍으로 생각하고 나중에 하나 남으면 모두 아닌 값을 출력