
## 常用操作
### 导出
- 导出表结构：Management--Data Export--选择需要导出的数据，注意下拉选择“Dump Struture Only”，仅导出表结构。
- 导出结果记录集：Query--Export Results--保存类型选择“SQL INSERT”
- 

### 查询数据
- 查询多个值：SELECT * FROM tableName where id in('a','b');


### 快捷键
- ctrl+enter：        执行当前语句(光标位置处)： 
- ctrl+shift+enter：  执行全部或选中的语句(execute all or selection)





## 问题记录
### 连接时提示Bad handshake 
原因：https://bugs.mysql.com/bug.php?id=91828
解决：使用“Navicat for MySQL”或降低版本。

