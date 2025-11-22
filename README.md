# HTB-SQL_Injection_Fundamentals

## Table of Contents
1. [MySQL](#mysql)
    1. [Intro to MySQL](#intro-to-mysql)
    2. [SQL Statements](#sql-statements)
    3. [Query Results](#query-results)
    4. [SQL Operators](#sql-operators)


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

