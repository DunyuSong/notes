## 子查询
### 标量 子查询（单行子查询）
-- 返回公司工资最少的员工的last_name, job_id 和 salary
SELECT last_name, job_id, salary, employee_id
from employees
WHERE salary = (
		SELECT MIN(salary) AS '工资最少'
		from employees
);
-- 查询最低工资大于50号部门最低工资的部门ID和其最低工资
-- 查询每个部门的最低工资
SELECT MIN(salary), department_id
FROM employees
GROUP BY department_id
HAVING MIN(salary) > (
		-- 查询出50号部门的最低工资
		SELECT Min(salary)
		FROM employees
		where department_id = 50
);
### 列 子查询（多行子查询,一列多行）
-- 查询location id 是1400 或1700的部门中的所有员工姓名
select last_name 
from employees
where department_id in(
		select department_id
		FROM departments
		where location_id IN(1400, 1700)
);
### 行 子查询（多列多行）
SELECT * FROM employees
WHERE (employee_id, salary)=(
		SELECT MIN(employee_id), MAX(salary)
		FROM employees
)
-- 或
SELECT * FROM employees
WHERE employee_id = (
		SELECT MIN(employee_id) FROM employees
)AND salary=(
		SELECT MAX(salary) FROM employees
);

### 子查询放在 select 中使用
-- 查询每个部门的员工个数
SELECT d.*, (
		select count(*) from employees e
		where e.department_id = d.department_id
) AS '个数' 
FROM departments d;
### 子查询放在 from 中使用
-- 需要将查询结果的充当一张表，必须要起别名
-- 查询每个部门的平均薪资等级



### 放在 exists后面（相关子查询）
/*
EXISTS(完整的查询语句)，最终返回结果0或1
*/
-- 查询有员工的部门名
SELECT department_name
FROM departments d
WHERE EXISTS(
	SELECT * 
		FROM employees e
		WHERE d.department_id = e.department_id
);























