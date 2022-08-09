## 18 - 1 특정 열을 선택하여 처리하는 커서
<br/>

> 커서란?

- 커서(cursor)는 SELECT문 또는 데이터 조작어 같은 SQL문을 실행했을 때 해당 SQL문을 처리하는 정보를 저장한 메모리 공간을 뜻한다.
<br/>

> SELECT INTO 방식

- SELECT INTO문은 조회되는 데이터가 단 하나의 행일 때 사용 가능한 방식이다. 
- 커서는 결과 행이 하나이든 여러 개이든 상관없이 사용할 수 있다.
- SELECT INTO문은 SELECT절에 명시한 각 열의 결과 값을 다음과 같이 변수에 대입해 준다.

```sql
// 기본 형식
SELECT 열1, 열2, ..., 열n INTO 변수1, 변수2, ..., 변수n
FROM ...
///

```sql
// SELECT INTO를 사용한 단일행 데이터 저장하기
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

<br/>

> 명시적 커서

- 명시적 커서는 사용자가 직접 커서를 선언하고 사용하는 커서를 뜻한다.

| 단계 | 명칭 | 설명 |
|:-----:|:------:|:-------------:|
| 1단계 | 커서 선언<br/> (declaration) | 사용자가 직접 이름을 지정하여 사용할 커서를 SQL문과 함께 선언합니다. |
| 2단계 | 커서 열기<br/> (open) | 커서를 선언할 때 작성한 SQL문을 실행합니다. 이때 실행한 SQL문에 영향을 받는 행을 active set라 합니다. |
| 3단계 | 커서에서 읽어온<br/> 데이터 사용<br/> (fetch) | 실행된 SQL문의 결과 행 정보를 하나씩 읽어 와서 변수에 저장한 후 필요한 작업을 수행합니다. 각 행별로 공통 작업을 반복해서 실행하기 위해 여러 종류의 LOOP문을 함께 사용할 수 있습니다. |
| 4단계 | 커서 닫기<br/> (close) | 모든 행의 사용이 끝나고 커서를 종료합니다. |

```sql
// 기본 형식
DECLARE
	CURSOR 커서 이름 IS SQL문; -- 커서 선언(Declaration)
BEGIN
	OPEN 커서 이름;			  -- 커서 열기(Open)
    FETCH 커서이름 INTO 변수	 -- 커서로부터 읽어온 데이터 사용(Fetch)
    CLOSE 커서이름;			  -- 커서 닫기(Close)
END;
```

<br/>

### 하나의 행만 조회되는 경우

- 하나의 행만 조회되는 SELECT문을 커서로 지정하여 사용할 경우 SELECT INTO문을 사용할 때보다 복잡한 여러 단계를 작성해여 하므로 다소 번거롭다. 커서의 효용성은 조회되는 행이 여러 개일 때 극대화된다.

<br/>

### 여러 행이 조회되는 경우 사용하는 LOOP문

- 커서에 지정한 SELECT문이 여러 행을 결과 값을 가질 경우에 여러 방식의 LOOP문을 사용할 수 있다.

- %NOTFOUND는 실행된 FETCH문에서 행을 추출했으면 false, 추출하지 않았으면 true를 반환한다.
- FETCH문을 통해 더 이상 추출한 데이터가 없을 경우에 LOOP 반복이 끝난다.

| 속성 | 설명 |
|:------:|:------------:|
| 커서 이름%NOTFOUND | 수행된 FETCH문을 통해 추출된 행이 있으면 false, 없으면 true를 반환합니다. |
| 커서 이름%FOUND | 수행된 FETCH문을 통해 추출된 행이 있으면 true, 없으면 false를 반환합니다. |
| 커서 이름%ROWCOUNT | 현재까지 추출된 행 수를 반환합니다. |
| 커서 이름%ISOPEN | 커서가 열려(open) 있으면 true, 닫혀(close) 있으면 false를 반환합니다. |

<br/>

### 여러 개의 행이 조회되는 경우(FOR LOOP문)

- 커서에 FOR LOOP문을 사용하면 좀 더 간편하게 여러 행을 다룰 수 있다.

```sql
// 기본 형식
FOR 루프 인덱스 이름 IN 커서 이름 LOOP
   결과 행별로 반복 수행할 작업;
END LOOP;
```

<br/>

### 커서에 파라미터 사용하기

- 고정 값이 아닌 직접 입력한 값 또는 상황에 따라 여러 값을 번갈아 사용하려면 다음과 같이 커서에 파라미터를 지정할 수 있다.

```sql
// 기본 형식
CURSOR 커서 이름(파라미터 이름 자료형, ...) IS
SELECT ...
```

<br/>

> 묵시적 커서

- 묵시적 커서는 별다른 선언 없이 SQL문을 사용했을 때 오라클에서 자동으로 선언되는 커서를 뜻한다.
- 자동으로 생성되어 실행되는 묵시적 커서는 별다른 PL/SQL문을 작성하지 않아도 되지만, 다음 묵시적 커서의 속성을 사용하면 현재 커서의 정보를 확인할 수 있다.

| 속성 | 설명 |
|:------:|:------------:|
| SQL%NOTFOUND | 묵시적 커서 안에 추출한 행이 있으면 false, 없으면 true를 반환합니다. DML 명령어로 영향을 받는 행이 없을 경우에도 true를 반환합니다. |
| SQL%FOUND | 묵시적 커서 안에 추출한 행이 있으면 true, 없으면 false를 반환합니다. DML 명령어로 영향을 받는 행이 있다면 true를 반환합니다. |
| SQL%ROWCOUNT | 묵시적 커서에 현재까지 추출한 행 수 또는 DML 명령어로 영향받는 행 수를 반환합니다. |
| SQL%ISOPEN | 묵시적 커서는 자동으로 SQL문을 실행한 후 CLOSE되므로 이 속성은 항상 false를 반환합니다. |

---

