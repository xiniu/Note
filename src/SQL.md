# SQL

## Preface

### Table Used in This Book

For postgresql, I found a script to create EMP and DEPT TAB.
https://github.com/rsp/pg-scott/blob/master/pg-scott.sql
```SQL
begin;

create table dept (
  deptno integer,
  dname  text,
  loc    text,
  constraint pk_dept primary key (deptno)
);

create table emp (
  empno    integer,
  ename    text,
  job      text,
  mgr      integer,
  hiredate date,
  sal      integer,
  comm     integer,
  deptno   integer,
  constraint pk_emp primary key (empno),
  constraint fk_mgr foreign key (mgr) references emp (empno),
  constraint fk_deptno foreign key (deptno) references dept (deptno)
);

create table salgrade (
  grade integer,
  losal integer,
  hisal integer,
  constraint pk_salgrade primary key (grade)
);

/*
create table bonus (
  ename text,
  job   text,
  sal   integer,
  comm  integer
);
*/


-- DML - Data Manipulation Language:

insert into dept (deptno,  dname,        loc)
       values    (10,     'ACCOUNTING', 'NEW YORK'),
                 (20,     'RESEARCH',   'DALLAS'),
                 (30,     'SALES',      'CHICAGO'),
                 (40,     'OPERATIONS', 'BOSTON');

insert into salgrade (grade, losal, hisal)
       values        (1,      700,  1200),
                     (2,     1201,  1400),
                     (3,     1401,  2000),
                     (4,     2001,  3000),
                     (5,     3001,  9999);

insert into emp (empno, ename,    job,        mgr,   hiredate,     sal, comm, deptno)
       values   (7369, 'SMITH',  'CLERK',     7902, '1980-12-17',  800, NULL,   20),
                (7499, 'ALLEN',  'SALESMAN',  7698, '1981-02-20', 1600,  300,   30),
                (7521, 'WARD',   'SALESMAN',  7698, '1981-02-22', 1250,  500,   30),
                (7566, 'JONES',  'MANAGER',   7839, '1981-04-02', 2975, NULL,   20),
                (7654, 'MARTIN', 'SALESMAN',  7698, '1981-09-28', 1250, 1400,   30),
                (7698, 'BLAKE',  'MANAGER',   7839, '1981-05-01', 2850, NULL,   30),
                (7782, 'CLARK',  'MANAGER',   7839, '1981-06-09', 2450, NULL,   10),
                (7788, 'SCOTT',  'ANALYST',   7566, '1982-12-09', 3000, NULL,   20), -- date fixed
                (7839, 'KING',   'PRESIDENT', NULL, '1981-11-17', 5000, NULL,   10),
                (7844, 'TURNER', 'SALESMAN',  7698, '1981-09-08', 1500,    0,   30),
                (7876, 'ADAMS',  'CLERK',     7788, '1983-01-12', 1100, NULL,   20), -- date fixed
                (7900, 'JAMES',  'CLERK',     7698, '1981-12-03',  950, NULL,   30),
                (7902, 'FORD',   'ANALYST',   7566, '1981-12-03', 3000, NULL,   20),
                (7934, 'MILLER', 'CLERK',     7782, '1982-01-23', 1300, NULL,   10);

commit;
```

And I Test script in this website:https://sqliteonline.com/


## Chapter 1 Retrieving Recoard

Very basic SELECT statements;

- 1.1 Retrieving ALL

```SQL
select * from emp;
-- in code, it's better to list every column individually
select empno, ename, job, mgr, hiredate, sal, comm, deptno from emp;
```

- 1.2 Retriev Subset Result

```SQL
--  you can use > < >= <= <> ! 
select * from emp where deptno = 10
```

- 1.3 Multipal Conditions

```SQL
-- use and/or/parenthesi
select * from emp
where deptno = 10 
  or comm is not null
  or sal < 2000 and deptno = 20;
  
-- what if ?
select * from emp
where (deptno = 10 
  or comm is not null
  or sal < 2000) and deptno = 20
--

- 1.4 Retrieving Subset columns

```SQL
select ename, deptno, sal from emp;
```

- 1.5 Meaningful Names for columns

```SQL
select sal as salary, comm as commision
	from emp;
```

- 1.6 Reference aliased column in where clause

```SQL
-- you need a inline view
select * from (select sal as salary, comm as commision
	from emp)x where salary < 1000;
```

- 1.7 Concatenating columns value

```SQL

-- for db2 oracle postgresql
select ename ||  'WORKS AS A ' || job as msg from emp 
	where deptno = 10
  
-- for MS SQL
select ename +  ' WORKS AS A ' + job as msg from emp 
	where deptno = 10
```

- 1.8 Using condition logic in a select statement

```SQL
select ename, sal,
	case when sal <= 2000 then 'UNDERPAIED'
    	 WHEN sal >= 4000 THEn 'OVERPAIED'
         else 'OK'
    end as status
    from emp
```

- 1.9 Limiting the numer of rows returned

```SQL

-- for postgresql and mysql
select *
    from emp limit 5;

-- for ms sql server
select top 5 * from emp

- for oracle
select * from from emp where rownum <= 5
```

- 1.10 Returnning n random records

```SQL

-- for postgresql
select *
    from emp ORDER by random() limit 5;
-- for ms sql server
select top 5 * 
  from amp order by newid()
 
-- for oracle   
select * from (select ename, job
  from emp ordery by dbms_random.value() 
  ) where rowum <= 5
 
```
It is important that you don't confuse using a function in the ORDER BY clause with using a numeric constant. When specifying a numeric constant in the ORDER BY clause, you are requesting that the sort be done according the column in that ordinal position in the SELECT list. When you specify a function in the ORDER BY clause, the sort is performed on the result from the function as it is evaluated for each row.


- 1.12 Transformint null to reat Value

```
-- this is more easier
select COALESCE (comm, 0) from emp;

select CASE when comm is null then 0 
	else comm
    END
    from emp;
```

- 1.13 Searching for partterns

```SQL
select ename, job from emp
where deptno in (10,20)
and (ename like '%I%' or job like '%ER')

-- % used for match any sequence of characters; _ for match single character in most sql implement
```






































































