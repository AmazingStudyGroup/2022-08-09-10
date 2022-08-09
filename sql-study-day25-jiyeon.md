## 18 커서와 예외 처리
### 18-1 특정 열을 선택하여 처리하는 커서
#### 커서란? 
SELECT문 또는 데이터 조작어 같은 SQL문을 실행했을 때, **해당 SQL문을 처리하는 정보를 저장하는 메모리 공간**을 뜻한다.     
- 커서를 사용하면 실행된 SQL문의 결과값을 사용할 수 있다.   
- 커서는 사용방법에 따라 명시적 커서와 묵시적 커서로 나뉜다.    

#### SELECT INTO 방식
SELECT INTO문은 조회되는 데이터가 단 하나의 행일 때 사용 가능한 방식( **커서는 결과행이 하나이든 여러 개이든 상관 없음** )   
```sql
SELECT 열1, 열2, ..., 열n INTO 변수1, 변수2, ..., 변수n
FROM ...
```

```SQL
-- SELECT INTO문을 사용하여 DEPT테이블의 40번 부서 데이터를 조회하고 각 열의 값을 변수에 저장한 후 출력

DECLARE
  V_DEPT_ROW DEPT%ROWTYPE;
BEGIN
  SELECT DEPTNO, DNAME, LOC INTO V_DEPT_ROW
    FROM DEPT
   WHERE DEPTNO = 40;
  DBMS_OUTPUT.PUT_LINE('DEPTNO : ' || V_DEPT_ROW.DEPTNO);
  DBMS_OUTPUT.PUT_LINE('DNAME : ' || V_DEPT_ROW.DNAME);
  DBMS_OUTPUT.PUT_LINE('LOC : ' || V_DEPT_ROW.LOC);
END;
/
```

#### 명시적 커서
사용자가 직접 커서를 선언하고 사용하는 커서     
1) 커서 선언     
사용자가 직접 이름을 지정하여 사용할 커서를 SQL문과 함께 선언한다.     
2) 커서 열기    
커서를 선언할 때 작성한 SQL문을 실행한다. 이 때, 실행한 SQL문에 영향을 받는 행을 active set이라고 한다.    
3) 커서에서 읽어온 데이터 사용     
실행된 SQL문의 결과행 정보를 하나씩 읽어와서 변수에 저장한 후, 필요한 작업 수행. 각 행별로 공통 작업을 반복해서 실행하기 위해 여러 종류의 LOOP문을 함께 사용할 수 있다.    
4) 커서 닫기     
모든 행의 사용이 끝나고 커서를 종료한다.     

```sql
DECLARE 
    CURSOR 커서 이름 IS SQL문; -- 커서 선언
BEGIN
    OPEN 커서 이름;            -- 커서 열기 
    FETCH 커서 이름 INTO 변수   -- 커서로부터 읽어온 데이터 사용
    CLOSE 커서 이름;           -- 커서 닫기
END;
```

##### 여러 행의 데이터를 커서에 저장하여 사용하기(LOOP문 사용)

```sql
DECLARE
-- 커서 데이터를 입력할 변수 선언
V_DEPT_ROW DEPT%ROWTYPE;

-- 명시적 커서 선언
CURSOR c1 IS
  SELECT DEPTNO, DNAME, LOC
    FROM DEPT;
    
BEGIN
-- 커서 열기
OPEN c1;

LOOP
  -- 커서로부터 읽어온 데이터 사용
  FETCH c1 INTO V_DEPT_ROW;
  
  -- 커서의 모든 행을 읽어오기 위해 %NOTFOUND 속성 지정
  EXIT WHEN c1%NOTFOUND;
  
DBMS_OUTPUT.PUT_LINE('DEPTNO : ' || V_DEPT_ROW.DEPTNO
                 || ', DNAME : ' || V_DEPT_ROW.DNAME
                 || ', LOC : || V_DEPT_ROW.LOC);
                 
END LOOP;            

-- 커서 닫기
CLOSE c1;

END;
/
```

|속성|설명|
|-----|----|
|커서 이름%NOTFOUND|수행된 FETCH문을 통해 추출된 행이 있으면 false, 없으면 true를 반환|
|커서 이름%FOUND|수행된 FETCH문을 통해 추출된 행이 있으면 true, 없으면 false를 반환|
|커서 이름%ROWCOUNT|현재까지 추출된 행 수를 반환|
|커서 이름%ISOPEN|커서가 열려(open) 있으면 true, 닫혀(close)있으면 false를 반환|


##### 여러 개의 행이 조회되는 경우(FOR LOOP문)
```SQL
FOR 루프 인덱스 이름 IN 커서 이름 LOOP
  결과 행별로 반복 수행할 작업;
END LOOP;
```

- 루프 인덱스 : 커서에 저장된 각 행이 저장되는 변수를 뜻하며 '.'을 통해 행의 각 필드에 접근할 수 있다.      
ex) 커서에 저장할 SELECT문에 DEPTNO열이 존재하고, 이 커서를 사용하는 루프 인덱스 이름이 c1_rec일 경우,       
c1_rec.DEPTNO는 SELECT문을 통해 조회된 데이터의 각 행에 해당하는 DEPTNO열의 데이터를 가리키게 된다.     
- 커서에서 FOR LOOP문을 사용하면 OPEN, FETCH, CLOSE문을 작성하지 않는다.     

```sql

-- FOR LOOP문을 활용하여 커서 사용하기
DECLARE
  -- 명시적 커서 선언
  CURSOR c1 IS
  SELECT DEPTNO, DNAME, LOC
    FROM DEPT;
    
    
BEGIN      
  -- 커서 FOR LOOP 시작 (자동 Open, Fetch, Close)
  FOR c1_rec IN c1 LOOP
    DBMS_OUTPUT.PUT_LINE('DEPTNO : ' || c1_rec.DEPTNO
                       ||', DNAME : '|| c1_rec.DNAME
                       ||', LOC : ' || c1_rec.LOC);
  END LOOP;

END;
/ 
```

`결과 화면`

DEPTNO : 10, DNAME : ACCOUTNING, LOC: NEW YORK     
DEPTNO : 20, DNAME : RESEARCH, LOC: DALLAS    
DEPTNO : 30, DNAME : SALES, LOC: CHICAGO    
DEPTNO : 40, DNAME : OPERATIONS, LOC: BOSTON        
     

##### 커서에 파라미터 사용하기
고정 값이 아닌 직접 입력한 값 또는 상황에 따라 여러 값을 번갈아 사용하려면 다음과 같이 커서에 파라미터를 지정할 수 있다.    
```sql
CURSOR 커서 이름(파라미터 이름 자료형, ...) IS 
SELECT ...
```

```sql
-- 만약 DEPT 테이블의 부서 번호가 10번 또는 20번일 때 다른 수행을 하고 싶다면, 다음과 같이 커서의 OPEN을 각각 명시하여 실행한다.   
DECLARE
  -- 커서 데이터를 입력할 변수 선언
  V_DEPT_ROW DEPT%ROWTYPE;
  -- 명시적 커서 선언
      CURSOR c1 (p_deptno DEPT.DEPTNO%TYPE) IS
        SELECT DEPTNO, DNAME, LOC
          FROM DPET
        WHERE DEPTNO = p_deptno;
BEGIN
  -- 10번 부서 처리를 위해 커서 사용
  OPEN c1 (10);
    LOOP
      FETCH c1 INTO V_DEPT_ROW;
      EXIT WHEN c1%NOTFOUND;
      DBMS_OUTPUT.PUT_LINE('10번 째 부서 - DEPTNO : ' || V_DEPT_ROW.DEPTNO
                                    || ', DNAME : ' || V_DEPT_ROW.DNAME
                                    || ', LOC : ' || V_DEPTNO.LOC);
    END LOOP;
  CLOSE c1;
  -- 20번 부서 처리를 위해 커서 사용
  OPEN c1 (20);
    LOOP
      FETCH c1 INTO V_DEPT_ROW;
      EXIT WHEN c1%NOTFOUND;
      DBMS_OUTPUT.PUT_LINE('20번 째 부서 - DEPTNO : ' || V_DEPT_ROW.DEPTNO
                                    || ', DNAME : ' || V_DEPT_ROW.DNAME
                                    || ', LOC : ' || V_DEPTNO.LOC);


    END LOOP;            
  CLOSE c1;
END;
/
```

##### 커서에 사용할 파라미터 입력받기
```sql
DECLARE
  -- 사용자가 입력한 부서 번호를 저장하는 변수선언
  v_deptno DEPT.DEPTNO%TYPE;
  -- 명시적 커서 선언(Declaration)
  CURSOR c1 (p_deptno DEPT.DPETNO%TYPE) IS
    SELECT DEPTNO, DNAME, LOC
      FROM DEPT
     WHERE DEPTNO = p_deptno;
BEGIN
  -- INPUT_DEPTNO에 부서 번호를 입력받고 v_deptno에 대입
  v_deptno := &INPUT_DEPTNO;
  -- 커서 FOR LOOP 시작. c1 커서에 v_deptno를 대입
  FOR c1_rec IN c1(v_deptno) LOOP
    DBMS_OUTPUT.PUT_LINE('DEPTNO : ' || c1_rec.DEPTNO
                     || ', DNAME : ' || c1_rec.DNAME
                     || ', LOC : ' || c1_rec.LOC);
  END LOOP;
END;
/
```

#### 묵시적 커서
- 별다른 선언 없이 SQL문을 사용했을 때 오라클에서 자동으로 선언되는 커서를 뜻한다.    
- 사용자가 OPEN, FETCH, CLOSE를 지정하지 않는다.    
- PL/SQL문 내부에서 DML명령어나 SELECT INTO문 등이 실행될 때 자동으로 생성 및 처리된다.     
