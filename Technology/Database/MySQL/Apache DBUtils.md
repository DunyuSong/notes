# 官网
- https://github.com/apache/commons-dbutils
- http://commons.apache.org/proper/commons-dbutils/

## 配置
- 环境：MySQL+idea+maven
- pom文件添加相关依赖（**必须用5.1.47版本**）
```xml
        <!-- 连接MySQL -->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.47</version>
        </dependency>
        <!-- https://mvnrepository.com/artifact/com.mchange/c3p0 -->
        <dependency>
            <groupId>com.mchange</groupId>
            <artifactId>c3p0</artifactId>
            <version>0.9.5.2</version>
        </dependency>
        <!-- 数据库操作 -->
        <dependency>
            <groupId>commons-dbutils</groupId>
            <artifactId>commons-dbutils</artifactId>
            <version>1.7</version>
        </dependency>
```
- 在项目resource目录添加"c3p0-config.xml"
```xml
<?xml version="1.0" encoding="UTF-8"?>
<c3p0-config>
    <!-- name为自定义数据的名称，支持配置多个，加载不同的数据库需要在代码里指定.
    private static ComboPooledDataSource ds = new ComboPooledDataSource("mysql1");
    -->
    <named-config name="mysql1">
        <property name="user">root</property>
        <property name="password">ppEnvDevYtt</property>
        <property name="driverClass">com.mysql.cj.jdbc.Driver</property>
        <property name="jdbcUrl">jdbc:mysql://10.122.156.45:3306/qa_temp?useUnicode=true&amp;characterEncoding=UTF-8&amp;useSSL=false</property>

        <!-- 连接池初始化时建立的连接数 默认值是3 -->
        <property name="initialPoolSize">3</property>
        <!-- 连接的最大空闲时间  单位秒 默认是0-代表永远不会断开连接  超过设定时间的空闲连接将会断开 -->
        <property name="maxIdleTime">30</property>
        <!-- 连接池中拥有的最大连接数 默认值为15个 -->
        <property name="maxPoolSize">20</property>
        <!-- 连接池中保持的最小连接数  默认值为3个-->
        <property name="minPoolSize">3</property>
        <!-- 将连接池的连接数保持在minpoolsize 必须小于maxIdleTime设置  默认值为0代表不处理  单位秒 -->
        <property name="maxIdleTimeExcessConnections">15</property>
    </named-config>
</c3p0-config>
```
- mysql 中创建表
```mysql
qa_temp database if exists qa_temp;
CREATE DATABASE IF NOT EXISTS qa_temp;
USE qa_temp;
 
drop table if exists user ;
CREATE TABLE IF NOT EXISTS `user ` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  ` name` varchar(50) DEFAULT NULL ,
  `pwd` varchar(50) DEFAULT NULL ,
  PRIMARY KEY (`id`)
) ;
#数据初始化
insert into user values (null ,'zhangsan' ,'123456' );
insert into user values (null ,'lisi' ,'123456' );
```
## 使用DBUtils实现增删改查
### 创建初始化工具类JDBCUtils.java
```java
package com.database.dbutils;

import com.mchange.v2.c3p0.ComboPooledDataSource;

import javax.sql.DataSource;
import java.sql.Connection;
import java.sql.SQLException;

/**
 * @Author: shandongdong
 * @Date: 2018/7/13 13:54
 * @Description:
 */
public class JDBCUtils {

    public static String connectDataSourceConfigName = "mysql1";

    //数据库连接池 使用C3P0数据库连接池 使用named-config模式 则通过有参构造函数初始化  带上name
    private static ComboPooledDataSource ds = new ComboPooledDataSource(connectDataSourceConfigName);

    /**
     * 获得数据库连接对象
     *
     * @return Connection
     * @throws SQLException
     */
    public static Connection getConnection() throws SQLException {
        return ds.getConnection();
    }

    /**
     * 获得c3p0连接池对象
     *
     * @return
     */
    public static DataSource getDataSource() {
        return ds;
    }
}
```
### Insert
使用queryRunner对象完成增删改操作：
```java
    //需求：向user表插入一条数据
    public void testInsert() {
        //第一步：创建queryRunner对象，用来操作sql语句
        QueryRunner qr = new QueryRunner(JDBCUtils.getDataSource());

        //第二步：创建sql语句
        String sql = "insert into user(name, password) values(?,?)";

        //第三步：执行sql语句,params:是sql语句的参数
        //注意，给sql语句设置参数的时候，按照user表中字段的顺序
        try {
            Object params[] = {"测试", "123456"};
            int update = qr.update(sql,params);
            System.out.println(update);
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
```
### Update
```java

    //需求：修改id==3的数据
    public void testUpdate() {

        //第一步：创建queryRunner对象，用来操作sql语句
        QueryRunner qr = new QueryRunner(JDBCUtils.getDataSource());

        //第二步：创建sql语句

        String sql = "update user set name = ? where id = ?";

        //第三步：执行sql语句,params:是sql语句的参数

        //注意，给sql语句设置参数的时候，按照user表中字段的顺序

        try {
            int update = qr.update(sql, "柳岩", 3);
            System.out.println(update);
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
```
### Delete
```java
    //需求：删除id==5的数据
    public void testDelete() {
        //第一步：创建queryRunner对象，用来操作sql语句
        QueryRunner qr = new QueryRunner(JDBCUtils.getDataSource());

        //第二步：创建sql语句
        String sql = "delete from user where id = ?";

        //第三步：执行sql语句,params:是sql语句的参数
        //注意，给sql语句设置参数的时候，按照user表中字段的顺序
        try {
            int update = qr.update(sql, 5);
            System.out.println(update);
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
```
### Query 查询数据
- 创建User.java，并生产get、set方法
```java
public class User {
    private Integer id;
    private String name;
    private String password;
}
```
- 创建MyHandler.java，用于自定义查询
```java
// ResultSetHandler<T>,<T>表示封装结果的类型
//MyHandler 是自定义的ResultSetHandler封装结果集策略对象
public class MyHandler implements ResultSetHandler<List<User>> {

    @Override
    public List<User> handle(ResultSet rs) throws SQLException {
        // 封装数据，数据从 Resultset 中获取
        List<User> list = new ArrayList<User>();
        while (rs.next()) {
            User user = new User();
            user.setId(rs.getInt("id"));
            user.setName(rs.getString("name"));
            user.setPassword(rs.getString("password"));
            list.add(user);
        }
        return list;
    }
}
```
- 获取表数据
```java

    //需求：获取user表中所有的数据
    public void testQuery() {
        //第一步：创建queryRunner对象，用来操作sql语句
        QueryRunner qr = new QueryRunner(JDBCUtils.getDataSource());
        //第二步：创建sql语句
        String sql = "select * from user";
        //第三步：执行sql语句,params:是sql语句的参数
        //注意，给sql语句设置参数的时候，按照user表中字段的顺序
        try {
            List<User> list = qr.query(sql, new MyHandler());
            System.out.println(list);
            for (User u : list) {
                String id = String.valueOf(u.getId());
                String name = u.getName();
                String password = u.getPassword();
                System.out.println("id:" + id + ", name:" + name + ", password:" + password);
            }
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
```
- 直接获取表中的数据
```java
    public void testQueryById() throws SQLException {
        QueryRunner queryRunner = new QueryRunner(JDBCUtils.getDataSource());
        String sql = "select * from user";

        Object object = queryRunner.query(sql, new MyResultSet());
        System.out.println("object:" + object);
    }

    @Test(description = "根据条件查找数据")
    public void testQueryById() throws SQLException {
        QueryRunner queryRunner = new QueryRunner(JDBCUtils.getDataSource());
        String sql = "select * from user where id = ?";
        // 查找id为2的数据，不设置则查找所有
        Object object = queryRunner.query(sql, new MyResultSet(), 2);
        System.out.println("object:" + object);
    }

    private class MyResultSet implements ResultSetHandler<Object> {
        @Override
        public Object handle(ResultSet resultSet) throws SQLException {
            while (resultSet.next()) {
                Integer id = resultSet.getInt(1);   //获取第1列单元格
                String name = resultSet.getString("name");  //获取列名称为name的单元格值
                System.out.println("id:" + id + ", name:" + name);
            }
            return null;
        }
    }
```
## ResultSetHandler实现类介绍（由DbUtils框架提供）
- DbUtils给我们提供了10个ResultSetHandler实现类，分别是：
1. ArrayHandler：     将查询结果的第一行数据，保存到Object数组中
2. ArrayListHandler     将查询的结果，每一行先封装到Object数组中，然后将数据存入List集合
3. BeanHandler     将查询结果的第一行数据，封装到user对象
4. BeanListHandler     将查询结果的每一行封装到user对象，然后再存入List集合
5. ColumnListHandler     将查询结果的指定列的数据封装到List集合中
6. MapHandler     将查询结果的第一行数据封装到map结合（key\==列名，value==列值）
7. MapListHandler     将查询结果的每一行封装到map集合（key\==列名，value==列值），再将map集合存入List集合
8. BeanMapHandler     将查询结果的每一行数据，封装到User对象，再存入mao集合中（key\==列名，value==列值）
9. KeyedHandler     将查询的结果的每一行数据，封装到map1（key==列名，value==列值 ），然后将map1集合（有多个）存入map2集合（只有一个）
10. ScalarHandler     封装类似count、avg、max、min、sum......函数的执行结果

### 使用内置Api
- BeanHandler:将查询结果的第一行数据，封装到user对象
```java
    //需求：测试BeanHandler策略
    @Test
    public void testBeanHandler() {
        //第一步：创建queryRunner对象，用来操作sql语句
        QueryRunner qr = new QueryRunner(JDBCUtils.getDataSource());
        //第二步：创建sql语句
        String sql = "select * from user";

        //第三步：执行sql语句,params:是sql语句的参数
        //注意，给sql语句设置参数的时候，按照user表中字段的顺序
        try {
            User user = qr.query(sql, new BeanHandler<User>(User.class));
            System.out.println("id:" + user.getId() + ", password:" + user.getPassword());
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }

```
- BeanListHandler:将查询结果的每一行封装到user对象，然后，再存入list集合
```java
    //需求：测试BeanListHandler策略
    @Test
    public void testBeanListHandler() {
        //第一步：创建queryRunner对象，用来操作sql语句
        QueryRunner qr = new QueryRunner(JDBCUtils.getDataSource());
        //第二步：创建sql语句
        String sql = "select * from user where id = ?";

        //第三步：执行sql语句,params:是sql语句的参数
        //注意，给sql语句设置参数的时候，按照user表中字段的顺序
        try {
            // 参数3为上面的根据指定where条件查询，不设置则查找所有记录集
            List<User> list = qr.query(sql, new BeanListHandler<User>(User.class), 3);
            System.out.println(list);
            for (User user : list) {
                System.out.println("id:" + user.getId() + ", name:" + user.getName() + ", password:" + user.getPassword());
            }
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
```

- ScalarHandler执行SQL中的函数
```java
    //ScalarHandler:封装类似count、avg、max、min、sum。。。。函数的执行结果
    @Test(description = "测试ScalarHandler策略")
    public void testScalarHandler() {
        //第一步：创建queryRunner对象，用来操作sql语句
        QueryRunner qr = new QueryRunner(JDBCUtils.getDataSource());
        //第二步：创建sql语句
        String sql = "select count(*) from user";
        //第三步：执行sql语句,params:是sql语句的参数
        //注意，给sql语句设置参数的时候，按照user表中字段的顺序
        try {
            Object object = qr.query(sql, new ScalarHandler());
            System.out.println(object);
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
```
## 常见问题
### BeanListHandler 处理数据库字段名xx_xx格式转换为java风格的xxXX格式
- 问题：使用BeanListHandler转换对象是出现getId为null的情况。
- 原因：通常，数据库的字段往往会出现2个单词以上的情况，比如TEMPLATE_ID这个字段名，以下划线作为分隔。对应的JavaBean的属性名，按照Java的命名规范（驼峰原则），则是templateId。这种情况下，BeanListHandler就无法做TEMPLATE_ID->templateId的映射了。
- 



# 参考资料
- https://www.cnblogs.com/CQY1183344265/p/5854418.html
- 