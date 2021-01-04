## 配置
1. 新增环境变量：MAVEN_HOME=D:\apache-maven-3.6.0
2. 编辑环境变量path:%MAVEN_HOME%\bin
3. 验证：cmd==>mvn -v

## 仓库
### 本地仓库
1. 电脑安装之后默认会有仓库配置的配置。
2. 修改本地仓库位置：Maven目录\conf\settings.xml==><localRepository>D:\Maven</localRepository>
### 远程仓库
1. 私服：搭建在局域网环境中，为局域网服务。
2. 中央仓库：搭建在Internet上，全世界服务。
3. 中央仓库镜像：分担中央仓库的流量，提升用户访问速度。

## 依赖
### 依赖范围
范围    |   对主程序是否有效  | 对测试程序是否有效  |   是否参与打包    |   是否参与部署    |   例子
---|---|---|---|---|---
compile |       有效          |     有效            |       参与        |       参与        |   spring-core
test    |       无效          |     有效            |       不参与      |       不参与      |   junit
provided|       有效          |     有效            |       不参与      |       不参与      |   servlet-api.jar(Tomcat自带就有)
### 依赖传递性
1. B项目依赖A项目，那么A项目中添加依赖时B项目也会自动添加。
## 继承
- 解决问题：多个模块统一管理依赖、调用。比如：<scope>test</scope>返回的依赖不能传递，所以必然会分散到各个模块工程中，很容易造成版本不一致。
- 解决思路：将依赖统一提取到“父”工程中，在子工程中声明依赖时不指定版本，则默认子工程使用的版本为父项目版本。
## 操作步骤
1. 创建一个Maven工程作为父工程。File==>new project==>选择Maven==>这里不需要勾选模板(Create from archetype)，因为父项目中仅仅用于管理子工程==>默认下一步即可。
```xml
    <!--    这里打包方式需要改为pom -->
    <packaging>pom</packaging>
```
2. 创建一个Maven工程作为子工程。项目上右键==>New Module==>选择Maven==>根据自己项目需求选择archetype。这里最好手动指定一下父工程pom文件路径
```xml
    <!--    引入父工程坐标 -->
    <parent>
        <artifactId>autotest</artifactId>
        <groupId>com.github.shandongdong</groupId>
        <version>1.0-SNAPSHOT</version>
        <!--  以当前文件为基准的父工程pom.xml文件的相对路径      -->
        <relativePath>../pom.xml</relativePath>
    </parent>
```
3. 父工程统一管理子工程依赖版本。
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <!--    这个作为父项目统一管理 -->
    <groupId>com.github.shandongdong</groupId>
    <artifactId>autotest</artifactId>
    <version>1.0-SNAPSHOT</version>
    <!--    包含的子模块-->
    <modules>
        <module>notification</module>
    </modules>
    <modules>
        <module>子模块2</module>
    </modules>
    <!--    这里打包方式需要改为pom -->
    <packaging>pom</packaging>
    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <testng.version>6.14.3</testng.version>
    </properties>
    <!-- 统一管理依赖版本问题 -->
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.testng</groupId>
                <artifactId>testng</artifactId>
                <version>${testng.version}</version>
                <scope>test</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
</project>
```
4. 子工程添加依赖时无需填写版本号，会自动使用父项目的版本号。
- 可以通过Maven插件窗口看到Dependencies中的版本号,父项目修改版本，子项目自动也会修改版本
```xml
    <!--    引入父工程坐标 -->
    <parent>
        <artifactId>autotest</artifactId>
        <groupId>com.github.shandongdong</groupId>
        <version>1.0-SNAPSHOT</version>
        <!--  以当前文件为基准的父工程pom.xml文件的相对路径      -->
        <relativePath>../pom.xml</relativePath>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>notification</artifactId>
    <packaging>war</packaging>
    <description>通知中心：负责分发消息，包括邮件、短信、IM等</description>

    <name>notification Maven Webapp</name>
    <!-- FIXME change it to the project's website -->
    <url>http://www.example.com</url>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <maven.compiler.source>1.7</maven.compiler.source>
        <maven.compiler.target>1.7</maven.compiler.target>
    </properties>
    <dependencies>
        <dependency>
            <groupId>org.testng</groupId>
            <artifactId>testng</artifactId>
            <!--            无需写版本自动使用父项目的版本号，可以通过Maven插件窗口看到Dependencies中的版本号-->
            <scope>test</scope>
        </dependency>
    </dependencies>

```
5. 编写测试类进行测试.
- 需要注意的时候idea有的时候更新没有那么及时，会一直显示依赖导入错误，此时关闭所有打开的idea，然后重新打开查看。如果还是提示错误，可以运行maven test 命令验证。
```
public class Test01 {

    @Test
    public void test() {
        System.out.println("测试自动导入依赖包版本");
    }
}

```
6.注意：配置继承之后执行安装命令，需要先执行父工程。

## 聚合
- 类型：<packeting>pom</packeting>
- 作用：一键安装各个模块工程。
- 配置方式：在一个“总的聚合工程”中配置各个参数与聚合模块。 （可以在父工程，也可以另外建一个工程来做这个事情）
```
<!--    引入父工程坐标 -->
    <parent>
        <artifactId>autotest</artifactId>
        <groupId>com.github.shandongdong</groupId>
        <version>1.0-SNAPSHOT</version>
        <relativePath>../pom.xml</relativePath>
    </parent>
    <!-- 配置依赖管理。比如Controll打成war包，service、mapper、pojo等打成jar包并且controller依赖service，service里面配置依赖mapper，这样最终组合打包一个controller时就会将依赖一并打包，最终包含了所有的模块 -->
    <dependencies>
        <dependency>
            <groupId>com.github.shandongdong</groupId>
            <artifactId>service</artifactId>
        </dependency>
    </dependencies>
```
- 摘选例子
```
taotao-parent -- 管理依赖jar包的版本，全局，pom类型。通过dependencyManagement配置
|--taotao-common  --- 通用组件、工具类。打成jar包。
|--taotao-manage  --- 聚合工程，pom类型。所有模块继承自它
  |--com.taotao.manage.web       // controller模块，打成war包，依赖service
  |--com.taotao.manage.service   // service模块，打成jar包，依赖web
  |--com.taotao.manage.mapper    // mapper模块，打成jar包，依赖service
  |--com.taotao.manage.pojo      // pojo模块，打成jar包，无需依赖其它
```

- 使用方式：在聚合工程的pom.xml文件 maven install。

## 自动化部署
- 手动方式：一般是运行 maven package 打出war包，然后放到tomcat的webapp下，启动tomcat就可以自动解压，访问web项目了。
- 自动化方式：使用cargo插件，作为了解吧。实际不会这么用。可以参考：https://blog.csdn.net/xiaojin21cen/article/details/78570030 