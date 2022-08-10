### 18-2 오류가 발생해도 프로그램이 비정상 종료되지 않도록 하는 예외처리
#### 오류란?
오라클에서 SQL 또는 PL/SQL이 정상 수행되지 못하는 상황       
- 컴파일 오류(문법오류) : 문법이 잘못되었거나 오타로 인한 오류     
- 런타임 오류(실행오류) : 명령문의 실행 중 발생한 오류     
> 이 중 '런타임 오류'를 **예외** 라고 한다.      

<br>

실행 중 예외가 발생했을 때 프로그램이 비정상 종료되는 것을 막기 위해 특정 명령어를 PL/SQL 안에 작성한다.     
이를 '예외처리'라고 한다.    

```sql
DECLARE
  v_wrong NUMBER;
BEGIN
  SELECT DNAME INTO v_wrong
    FROM DEPT
   WHERE DEPTNO = 10; -- 문자열 데이터를 숫자 자료형 변수에 대입하려해서 예외발생
EXCEPTION
  WHEN VALUE_ERROR THEN
    DBMS_OUTPUT.PUT_LINE('예외 처리 : 수치 또는 값 오류 발생');
END;
/
```
> 예외처리부(EXCEPTION 키워드 뒤에 작성한 코드 부분)가 실행되면 예외가 발생한 코드 이후의 내용은 실행안됨     

#### 예외 종류
- 내부 예외 : 오라클에서 미리 정의한 예외    
  - 사전 정의된 예외 : 내부 예외 중 예외 번호에 해당하는 이름이 존재하는 예외(비교적 자주 발생하는 예외)      
  - 이름이 없는 예외 : 내부 예외 중 이름이 존재하지 않는 예외(사용자가 필요에 따라 이름을 지정할 수 있음)
- 사용자 정의 예외 : 사용자가 필요에 따라 직접 정의한 예외

#### 예외 처리부 작성
- EXCEPTION절에 필요한 코드를 사용하여 작성     
- WHEN으로 시작하는 절을 '예외핸들러'라고 하며, 발생한 예외 이름과 일치하는 WHEN절의 명령어를 수행한다.      
- OTHERS는 먼저 작성한 어느 예외와도 일치하는 예외가 없을 경우에 처리할 내용을 작성한다.     

```sql
EXCEPTION 
  WHEN 예외 이름1 [OR 예외 이름2 - ] THEN
    예외 처리에 사용할 명령어;
  WHEN 예외 이름3 [OR 예외 이름4 - ] THEN
    예외 처리에 사용할 명령어;
  ...
  WHEN OTHERS THEN
```

##### 사전 정의된 예외 사용
예외 핸들러에 사전 정의된 예외만을 사용할 때는 앞에서 살펴본 작성 방식대로 발생할 수 있는 예외를 명시한다.    
```sql
DECLARE
  v_wrong NUMBER;
BEGIN
  SELECT DNAME INTO v_wrong
    FROM DEPT
   WHERE DEPTNO = 10; 
   
   DBMS_OUTPUT.PUT_LINE('예외가 발생하면 다음 문장은 실행되지 않습니다');
   
EXCEPTION
  WHEN TOO_MANY_ROWS THEN
    DBMS_OUTPUT.PUT_LINE('예외 처리 : 요구보다 많은 행 추출 오류 발생');
  WHEN VALUE_ERROR THEN
    DBMS_OUTPUT.PUT_LINE('예외 처리 : 수치 또는 값 오류 발생');
  WHEN OTHERS THEN
    DBMS_OUTPUT.PUT_LINE('예외 처리 : 사전 정의 외 오류 발생');
END;
/
```

##### 이름 없는 예외 사용
- 이름 없는 내부 예외를 사용해야 한다면 이름을 직접 지정해 주어야 예외 처리부에서 사용할 수 있다.     
- 선언부에서 오라클 **예외 번호와 함께 이름을 붙인다.**    

```sql
DECLARE
  예외 이름1 EXCEPTION;
PRAGMA EXCEPTION_INIT(예외 이름1, 예외 번호);
.
.
.
EXCEPTION
  WHEN 예외 이름1 THEN
    예외 처리에 사용할 명령어;
  ...
END;  
```

##### 사용자 정의 예외 사용
- 예외 이름을 정해주고, 실행부에서 직접 정의한 오류 상황이 생겼을 때 RAISE 키워드를 사용하여 예외를 직접 만들 수 있다.     

```sql
DECLARE
  사용자 예외 이름 EXCEPTION;
  ...
BEGIN
  IF 사용자 예외를 발생시킬 조건 THEN
    RAISE 사용자 예외 이름
  ...
  END IF;
EXCEPTION
  WHEN 사용자 예외 이름 THEN
    예외 처리에 사용할 명령어;
  ...
END;  
```

##### 오류 코드와 오류 메시지 사용
- PL/SQL문의 정상 종료 여부와 상관없이 발생한 오류 내역을 알고 싶을 때 SQLCODE, SQLERRM 함수를 사용한다.     
  - SQLCODE : 오류 번호를 반환하는 함수    
  - SQLERRM : 오류 메시지를 반환하는 함수     
> 두 함수는 PL/SQL에서만 사용 가능한 함수이다 (SQL문에서는 사용할 수 없다)     

```sql
DECLARE
  v_wrong NUMBER;
BEGIN
  SELECT DNAME INTO v_wrong
    FROM DEPT
   WHERE DEPTNO = 10; 
   
   DBMS_OUTPUT.PUT_LINE('예외가 발생하면 다음 문장은 실행되지 않습니다');
   
EXCEPTION
  WHEN OTHERS THEN
    DBMS_OUTPUT.PUT_LINE('예외 처리 : 사전 정의 외 오류 발생');
    DBMS_OUTPUT.PUT_LINE('SQLCODE : ' || TO_CHAR(SQLCODE));
    DBMS_OUTPUT.PUT_LINE('SQLERRM : ' || SQLERRM);
END;
/
```

#### 잊기 전에 꼭
```sql
-- Q1
-- 명시적 커서를 사용하여 EMP 테이블의 전체 데이터를 조회한 후 커서 안의 데이터가 다음과 같이 출력되도록 PL/SQL문을 작성해 보세요.     
-- 1) LOOP를 사용한 방식
-- 2) FOR LOOP를 사용한 방식

-- Q2
-- 다음 PL/SQL문의 실행 중 발생하는 예외를 다음 결과와 같이 처리하는 예외 처리부를 완성하세요.

```











