# SQLAlchemy 






## 参考文档
1. [Flask数据的增删改查(CRUD)](https://blog.csdn.net/weixin_42670402/article/details/91046647)
2. [Flask 入门 5 – Flask SQLAlchemy 与 MySQL 简介](https://www.hizxc.com/1456.html#zhi_xing_yuan_sheng_SQL_yu_ju)
2. [如何执行原生SQL语句并遍历返回结果](
https://stackoverflow.com/questions/17972020/how-to-execute-raw-sql-in-flask-sqlalchemy-app]
```
# 原生sql查询id = 1的信息，结果是一个可以遍历的对象
array = []
sql = 'select * from students where student_id=1 or student_id=3;'
base_query = db.session.execute(sql)
for r in base_query:
    # print(r[0])  # Access by positional index
    # print(r['name'])  # Access by column name as a string
    r_dict = dict(r.items())  # convert to dict keyed by column names
    array.append(r_dict)

return jsonify({'data': array})
```