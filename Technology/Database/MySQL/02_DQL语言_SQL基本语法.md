# SQL 基本语法
```sql
# 基础查询语句
USE myemployees;
## 按列查询
SELECT `first_name` ,employee_id, email, phone_number, job_id FROM employees;
SELECT * from employees;

## 起别名，结果列
-- 使用AS关键字 或 空格
SELECT last_name AS lastName FROM employees;
SELECT last_name "lastName" FROM employees;

## 去重:DISTINCT,表示去重。但是如果有多个字段时这个去重条件就会变成多个字段同时满足才会去重。
-- 查询员工表里面涉及到的部门。 这里虽然使用distinct但是条件是必须 department_id = manager_id 都相同时才会去重
SELECT DISTINCT department_id, manager_id
FROM employees

## 条件查询
-- 逻辑运算符查询。查询编号不是在 90 到 110 之间，或者工资高于 15000 的
SELECT last_name, salary, commission_pct, department_id from employees
where NOT (department_id >= 90 and department_id <= 110) OR salary > 150000;

## 模糊查询
/*
like: 一般和通配符一起使用。%任意多个字符,包含0个字符;_任意单个字符。
between and : 查询区间,包含前后值，但是注意的是这两个值不能颠倒。完全等价于：id >= 100 and id <= 120;
in：查询多个符合条件的值，等价于 id = 1 or id = 2,
is null or is not null：查询必须使用 is 关键字，不能直接使用 = null 方式
*/
-- 查询员工名中包含 xx 的员工
select * from employees
where last_name LIKE '%a%';

-- 查询员工名中第三个为n， 第五个字符为l的员工和工资
select last_name, salary from employees
where last_name LIKE '__n_l%';

-- Like:查询员工名中第二个字符为_的员工名。可以用过使用\转义字符或使用关键字 escape 
select last_name, salary from employees
where last_name LIKE '_\_%';

-- 查询员工编号在100~120之间
select * from employees
where employee_id BETWEEN 100 AND 120;
-- bewteen and 完全等价于下面的语句，建议还是用逻辑运算符判断
select * from employees
where employee_id >= 100 and employee_id <= 120;
-- 查询多个符合条件的值
SELECT * from employees
where job_id in ('AD_VP', 'IT_PROG');

-- 查询 is null
select last_name, commission_pct from employees
where commission_pct is not null;

## 排序查询
-- ORDER BY: asc默认升序, desc 降序
-- 按照年薪从低到高排序, 按照表达式排序
SELECT *, salary * 12 * (1 + IFNULL(commission_pct,0)) AS 年薪 from employees
WHERE department_id >= 90
ORDER BY salary * 12 * (1 + IFNULL(commission_pct,0)) ASC;
-- 按照年薪从低到高排序, 按照别名排序
SELECT *, salary * 12 * (1 + IFNULL(commission_pct,0)) AS 年薪 from employees
WHERE department_id >= 90
ORDER BY 年薪 ASC;

-- 按多个字段排序：在保障第一个条件数据的排序之后如果第二个条件有相同的值时那么按照第二个条件排序。
-- 先按工资降序，然后在按员工编号升序。这个排序的意思是先按照salary进行排序，在按照employee_id进行排序
SELECT * from employees
ORDER BY salary DESC, employee_id ASC;

## 函数调用
-- 语法: select function(args) [from table]
SELECT NOW();
SELECT * FROM employees WHERE hiredate = STR_TO_DATE('4-3 1992','%c-%d %Y');
-- CASE 函数，类似于Java中的 switch() case 语法
-- 根据不同部门，显示工资*基础
select salary 原工资 

## 流程控制
SELECT IF(10 < 5,"大","小");

-- count 函数
-- 统计行数
SELECT COUNT(*) FROM employees;
SELECT count(1) from employees;	

select AVG(salary) from employees;

-- 查询两个日期之间相差的天数
SELECT DATEDIFF(MAX(hiredate), MIN(hiredate)) AS '相差时间' 
FROM employees;

-- 查询最高工资与最低工资的差距
select max(salary) - min(salary) diffrent from employees;
```
