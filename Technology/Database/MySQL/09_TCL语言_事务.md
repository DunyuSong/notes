# TCL Transaction Control Language
-- 事务：一个或一组SQL语句组成一个执行单元，这个执行单元要么全部执行，要么全部不执行。
show engines;
## 事务的创建
-- 隐式事务：没有开始标记，也没有结束标记。比如insert, delete, update语句
-- 显式事务：有开始标记，有结束标记。前提：必须先设置自动提交功能为禁用(set autocommit=0; 当次回话有效)，默认是开启自动提交的。
SHOW variables LIKE 'autocommit';
## 事务的四个关键属性（ACID）
### 原子性：要么都执行，要么都回滚
### 一致性：保证数据的状态操作前和操作后保持一致
### 隔离性：多个事务同时操作相同数据库的同一个数据时，一个事务的执行不受另外一个事务的干扰
### 持久性：一个事务一旦提交，则数据将持久化到本地，除非其他事务对其进行修改
相关步骤：

	1、开启事务
	2、编写事务的一组逻辑操作单元（多条sql语句）
	3、提交事务或回滚事务

## 事务的使用
### 设置事务为commit，提交生效
-- step1：开启事务
SET autocommit = 0;
START TRANSACTION;
-- step2:编写一组SQL语句
UPDATE beauty SET phone=500 where name='柳岩';
UPDATE beauty SET phone=1500 where name='苍老师';
-- step3:结束事务
COMMIT;
SELECT * from beauty;
### 设置事务为 ROLLBACK ,也就是运行到 rollback 时 start 之后做的事都是无效的，回滚到start 之前的数据。
-- step1：开启事务
SET autocommit = 0;
START TRANSACTION;
-- step2:编写一组SQL语句
UPDATE beauty SET phone=100 where name='柳岩';
UPDATE beauty SET phone=500 where name='苍老师';
-- step3:结束事务
ROLLBACK;

SELECT * from beauty;

### 设置事务为 ROLLBACK ，但是执行第二条SQL语句出错，这里模拟SQL语句写错。那么第一条SQL还是会正常执行，第二条失败了相当于语句终止了，没有进行 rollback
-- step1：开启事务
SET autocommit = 0;
START TRANSACTION;
-- step2:编写一组SQL语句
UPDATE beauty SET phone=100 where name='柳岩';	-- 这条语句生效提交了
UPDATE beauty SET sex=500 where name='苍老师';	-- 这条语句失败了，没有进行回滚，因为后面的相当于没有执行
-- step3:结束事务
ROLLBACK;

SELECT * from beauty;

## 事务的4种隔离级别
-- 查看当前隔离级别
SELECT @@tx_isolation;
SET TRANSACTION ISOLATION LEVEL repeatable read;

### read uncommitted(读未提交/脏读)：
- 读取的数据可能是另一个事务已经修改但是没有提交。
- 比如：开启事务1正在update一个字段，此时未提交; 事务2此时读取的数据那么就是update之后的数据。如果事务1此时做了提交，
	事务2重新读取一次就会发现读取的数据是提交之后的数据，也就是说事务2每次读取的都是事务1修改过的数据，不管事务1是否提交。
### read committed(读已提交/不可重复读)：
- 读取的数据每次都是另一个事务1已经提交之后的数据，未提交的数据不会被读取。会存在事务2还未结束的时候读取了事务1已经提交的数据。正常情况下我们是希望在一个事务中每次读取的数据应该是相同的，不依赖其他事务的修改或者提交。
- 比如：开启事务1正在update一个字段，此时未提交；事务2此时读取的数据那么就是update之前的数据。如果事务1此时做了提交，事务2重新读取一次就会发现读取的数据是提交之后的数据，
				也就是说事务2每次读取的都是事务1已经提交的数据，不管事务1提交之前所做的所有修改。会造成事务2在事务期间读取的数据不一致。
### repeatable read(可重复读/幻读)：
- 解决前面两种问题，如果设置为这个隔离级别那么在同一个事务中，每次读取的数据都是相同过的，不管另一个事务是否提交，只有当我当前事务结束之后新开启一个新的事务那么才会读取其他事务提交的数据。
- 比如：开启事务1正在update一个字段，此时未提交; 事务2此时读取的数据那么就是update之前的数据。如果事务1此时做了提交，事务2重新读取一次就会发现读取的数据还是update之前的，只有等待事务2结束之后重新再读取才会读取新的数据。
### serializable(串行化)：
- 幻读一般出现在事务1准备做update的时候，此时事务2正在进行insert。开启事务1查看表 temp，开启事务2向temp表中insert 数据，
		此时会发现命令行insert语句会一直卡主，变成等待状态，只有等事务1执行完commit结束事务1时，事务2的insert语句才会自动继续执行。
		类似于Java中的锁，事务1将temp表中的数据锁定了，只有等事务1释放掉锁，事务2才能进行数据的操作。这个隔离级别会阻止表的insert、update、delete操作，防止并发问题。
- 比如：开启事务1正准备update一个字段修改3行数据，此时update语句还没执行的时候事务2正在insert一行数据，那么当insert完之后此时事务1回车执行update发现修改了4行数据。
- 总结：MySQL默认的隔离级别为read-repeatable，因为serializable隔离级别性能会比较低，所以一般不建议用。




