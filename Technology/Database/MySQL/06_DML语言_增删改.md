## 插入语句
/*
语法：insert into 表名(列名,...) values(值1,...)
支持嵌套子查询、支持批量插入值
*/

insert into beauty(id, name, sex, borndate, phone, boyfriend_id)
values (13, '刘亦菲', '女', '1990-11-23', '15606690055', 3);

-- 使用子查询方式，将多个值查询出来插入
insert into beauty( name, sex, borndate, phone, boyfriend_id)
SELECT b.`name`, b.sex, b.borndate, b.phone, b.boyfriend_id from beauty b WHERE b.id < 3;

SELECT * from beauty;

## 修改语句
/*
UPDATE 表名 SET 列=新值，。。。 
WHERE 筛选条件;
*/
-- 修改单表
UPDATE beauty SET phone='11561565641', borndate='1990-11-23' WHERE id < 3;
-- 修改多表

## 删除语句
/*
语法：delete from 表名 where 筛选条件
*/
DELETE FROM beauty WHERE phone LIKE '%9';
-- 删除多表
DELETE b, bo
FROM beauty b
inner join boys bo ON b.boyfriend_id = bo.id
WHERE bo.boyname = '黄晓明';



