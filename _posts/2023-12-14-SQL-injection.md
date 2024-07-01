## 개요

처음엔 SQL Injection을 이해하지 못했다. 인터넷을 돌아다니며 처음 봤던 예제가 이해하기 어려웠던 것 같다. 그래서 이번 기회에 정리해보려한다.

## 어떻게 가능한 걸까?

![image](https://github.com/jh10253267/TIL/assets/108499717/192cd8b8-c3de-4f1a-8275-d9709afd3235)
재밌는 만화

우리가 입력한 값은 db에 칼럼값으로 들어가게된다.

```sql
INSERT INTO STUDENTS(name) VALUES('학생 이름');
```
만약 여기서 공격자가 서버에서 실행될 쿼리문을 유추해서 악의적인 목적으로 학생의 이름을 `Robert'); DROP TABLE STUDENTS;--')`로 입력하면 이 값이 value 괄호안에 들어가서 쿼리문이 실행된다. (뒤에 -- 기호는 주석을 나타낸다.)

```sql
INSERT INTO STUDENTS(name) VALUES('Robert'); DROP TABLE STUDENTS;--')
```
SQL Injection은 이런식으로 일어난다.

또 다른 예

```sql
SELECT user FROM user_table WHERE id='입력한 아이디' AND password='입력한 비밀번호';
```

아이디와 비밀번호로 로그인하는 쿼리문이다.  
일반적인 사용자라면 문제가 없이 작동할 것이다.

그러나 악의적인 목적을 가진 유저라면
id 값으로 admin, password값으로 ' OR '1' = '1과 같은 비정상적인 값을 입력할 것이다.

```sql
SELECT user FROM user_table WHERE id='user' AND password=' ' OR '1' = '1';
```
1. id가 user이고 password가 빈 문자열인 데이터를 찾는다.
2. '1' = '1' 의 조건을 or로 추가하여 언제나 true를 리턴한다.
3. 앞의 아이디와 패스워드가 일치하지 않더라도 SQL Injection 공격의 영향으로 결국 권한이 없더라도 모든 데이터를 가져오게 된다.

## Statement와 PreparedStatement 

전에 살펴봤던 JDBC에서 SQL Injection을 막기위해 Statement 대신 PreparedStatement를 사용한다고 한적이 있다.

ps는 물음표로 바인딩된 값을 스트링으로 처리한다. 즉 좀전에 살펴본 예에서의 악의적인 구문이 sql문으로써 작용하지 않고 단순한 문자열로 작용하기 때문에 SQL Injection에 안전한 것이다.

반면 Statement의 경우

```sql
String query = "SELECT * FROM STUDENTS WHERE name = '" + name + "';
```

위와 같이 사용된다. 이렇게 되면 악의적인 구문이 그대로 쿼리문이되고 실제로 db에 영향을 미치게된다.

위에서 본 만화와 같은 상황에 실제로 일어나게 되는 것이다.

Statement사용을 지양하고 PreparedStatement를 사용해야하는 이유를 좀 더 자세히 알아보자면 이렇다.
![image](https://github.com/jh10253267/TIL/assets/108499717/0c04148b-8548-477c-9c7f-fc5c71d88357)
우리가 작성한 쿼리는 다음의 과정을 거쳐 수행된다.
1. Parsing - 쿼리문이 개별 단어로 변환되는 과정으로 구문 오류 및 철자 오류를 체크한다.
2. Semantics Check - 쿼리의 유효성을 체크한다. 쿼리문에 명시된 테이블이 정말 존재하는지 컬럼은 존재하는지 쿼리를 실행한 유저가 해당 작업을 수행할 자격이 있는지.
3. Binding - 쿼리가 기계친화적 언어로 바뀌는 단계
4. Query Optimization - 쿼리문을 실행하는데 가장 최적의 알고리즘을 찾는 단계. 
5. Cache - 가장 최적의 알고리즘이 캐시에 저장된다. 같은 쿼리가 다시 들어올 경우 앞의 단계들을 건너뛰고 바로 실행된다.
6. Execution - 쿼리가 실행되고 결과가 반환된다.

그리고 PreparedStatement는 다음과 같은 과정을 거친다.
![image](https://github.com/jh10253267/TIL/assets/108499717/c093d945-4930-4b29-93ae-7d18b469c294)
1. Parsing and Semantics Check는 동일하다.
2. Binding - 우리가 물음표로 설정한 부분을 인식하고 쿼리가 placeholders와 같이 컴파일된다. 
3. Cache - 앞에서 본 캐시와 같다.
4. Placeholder Replacement - Placeholder는 사용자가 입력한 데이터로 대치된다. 그리고 이 단계에서 쿼리는 이미 binding이 되어 컴파일된 상태이다. 그래서 유저가 입력한 정보는 하나의 단순한 문자열로 해석되며 바인딩된 쿼리의 의미를 변화시킬 수 없다. (이미 쿼리문은 정해졌다.)

이런 이유로 PreparedStatement를 사용하면 SQL Injection으로부터 안전한 것이다.