## 分页查询
/*
语法： select 查询列表 from 表 [其他筛选] LIMIT 起始条目索引，显示的条目数
*/
-- 查询11~25条员工信息
SELECT * FROM employees
LIMIT 10, 15;

-- 查询有奖金的员工信息，并且工资较高的前10名显示出来
SELECT * FROM employees
where commission_pct is not null
ORDER BY salary DESC LIMIT 0, 10;

## 联合查询
/*
union : 将多个查询结果合并成一个结果
语法： 查询语句1 union 查询语句2
应用场景：当我们需要查询的数据来自于多张表，而这两张表没有任何的连接关系，那么比较适合用连接查询。
注意：1. 使用union时会默认会去重，如果不想去重需要使用 union all 
*/

-- 例如：查询部门编号 >90 或 邮箱包含 a 的员工
SELECT * from employees WHERE department_id > 90 OR email LIKE '%a%';
SELECT * from employees WHERE department_id > 90 UNION ALL SELECT * from employees WHERE email LIKE '%a%';


