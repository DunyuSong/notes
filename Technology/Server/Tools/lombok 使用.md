# lombok

## 使用slf4j进行日志管理
- https://zhuanlan.zhihu.com/p/32475568
1. 添加Maven依赖
```xml
        <!-- 开始: lombok -->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>${lombok.version}</version>
        </dependency>
        <!-- 结束: lombok -->
        <!-- 开始: 日志 -->
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-api</artifactId>
            <version>${slf4j-api.version}</version>
        </dependency>
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-log4j12</artifactId>
            <version>${slf4j-log4j12.version}</version>
        </dependency>
        <dependency>
            <groupId>log4j</groupId>
            <artifactId>log4j</artifactId>
            <version>${log4j.version}</version>
        </dependency>
        <!-- 结束: 日志 -->
```
2. 在resource目录下天机“log4j.properties”
3. 使用：在类上添加@Slf4j注解。
4. 代码中直接使用log.info即可。（idea需要添加插件）
5. 