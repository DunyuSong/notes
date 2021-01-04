## 分组查询
/*	
语法：
	select 查询的字段，分组函数
	from 表
	group by 分组的字段
	特点：
	1、可以按单个字段分组
	2、和分组函数一同查询的字段最好是分组后的字段
	3、分组筛选		
								针对的表							位置							关键字
	分组前筛选：	原始表						group by的前面				where
	分组后筛选：	分组后的结果集		group by的后面				having
*/
-- 查询哪个部门下的员工个数大于2
SELECT COUNT(*) AS 员工数, department_id FROM employees
group by department_id
having 员工数 > 2
-- 查询每个工种有奖金的员工的最高工资 >12000的工种编号和最高工资
SELECT job_id, salary
from employees
where salary > 12000 and commission_pct is not NULL
ORDER BY salary desc;

select job_id, salary
from employees
where commission_pct is not NULL
group by job_id
having salary > 12000;

-- 查询领导编号>102的每个领导手下的最低工资>5000的领导编号是哪个，以及其最低工资
SELECT MIN(salary) AS MIN最低工资, manager_id
FROM employees
where manager_id > 102
group by manager_id
having  MIN最低工资 > 5000;

## 多表 连接查询
/*
	语法：
	select 字段，...
	from 表1
	【inner|left outer|right outer|cross】join 表2 on  连接条件
	【inner|left outer|right outer|cross】join 表3 on  连接条件
	【where 筛选条件】
	【group by 分组字段】
	【having 分组后的筛选条件】
	【order by 排序的字段或表达式】
*/
### 内连接

#### 等值连接：筛选条件使用=连接，取交集。注意笛卡尔集 现象。
-- 查询 beauty 表中中boyfriend_id 与 boys表中id相等的数据
SELECT `name`, boyName
FROM beauty b inner join boys bo
ON b.boyfriend_id = bo.id;

-- 查询城市名称中第二个字符为o的部门名和城市名
SELECT l.city, d.department_name
FROM locations l inner join departments d
ON d.location_id = l.location_id;
WHERE l.city LIKE '_o%';

-- 查询有奖金的每个部门的部门名和部门的领导编号和该部门的最低工资
SELECT d.department_name, d.manager_id, d.department_id, MIN(e.salary) AS '最低工资'
FROM employees e inner join departments d
ON e.department_id = d.department_id
WHERE e.commission_pct IS NOT NULL
group by d.department_name;

-- 查询员工名、部门名、和所在的城市
SELECT e.last_name, d.department_name, l.city
from employees e 
INNER JOIN departments d 
ON e.department_id = d.department_id 
INNER JOIN locations l
ON d.location_id = l.location_id
ORDER BY city DESC;


#### 非等值连接
-- 查询员工工资和工资级别
SELECT e.last_name, j.grade_level
FROM employees e 
INNER JOIN job_grades j
ON e.salary > j.lowest_sal AND e.salary < j.highest_sal
WHERE j.grade_level = 'A';

#### 自连接
-- 相当于等值连接，只是自连接涉及到的表只有自己的表。
-- 查询员工名及上级名称
SELECT e.employee_id, e.last_name, m.employee_id, m.last_name
FROM employees e 
INNER JOIN employees m
ON e.manager_id = m.employee_id
WHERE e.last_name LIKE '%k%';

### 外连接
/*
应用场景：用于查询一个表中有，另一个表中没有的记录
查询结果：外连接查询结果=内连接结果 + 主表中有而从表没有的记录。
左外连接：left join 左边的是主表
右外连接： right join 右边的是主表
主表：一般来说将你最终要查询的结果信息是来自于哪张表，那么这张表就作为主表。
*/
#### 左外连接
-- 查询男朋友不在男神表的女神名
SELECT be.`name`, b.*
FROM beauty be
left outer join boys b
ON be.boyfriend_id = b.id
WHERE b.id IS NULL;

#### 右外连接
SELECT b.`name`, be.*
FROM boys be
right outer join beauty b
ON b.boyfriend_id = be.id
WHERE be.id IS NULL;

-- 查询哪个部门没有员工
SELECT d.*, e.employee_id
FROM departments d
left outer JOIN employees e
ON e.department_id = d.department_id
WHERE employee_id is NULL;
-- GROUP BY department_id
-- HAVING employee_id is NULL;

#### 交叉连接（笛卡尔乘积）
-- 交叉连接一般不用，相当于 A * B
SELECT b.* , bo.*
FROM beauty b
CROSS JOIN boys bo
-- ON b.boyfriend_id = bo.id;	-- 内连接效果了




















