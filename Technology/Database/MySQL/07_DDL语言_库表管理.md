# DDL 数据定义语言
/**
创建：create
修改：alter
删除：drop
*/
## 库管理
### 创建库
-- 语法 create DATABASE [if not exists] 库名
-- 创建 books 库,默认会存储在安装目录下的data文件夹下面
CREATE DATABASE IF not EXISTS books;	
### 修改库
-- 新版本已经不支持通过SQL修改了，因为修改之后会存在安全问题，数据丢失。 如果一定要修改只能安装目录下data/库名
-- 修改库的字符集
ALTER DATABASE books CHARACTER SET 'utf8';

-- 删除库
-- DROP database IF exists books;

## 表管理
### 表创建
/*
语法：create table 表名(
	列名 列的类型[(长度) 约束]
)
*/
use books;
CREATE TABLE if not exists book(
	id int , 						-- 书的编号
	bName varchar(20), 	-- 书的名称
	price double,
	authorId INT,
	publishDate datetime
);

DESC book;

create table if not EXISTS author(
	id INT,
	au_name varchar(20),
	nation VARCHAR(20)
);
desc author;

### 表修改
-- 语法： alter table 表名 add|drop|modify|change  COLUMN 列名[列类型 约束];
-- 修改列名、类型或约束、添加列、删除列、修改表名
ALTER TABLE book CHANGE COLUMN publishDate pubDate TIMESTAMP;
-- 修改类型或约束。 修改名称使用 change ， 修改类型使用 modify
ALTER TABLE book MODIFY COLUMN pubDate datetime;
-- 添加列
ALTER TABLE book ADD COLUMN address varchar(20);
-- 删除列
ALTER TABLE book DROP COLUMN address;
-- 修改表名
ALTER TABLE book RENAME TO book_shop;
-- 查看当前库的所有表
use books;
SHOW tables;	
-- 查看表结构
DESC book_shop;

### 表删除
DROP TABLE IF exists author;

### 表复制

-- 仅复制表全部结构
CREATE TABLE copy_author LIKE author;

-- 复制表结构和里面的全部数据
CREATE TABLE copy_author_and_data SELECT * FROM author;

-- 复制表结构和里面的部分数据
CREATE TABLE copy_author_and_data2 SELECT id,au_name FROM author where nation = 'cc';

-- 仅复制表部分结构
CREATE TABLE copy_author_and_data3 SELECT id,au_name FROM author where 1 = 2;		-- false,或者写where=0

