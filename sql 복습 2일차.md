```sql
elect empno, ename, job, sal, hiredate, deptno
from EMP
where hiredate>'1982/01/01';
```
#### 날짜 대소 판단 가능
- 툴에서는 년도가 네 자리로 표현되지만 실제 YY/mm/dd 포맷으로 저장되어 있으며 콘솔 창에서 확인 가능
---------------------------------

```sql
alter session set nls_date_format= 'YYYY/MM/DD';
```
#### 날짜 형식 포맷 변경  
---------------------------------
```sql
# emp 테이블에서 부서번호별, 연봉별 정렬(큰순서대로)
select *
from emp
order by deptno asc, sal desc; 
```
#### order by: 정렬
- 나열된대로 각 순서대로 정렬 
- select에 명시된 alias는 오로지 order by절에만 적용됨, order by만이 select절보다 나중에 수행되기 query이기 때문 
---------------------------------
```sql
select 24*45 
from dual; 
```
#### dual
- 원래 from의 목적은 데이터 가져오는 것을 의미함 
- 가져올 테이블 없이 그냥 값을 출력할 목적일 경우 사용
----------------------------------
```sql
select sysdate
from dual;
```
#### sysdate: 현재시간 출력
-----------------------------------
```sql
select sysdate, sysdate+5*30,add_months(sysdate,5)
from dual;
=>  20/05/04 | 20/10/01 | 20/10/04
```
#### add_months(날짜,n): n개월 후 리턴
-------------------------------------
```sql
select trunc(months_between(sysdate, 
             '80/12/17'))
from dual;
```
#### months_between(날짜1,날짜2): 날짜 사이 개월수
--------------------------------------
```sql
# 다음 주 일요일 리턴
select next_day(sysdate,1) 
from dual;
```
#### next_day(날짜,요일) 
- 바로 다음에 오는 특정요일 일자 리턴, 1: 일요일 2:월요일...
- '일' 바꾸기 가능, 설정에 따라 영어나 한글 둘 중에 하나 가능 **수정 필요 
----------------------------------
```sql
select round(1234.56)
from dual; 
```
#### round: 반올림
- 함수는 입력값에 어떠한 규칙을 적용해 결과값을 출력하는 과정을 말한다
- round는 반올림하는 규칙을 적용한 함수임

**[참고] 숫자함수**
- trunc(숫자[,자리수]): 버리기
- mod(숫자,숫자): 뒤에 숫자로 나눠서 나머지 출력
- ceil(숫자): 숫자보다 크면서 최소 정수
- floor(숫자): 숫자보다 작으면서 최대 정수
- abs: 절대값
- sign(숫자): 양수면1, 음수면 -1, 0이면 0 반환
-----------------------------------
```sql
select ename, initcap(ename)
from emp;
```
#### initcap: 첫글자를 대문자로 바꿈
- column에 대한 적용 가능
-------------------------------------
```sql
select 'abcd', length('abcd'), lengthb('abcd'), 
       '한글', length('한글'), lengthb('한글') 
from dual;
```
#### length / lengthb: 글자의 갯수 / byte
- 한 문자당 한글은 2byte, 영어는 1byte 
----------------------------------
```sql
select 'abcdef', substr('abcdef',1,2)
from dual; 
=> ab
----------------------------------------
select 'abcdef', substr('abcdef',-3,2)
from dual; 
=> de
```
#### substr(원본문자열, 시작위치[, 개수]) 
- 생략 시 문자열 끝까지 출력 
- 시작위치 -가능: 뒤에서부터 시작, 추출방향은 같음
--------------------------------------
```sql
select 'abcdea', instr('abcdea','a',2,1)
from dual;
=>6
```
#### instr(원본, 찾을문자열[, 시작위치][, 발견횟수]
- 찾는 문자열의 위치를 전체문자열 기준으로 출력
- 찾고자 하는 문자열이 없다면 0 반환
- instr은 시작위치 -일 때 스캔방향이 역방향으로 바뀜!
----------------------------------------
```sql
select lpad('나는바보',10,'*'), rpad('나는바보', 10,'*')
from dual;
=>  **나는바보 | 나는바보**
```
#### pad(원본문자열, 총자리수, 채울문자열)
- 자리수는 byte로 계산
-----------------------------------------
```sql
select 'abcba', ltrim('abcba','a'), rtrim('abcba',
        'a'), trim('  abcd  ') 
from dual;
=>  abcba | bcba | abcb | abcd
```
#### trim(원본문자열, 제거문자열)
- 순서대로 제거대상을 제거(공백포함)하다가 방해물 만나면 작업 중지!
- trim은 양쪽의 공백만 제거 가능
------------------------------------------
```sql
select 'abcda', replace('abcda','a','A'),
        translate('abcda','a','A'),
        replace('abcda','ab','AB'),
        translate('abcda','ab','AB')
from dual;  
=>  abcda | AbcdA | AbcdA | ABcda | ABcdA
---------------------------------------------------
select translate('055)381-2158',')-','  ')
from student; 
=> 055)381-2158 | 055 381 2158
```
#### replace(원본문자열, 찾을단어, 바꿀단어)
- 뭉텅이로 치환(단어의 치환)

#### translate(원본문자열, 찾을글자, 바꿀글자)
- 일대일 대응으로 글자 치환!!
- 공백으로의 치환도 문자처럼 가능

**[참고] 한번에 여러 글자 삭제**
```sql
select 'abcda', translate('abcda','!ac','!') 
from dual;
=> bd
--------------------------------------------------
select 'abcda', translate('abcda','ac','') 
from dual;
=> ''
```
- 아예 공백으로의 치환은 원하는 문자열 모두 삭제됨
**이유 아시는 분은 수정바람
