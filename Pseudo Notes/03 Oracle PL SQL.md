# **Oracle PL SQL**

## **Date** : **`Nov 13, 2020`**   
## **Tags** : **`CSE 215-216 DATABASE`**

&nbsp;
# **General Structure**

```sql
DECLARE
	--<declarative section, optional>
	--variables are declared here

BEGIN
	--<executable section>
	--processing statements are written here

EXCEPTION
	--<exception code, optional>
	--exception handling codes are written here if any
END;
/
```

- **Every statement in a block ends with a semi-colon ( ; )**

# **Hello World**

```sql
BEGIN
	DBMS_OUTPUT.PUT_LINE('Hello World') ;
END ;
/
```

# **Declaring Variables**

Variables are initialized from SQL column name to PL SQL Variable using the `**INTO**` command

```sql
SELECT (FIRST_NAME ||' '|| LAST_NAME) INTO ENAME
```

And the variable needs to be declared in the **`DECLARE`** section with variable **name** and **type**

```sql
DECLARE
	ENAME VARCHAR2(100) ; -- variable name and type
BEGIN
	SELECT (FIRST_NAME ||' '|| LAST_NAME) INTO ENAME -- initialize
	FROM HR.EMPLOYEES
	WHERE EMPLOYEE_ID = 100 ;
	
	DBMS_OUTPUT.PUT_LINE('The name is: ' || ENAME) ;
END ;
/
```

# **Assigning Variables**

Variables can be assigned using `**:=`** operator

```sql
DECLARE
    EMPNAME VARCHAR2(64);
    JDATE DATE;
    MONTHS NUMBER;
BEGIN
	  SELECT FIRST_NAME||' '||LAST_NAME, HIRE_DATE INTO EMPNAME, JDATE FROM HR.EMPLOYEES WHERE EMPLOYEE_ID = 100;
		MONTHS := ROUND(MONTHS_BETWEEN(SYSDATE, JDATE),0);
		DBMS_OUTPUT.PUT_LINE('EMPLOYEE '||EMPNAME||' WORKED FOR '|| MONTHS || ' MONTHS');
END;
/
```

# **If else**

**`if`** **`elsif`** **`else`**

```sql
DECLARE
    EMPNAME VARCHAR2(64);
    JDATE DATE;
    MONTHS NUMBER;
    YEARS NUMBER;
BEGIN
    SELECT FIRST_NAME||' '||LAST_NAME, HIRE_DATE INTO EMPNAME, JDATE FROM HR.EMPLOYEES WHERE EMPLOYEE_ID = 100;
    MONTHS := ROUND(MONTHS_BETWEEN(SYSDATE, JDATE),0);
    YEARS := ROUND(MONTHS/12,0);
    
    IF YEARS>15 THEN
        DBMS_OUTPUT.PUT_LINE('EMPLOYEE IS SENIOR');
    ELSIF YEARS<5 THEN
        DBMS_OUTPUT.PUT_LINE('EMPLOYEE IS JUNIOR');
    ELSE
        DBMS_OUTPUT.PUT_LINE('EMPLOYEE IS NEITHER SENIOR NOR JUNIOR');
    END IF;
    
END;
/
```

# **Exception Handling**

**Common exceptions :**

- **`NO_DATA_FOUND`**
    - When a SELECT INTO statement cannot retrieve a row.
- **`TOO_MANY_ROWS`**
    - When a SELECT INTO statement retrieves more than one row.
- **`DUP_VAL_ON_INDEX`**
    - When duplicate values are attempted to be stored in a table column with unique index.
- **`NVALID_NUMBER`**
    - When the conversion of a character string into a number fails because the string does not represent a valid number.
- **`VALUE_ERROR`**
    - When an arithmetic, conversion, truncation, or size-constraint error occurs.
- **`ZERO_DIVIDE`**
    - When an attempt is made to divide a number by zero.
- **`OTHERS`**
    - When not sure about the type of exception

```sql
DECLARE
    EMPNAME VARCHAR2(64); --STORE EMPLOYEE NAME
    JDATE DATE; -- STORE JOINING DATE
    MONTHS NUMBER; -- VAR FOR CALCULATING MONTHS
    YEARS NUMBER; -- VAR FOR CALCULATING YEARS
BEGIN
    SELECT FIRST_NAME||' '||LAST_NAME, HIRE_DATE INTO EMPNAME, JDATE FROM HR.EMPLOYEES WHERE EMPLOYEE_ID = 1000;
    MONTHS := ROUND(MONTHS_BETWEEN(SYSDATE, JDATE),0);
    YEARS := ROUND(MONTHS/12,0);
    
    IF YEARS>15 THEN
        DBMS_OUTPUT.PUT_LINE('EMPLOYEE IS SENIOR');
    ELSIF YEARS<5 THEN
        DBMS_OUTPUT.PUT_LINE('EMPLOYEE IS JUNIOR');
    ELSE
        DBMS_OUTPUT.PUT_LINE('EMPLOYEE IS NEITHER SENIOR NOR JUNIOR');
    END IF;

EXCEPTION
    WHEN NO_DATA_FOUND THEN
        DBMS_OUTPUT.PUT_LINE('NO DATA FOUND');
    WHEN OTHERS THEN
        DBMS_OUTPUT.PUT_LINE('UNKNOWN ERROR');
END;
/
```

# **LOOPS**

- **For Loop**

    Syntax :

    ```sql
    FOR variable in lowest val..highest val
    LOOP
    	Statement1 ;
    	Statement2 ;
    	…
    END LOOP ;
    ```

    Example :

    ```sql
    DECLARE
    BEGIN
    	FOR i in 1..5
    	LOOP
    		DBMS_OUTPUT.PUT_LINE(i);
    	END LOOP ;
    End ;
    /
    ```

- **While Loop**

    Syntax :

    ```sql
    WHILE Condition -- declare variables needed here in the DECLARE section 
    LOOP
    	Statement1 ;
    	Statement2 ;
    	…
    END LOOP ;
    ```

    Example :

    ```sql
    DECLARE
        i NUMBER;
    BEGIN
        i := 1;
        WHILE i<=100
        LOOP
            DBMS_OUTPUT.PUT_LINE(i);
            i := i+1;
        END LOOP;
    End ;
    /
    ```

- **Unconditional Loop**

    ```sql
    DECLARE
    	i number;
    BEGIN
    --this is an unconditional loop, must have EXIT WHEN inside loop
    	i := 1;
    	LOOP
    		DBMS_OUTPUT.PUT_LINE(i);
    		i := i + 1;
    		EXIT WHEN (i > 100) ;
    	END LOOP ;
    End ;
    /
    ```

    **`EXIT WHEN`** can be used inside **for** / **while** loop too

Loop use case in PL SQL:

- iterating/computing on multiple row returned by a query (Cursor)

# **Cursor**

Syntax :

```sql
FOR variable in (SELECT statement) -- iterating over multiple rows
LOOP
	Statement1 ;
	Statement2 ;
	…
END LOOP ;
```

Example :

```sql
-- Find the number of employees working for more than 10 years
DECLARE
    N NUMBER;
    YEARS NUMBER;
BEGIN
    N :=0;
    FOR R IN (SELECT EMPLOYEE_ID, HIRE_DATE FROM HR.EMPLOYEES) -- cursor
    LOOP
        YEARS := ROUND(MONTHS_BETWEEN(SYSDATE,R.HIRE_DATE)/12,0);
        IF YEARS>10 THEN
            N := N+1;
        END IF;
    END LOOP;
    
    DBMS_OUTPUT.PUT_LINE('Number of employees working for more than 10 years: ' || N);
END;
/
```

```sql
--Add 1% commission for the employees who get more than the overall average salary.
DECLARE
    OLD_COM_PCT NUMBER;
    NEW_COM_PCT NUMBER;
BEGIN
    FOR R IN (SELECT EMPLOYEE_ID, SALARY FROM HR.EMPLOYEES2 WHERE SALARY>(SELECT AVG(SALARY) FROM HR.EMPLOYEES2))
    LOOP
        SELECT COMMISSION_PCT INTO OLD_COM_PCT FROM HR.EMPLOYEES2 WHERE EMPLOYEE_ID = R.EMPLOYEE_ID;

        NEW_COM_PCT := 1.01*OLD_COM_PCT;
        
        UPDATE HR.EMPLOYEES2 SET COMMISSION_PCT = NEW_COM_PCT WHERE EMPLOYEE_ID = R.EMPLOYEE_ID;
        
        --Check whether the transaction is reflected
        DBMS_OUTPUT.PUT_LINE(OLD_COM_PCT);
        
        SELECT COMMISSION_PCT INTO OLD_COM_PCT FROM HR.EMPLOYEES2 WHERE EMPLOYEE_ID = R.EMPLOYEE_ID;
        
        DBMS_OUTPUT.PUT_LINE(OLD_COM_PCT);
    END LOOP;
END;
/
```

# **Procedure**

will not return anything , if you want some variable to change their value inside the **procedure** , then you have to pass that variable as a parameter . Can be executed inside PL-SQL block and also outside using **`EXEC procudere_name`** command

Syntax : 

```sql
CREATE OR REPLACE PROCEDURE procedure_name (parameter_name  IN/OUT parameter_datatype , ... )
AS
	-- declare local variables
BEGIN
	-- routine body goes here, e.g.
END;
/
```

Example :

```sql
-- declaring procedure
CREATE OR REPLACE PROCEDURE IS_SENIOR_EMPLOYEE(EID IN VARCHAR2) 
AS
	-- declare local variables
	JDATE DATE ;
	YEARS NUMBER ;
BEGIN
	SELECT HIRE_DATE INTO JDATE
	FROM EMPLOYEES
	WHERE EMPLOYEE_ID = EID ;
	YEARS := (MONTHS_BETWEEN(SYSDATE, JDATE) / 12) ;
	IF YEARS >= 10 THEN
		DBMS_OUTPUT.PUT_LINE('The employee worked 10 years or more') ;
		ELSE
		DBMS_OUTPUT.PUT_LINE('The employee worked less than 10 years') ;
	END IF ;
END ;
/

-- calling procedure
DECLARE
BEGIN
	IS_SENIOR_EMPLOYEE(100) ;
	IS_SENIOR_EMPLOYEE(105) ;
END ;
```

```sql
-- Output from procedure
CREATE OR REPLACE PROCEDURE IS_SENIOR_EMPLOYEE(EID IN VARCHAR2, MSG OUT VARCHAR2)
AS
	JDATE DATE ;
	YEARS NUMBER ;
BEGIN
	SELECT HIRE_DATE INTO JDATE
	FROM EMPLOYEES
	WHERE EMPLOYEE_ID = EID ;
	YEARS := (MONTHS_BETWEEN(SYSDATE, JDATE) / 12) ;
	IF YEARS >= 10 THEN
		MSG := 'The employee worked 10 years or more' ;
	ELSE
		MSG := 'The employee worked less than 10 years' ;
	END IF ;
EXCEPTION
	WHEN NO_DATA_FOUND THEN
		MSG := 'No employee found.' ;
	WHEN TOO_MANY_ROWS THEN
		MSG := 'More than one employee found.' ;
	WHEN OTHERS THEN
		MSG := 'Some unknown error occurred.' ;

-- calling procedre
DECLARE
MESSAGE VARCHAR2(100) ;
BEGIN
	IS_SENIOR_EMPLOYEE(10000, MESSAGE) ;
	DBMS_OUTPUT.PUT_LINE(MESSAGE) ;
	IS_SENIOR_EMPLOYEE(105, MESSAGE) ;
	DBMS_OUTPUT.PUT_LINE(MESSAGE) ;
END ;
/
```

# **Function**

Unlike procedure , function can 

- return values
- used inside **`SELECT`** statement

Syntax : 

```sql
CREATE OR REPLACE
FUNCTION function_name (parameter_name parameter_datatype , ... )
RETURN return_type
AS
-- declare local variables
BEGIN
	-- routine body goes here, e.g.
	-- DBMS_OUTPUT.PUT_LINE('Navicat for Oracle');
	RETURN NULL;
END;
```

Example :

```sql
CREATE OR REPLACE FUNCTION GET_SENIOR_EMPLOYEE(EID IN VARCHAR2)
RETURN VARCHAR2 IS
-- declare local variables
	JDATE DATE ;
	YEARS NUMBER ;
	MSG VARCHAR2(100) ;
BEGIN
	SELECT HIRE_DATE INTO JDATE
	FROM EMPLOYEES
	WHERE EMPLOYEE_ID = EID ;
	YEARS := (MONTHS_BETWEEN(SYSDATE, JDATE) / 12) ;
	IF YEARS >= 10 THEN
		MSG := 'The employee worked 10 years or more' ;
	ELSE
		MSG := 'The employee worked less than 10 years' ;
	END IF ;
	RETURN MSG ; --return the message
EXCEPTION
--you must return value from this section also
	WHEN NO_DATA_FOUND THEN
		RETURN 'No employee found.' ;
	WHEN TOO_MANY_ROWS THEN
		RETURN 'More than one employee found.' ;
	WHEN OTHERS THEN
		RETURN 'Some unknown error occurred.' ;
END ;
/

-- calling function
DECLARE
	MESSAGE VARCHAR2(100) ;
BEGIN
	MESSAGE := GET_SENIOR_EMPLOYEE(10000) ;
	DBMS_OUTPUT.PUT_LINE(MESSAGE) ;
	MESSAGE := GET_SENIOR_EMPLOYEE(105) ;
	DBMS_OUTPUT.PUT_LINE(MESSAGE) ;
	END ;
/

-- using in a query
SELECT employee_id, GET_SENIOR_EMPLOYEE(employee_id)
FROM employees
```

# **Trigger**

Automatically call when a change in database occurs 

- **row level triggers :** if n rows are updated => n times call
- **statement level :**  1 time call only

**Structure** :

```sql
CREATE OR REPLACE TRIGGER trigger_name
BEFORE/AFTER INSERT/DELETE/UPDATE
OF column_name
ON table_name 
WHEN condition
-- + [for each row] for row level, default statement level
-- + DECLARE varaibles needed in the trigger
```

**Various usages** :

```sql
--This trigger will run before insert on STUDENTS table

CREATE OR REPLACE TRIGGER HELLO_WORLD2
BEFORE INSERT
ON STUDENTS
DECLARE
BEGIN
DBMS_OUTPUT.PUT_LINE('Hello World2');
END ;
/
```

```sql
--This trigger will run after an insert or a delete statement on STUDENTS table

CREATE OR REPLACE TRIGGER HELLO_WORLD3
AFTER INSERT OR DELETE - or
ON STUDENTS
DECLARE
BEGIN
DBMS_OUTPUT.PUT_LINE('Hello World3');
END ;
/
```

```sql
--The following trigger will run after an update statement on STUDENTS table.
--Added with that, this trigger will only run when update is performed on the
--CGPA column.

CREATE OR REPLACE TRIGGER HELLO_WORLD4
AFTER UPDATE
OF CGPA -- only run when update is performed on the CGPA column.
ON STUDENTS
DECLARE
BEGIN
DBMS_OUTPUT.PUT_LINE('Hello World4');
END ;
/
```

```sql
--The following trigger will run after an update operation on the STUDENTS table.
--Added with that, the trigger will run once for each row. 
--The previous rows will run once for the whole statement, where this trigger will run N times if N rows
--are affected by the SQL statement.

CREATE OR REPLACE TRIGGER HELLO_WORLD5
AFTER UPDATE
OF CGPA
ON STUDENTS
FOR EACH ROW -- for each row
DECLARE
BEGIN
DBMS_OUTPUT.PUT_LINE('Hello World5');
END ;
/
```

**Example** :

```sql
-- creating the table
CREATE TABLE STUDENTS(
STUDENT_NAME VARCHAR2(250),
CGPA NUMBER
) ;

-- creating trigger
CREATE OR REPLACE TRIGGER HELLO_WORLD
AFTER INSERT
ON STUDENTS
DECLARE
BEGIN
DBMS_OUTPUT.PUT_LINE('Hello World');
END ;
/

-- inserting into table 
INSERT INTO STUDENTS VALUES ('Fahim Hasan', 3.71);
INSERT INTO STUDENTS VALUES ('Ahmed Nahiyan', 3.80);
```

- If a new employee is recorded in database, update max salary and min salary

    ```sql
    CREATE OR REPLACE TRIGGER DEMO_TRIGGER 
    AFTER INSERT
    ON HR.EMPLOYEES
    DECLARE
        MX_SAL NUMBER;
        MIN_SAL NUMBER;
        
        OLD_MAX_SAL NUMBER;
        OLD_MIN_SAL NUMBER;
    BEGIN
        FOR R IN (SELECT DISTINCT JOB_ID FROM HR.JOBS)
        LOOP
            SELECT MAX(SALARY), MIN(SALARY) INTO MX_SAL, MIN_SAL FROM HR.EMPLOYEES WHERE JOB_ID = R.JOB_ID;
        
            SELECT MAX_SALARY, MIN_SALARY INTO OLD_MAX_SAL, OLD_MIN_SAL FROM HR.JOBS WHERE JOB_ID = R.JOB_ID;
            
            IF OLD_MAX_SAL<>MX_SAL THEN
                UPDATE HR.JOBS SET MAX_SALARY = MX_SAL WHERE JOB_ID = R.JOB_ID;
            END IF;
            
            IF OLD_MIN_SAL<>MIN_SAL THEN
                UPDATE HR.JOBS SET MIN_SALARY = MIN_SAL WHERE JOB_ID = R.JOB_ID;
            END IF;
        END LOOP;
        
    END;
    /
    ```

- Write a PL/SQL trigger LOG_CGPA_UPDATE. This trigger will log all updates done on the CGPA column of STUDENTS table. The trigger will save current user’s name, current system date and time in a log table named LOG_TABLE_CGPA_UPDATE.

    ```sql
    CREATE TABLE LOG_TABLE_CGPA_UPDATE
    (
    USERNAME VARCHAR2(25),
    DATETIME DATE
    ) ;

    CREATE OR REPLACE TRIGGER LOG_CGPA_UPDATE
    AFTER UPDATE
    OF CGPA
    ON STUDENTS
    DECLARE
    USERNAME VARCHAR2(25);
    BEGIN
    USERNAME := USER; --USER is a function that returns current username
    INSERT INTO LOG_TABLE_CGPA_UPDATE VALUES (USERNAME, SYSDATE);
    END ;
    /

    --First update
    UPDATE STUDENTS SET CGPA = CGPA + 0.01 ;
    --Another update
    UPDATE STUDENTS SET CGPA = CGPA - 0.01 ;
    --View the rows inserted by the trigger
    SELECT * FROM LOG_TABLE_CGPA_UPDATE
    ```

- Write a PL/SQL trigger BACKUP_DELETED_STUDENTS. This trigger will save all records that are deleted from the STUDENTS table into a backup table named STUDENTS_DELETED. The trigger will save student’s record along with current user’s name and current system date and time.

    ```sql
    CREATE TABLE STUDENTS_DELETED(
    STUDENT_NAME VARCHAR2(25),
    CGPA NUMBER,
    USERNAME VARCHAR2(25),
    DATETIME DATE
    ) ;

    CREATE OR REPLACE TRIGGER BACKUP_DELETED_STUDENTS
    BEFORE DELETE
    ON STUDENTS
    FOR EACH ROW
    DECLARE
    V_NAME VARCHAR2(25);
    V_USERNAME VARCHAR2(25);
    V_CGPA NUMBER;
    V_DATETIME DATE;
    BEGIN
    V_NAME := :OLD.STUDENT_NAME ;
    V_CGPA := :OLD.CGPA ;
    V_USERNAME := USER ;
    V_DATETIME := SYSDATE ;
    INSERT INTO STUDENTS_DELETED VALUES (V_NAME,V_CGPA,V_USERNAME,V_DATETIME);
    END ;
    /

    --Delete the two rows
    DELETE FROM STUDENTS WHERE CGPA < 3.85 ;
    --View the rows inserted by the trigger
    SELECT * FROM STUDENTS_DELETED ;
    ```

Note the use of `**:OLD**` special reference. This reference is used to access the column values of **currently affected row** by the `DELETE` operation. Like the `:OLD` reference, there is a `:NEW` reference that can be used to retrieve the column values of the new row that will result after completion of the operation

- Write a PL/SQL trigger CORRECT_STUDENT_NAME. This trigger will be used to correct the text case of the student names when it is going to be inserted. So, this will be a BEFORE INSERT trigger. The trigger will change the case of the student’s name to INITCAP format if it was not given by the user in the INSERT statement. The trigger will thus ensure that all names stored in the STUDENTS table will be in a consistent forma

    ```sql
    CREATE OR REPLACE TRIGGER CORRECT_STUDENT_NAME
    BEFORE INSERT
    ON STUDENTS
    FOR EACH ROW
    DECLARE
    BEGIN
    	:NEW.STUDENT_NAME := INITCAP(:NEW.STUDENT_NAME) ;
    END ;
    /

    --Issue the following SQL statements and then view the rows of STUDENTS table
    INSERT INTO STUDENTS VALUES ('SHAkil ahMED', 3.80);
    INSERT INTO STUDENTS VALUES ('masum billah', 3.60);
    ```

    Note that the above trigger must be a **`BEFORE INSERT`** trigger. If you write an **`AFTER INSERT`** trigger, then it will not work. Because in an **`AFTER INSERT`** trigger, **you are not allowed to change values of the `:NEW` row**.

- Write a trigger that will save a student records in a table named LOW_CGPA_STUDENTS which contain only one column to store student’s names. The trigger will work before an update operation or an insert operation. Whenever the update operation results in a CGPA value less than 2.0, the trigger will be fired and the trigger will save the students name in the LOW_CGPA_STUDENTS table. Similarly, when an insert operation inserts a new row with CGPA less than 2.0, the corresponding row must be saved in the LOW_CGPA_STUDENTS table.

- raising error to deny insertion / deletion / update

    ```sql
    CREATE OR REPLACE TRIGGER INVALID_NAME
    BEFORE UPDATE OR INSERT
    ON STUDENTS
    FOR EACH ROW
    DECLARE 
     NAME VARCHAR2(50);
    BEGIN
     NAME := :NEW.STUDENT_NAME;
     IF REGEXP_LIKE (NAME,'[^A-Za-z]') THEN
      RAISE_APPLICATION_ERROR(-20222, 'INVALID NAME DISISH');
     END IF;
    END;
    /

    INSERT INTO STUDENTS VALUES('Z23xC', 3.9);
    ```