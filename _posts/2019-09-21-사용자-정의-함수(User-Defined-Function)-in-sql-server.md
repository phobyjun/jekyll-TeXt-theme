---
title: 사용자 정의 함수(User Defined Function) in sql server
excerpt: sql server에서의 사용자 정의 함수
tags: [database, udf, udfs, 사용자 정의 함수, sql server, dbms]
comment: true
---

일반적인 프로그래밍 언어와 같이 SQL Server 또한 UDFs(User Defined Functions)을 제공한다. SQL server로부터 2000개에 달하는 UDF들이 만들어졌다. UDF는 파라미터들을 입력받아 동작하고 그 동학의 결과를 반환해주는 프로그래밍 구조를 가진다. UDF의 결과는 스칼라 값 또는 결과 셋중 하나이다. UDFs는 스크립트, 프로시저 저장, 트리거, 그리고 데이터베이스 안의 다른 UDFs로 사용될 수 있다.

## UDF의 이점

1. UDFs는 모듈러 프로그래밍(modular programming)을 지원한다. 일단 UDF를 만들고 데이터베이스에 저장하면, 몇번이든 불러와 사용할 수 있다. 또한 소스 코드와 독립적으로 UDF를 수정할 수 있다.
2. UDFs는 캐싱 플랜(caching plan)과 반복 실행을 위한 재사용을 통해 T-SQL 코드의 compilation cost를 줄인다.
3. 네트워크 트래픽을 줄일 수 있다. 만약 복잡한 제약 조건(constraint)을 가진 데이터를 필터링(filter)하기를 원한다면, 이는 UDF로서 표현 될 수 있다. 그 다음, WHERE 절 안에서 이 UDF를 사용할 수 있다.

## UDF의 종류

1. Scalar Functions
2. Table Valued Functions

다음의 학생과 과목 테이블의 예를 보자:

![1.jpg](/assets/img/2019-09-21-1/1.jpg)

### 1. Scalar Functions

Scalar UDF는 0 또는 1 이상의 파라미터들을 받아 하나의 값을 반환한다. scalar function의 반환형(return type)은  text, ntext, image, cursor, timestamp를 제외한 어떤 형태이든 가능하다. Scalar function은 SQL Query의 WHERE절에서 사용될 수 있다.

**Creating Scalar Function**

Scalar Function을 생성할 때 다음의 문법이 사용된다.

```sql
CREATE FUNCTION function-name (Parameters)
RETURNS return-type
AS
BEGIN
	Statement 1
	Statement 2
		...
		...
	Statement n
	RETURN return-value
END
```

실제 함수를 생성하는 예는 다음과 같다.

```sql
CREATE FUNCTION GetStudent(@Rno INT)
	RETURNS VARCHAR(50)
	AS
BEGIN
	RETURN (SELECT Name FROM Student WHERE Rno=@Rno)
END
```

이 함수를 사용하기 위해 다음의 명령어를 사용한다.

```sql
PRINT dbo.GetStudent(1)
```

**Output**: Ram

### 2. Table Valued Functions

Table Valued UDF는 0 또는 1 이상의 파라미터들을 받아 table 변수를 반환한다. 이 함수의 형(type)은 특별하다. 그 이유는, 다른 table들과 합쳐진 결과에 질의(query)할 수 있는 table을 반환하기 때문이다. 또한, Table Valued function은 "Inline Table Valued Function"과 "Multi-Statement Table Valued Function"으로 세분화할 수 있다.

#### Inline Table Valued Function

Inline Table Valued Function은 SELECT절이어야만 하는 하나의 구문을 포함한다. 질의(query)의 결과는 함수의 반환값이 된다. Inline function에서는 BEGIN-END 블록(block)이 필요하지 않다.

**Creating Inline Table Valued Function**

Inline Table Valued Function을 생성할 때 다음의 문법이 사용된다.

```sql
CREATE FUNCTION function-name (Parameters)
RETURNS return-type
AS
RETURN
```

실제 함수를 사용하는 예는 다음과 같다.

```sql
CREATE FUNCTION GetAllStudents(@Mark INT)
RETURNS TABLE
AS
RETURNS
	SELECT *FROM Student WHERE Marks>=@Mark
```

이 함수를 사용하기 위해 다음의 명령어를 사용한다.

```sql
SELECT *FROM GetAllStudents(60)
```

**Output**: ![2.jpg](/assets/img/2019-09-21-1/2.jpg)

### Multi-Statement Table Valued Function

Multi-Statement Table Valued Function은 BEGIN-END 블록으로 둘러싸인 2개 이상의 SQL 문을 포함한다. 함수 안에서 데이터베이스로부터 데이터를 읽고 몇 가지 연산을 수행한다. Multi-Statement Table Valued Function에서 반환값은 table 변수로 선언되고 반환되어야 하는 table의 전체 구조를 포함한다. RETURN 절에는 값이 없고 반환되어야 하는 table 변수로 선언된다.

**Creating Multi-Statement Table Valued Function**

Multi-Statement Table Valued Function을 생성할 때 다음의 문법이 사용된다.

```sql
CREATE FUNCTION function-name (Parameters)
RETURNS @TableName TABLE
(Column_1 datatype,
		...
		...
 Column_1 datatype
)
AS
BEGIN
Statement 1
	Statement 2
		...
		...
	Statement n
	RETURN
END
```

실제 함수를 사용하는 예는 다음과 같다.

```sql
CREATE FUNCTION GetAvg(@Name varchar(50))
RETURNS @Marks TABLE
(Name VARCHAR(50),
 Subject1 INT,
 Subject2 INT,
 Subjent3 INT,
 Average DECIMAL(4, 2)
)
AS
BEGIN
	DECLARE @Avg DECIMAL(4, 2)
	DECLARE @Rno INT
	INSERT INTO @Marks (NAME)VALUES(@Name)
	SELECT @Rno=Rno FROM Student WHERE Name=@Name
	SELECT @Avg=(Subject1+Subject2+Subject3)/3 FROM Subjects WHERE Rno=@Rno
	UPDATE @Marks SET
	Subject1=(SELECT Subject1 FROM Subjects WHERE Rno=@Rno),
	Subject2=(SELECT Subject2 FROM Subjects WHERE Rno=@Rno),
	Subject3=(SELECT Subject3 FROM Subjects WHERE Rno=@Rno),
	Average=@Avg
	WHERE Name=@Name
RETURN
END
```

이 함수를 사용하기 위해 다음의 명령어를 사용한다.

```sql
SELECT * FROM GetAvg('Ram')
```

**Output**: ![3.jpg](/assets/img/2019-09-21-1/3.jpg)

> 출처: [https://www.c-sharpcorner.com/UploadFile/3194c4/user-defined-functions-in-sql-server/](https://www.c-sharpcorner.com/UploadFile/3194c4/user-defined-functions-in-sql-server/)

