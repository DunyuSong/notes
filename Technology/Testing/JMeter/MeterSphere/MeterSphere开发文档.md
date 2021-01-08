## MeterSphere开发文档
## Envronment
### 官方环境安装部署
1. [访问地址](http://192.168.163.41:8081/login)
2. 登入帐号密码：admin:metersphere
2. 机器：192.168.163.41; root:autotest#123; 
3. 安装方式：一键安装，全docker。模式：allinone，主副一体。
4. 安装目录:/opt
4. 接口文档: http://localhost:8081/swagger-ui/index.html?configUrl=/v3/api-docs/swagger-config
#### 调式技巧
- 查看接口运行日志： docker logs -f ms-server --tail 200
- 日志路径： /opt/metersphere/logs/
- 修改环境配置：vim /opt/metersphere/.env
- 重新加载服务：msctl reload

### 本地开发环境
1.  本地开发环境CentOS7。 配置文件:/opt/metersphere/conf/metersphere.properties
2.  本地开发环境CentOS7。 安装JMeter5.3(拷贝里面所有文件到/opt/jmeter目录下，并且用代码工程中的src/main/resources/jmeter/bin下的配置文件覆盖下载的JMeter配置文件/opt/jmeter/bin
)
3.  服务器CentOS 7(192.168.163.41): 在 /usr/local 搭建了 JDK 1.8、MySQL 5.7、kafka_2.13-2.6.0、zookeeper-3.6.2、maven-3.6.3、docker、jenkins-2.263
4.  数据库192.168.163.43: 创建了数据库metersphere_dev
5.  在服务器上创建Topic
```shell
创建2个topic
bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic JMETER_METRICS
bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic JMETER_LOGS
查看创建的topic
bin/kafka-topics.sh --list --zookeeper localhost:2181
```
5. 修改kafka的配置文件 config/server.properties。 添加端口访问
```properties
# 允许外部端口连接                                            
listeners=PLAINTEXT://0.0.0.0:9092

# 外部代理地址                                                
advertised.listeners=PLAINTEXT://192.168.163.41:9092
```

## 代码分析
### 项目启动项分析
1. 第一次启动的时候会读取配置文件中的配置，然后自动创建所需要的表结构。
2. 第一次启动会创建默认用户admin，并修改表：organization(创建默认组织),test_resource、test_resource_pool、user(创建默认用户admin信息)、user_role(给admin绑定3个角色) 、workspace(创建默认工作空间)

### 登入代码分析
- 登入之前前端会先通过socker连接服务器，判断是否开启了ldap登入方式。src/main/java/io/metersphere/ldap/controller/LdapController.java
```java
    @GetMapping("/open")
    public boolean isOpen() {
        // step1:
        boolean result = ldapService.isOpen();
        System.out.println(result);
        return result;
    }
```
### ldap登入
略

### 普通新用户登入系统
1. src/main/java/io/metersphere/controller/LoginController.java
```java
    @PostMapping(value = "/signin")
    public ResultHolder login(@RequestBody LoginRequest request) {
         SecurityUtils.getSubject().getSession().setAttribute("authenticate", UserSource.LOCAL.name());
        return userService.login(request);
    }
```
2. src/main/java/io/metersphere/service/UserService.java
```java

                // 自动选中组织，工作空间
                if (StringUtils.isEmpty(user.getLastOrganizationId())) {
                    List<UserRole> userRoles = user.getUserRoles();
                    List<UserRole> test = userRoles.stream().filter(ur -> ur.getRoleId().startsWith("test")).collect(Collectors.toList());
                    List<UserRole> org = userRoles.stream().filter(ur -> ur.getRoleId().startsWith("org")).collect(Collectors.toList());
                    if (test.size() > 0) {
                        String wsId = test.get(0).getSourceId();
                        switchUserRole("workspace", wsId);
                    } else if (org.size() > 0) {
                        String orgId = org.get(0).getSourceId();
                        switchUserRole("organization", orgId);
                    }
                }
                // 返回 userDTO
                return ResultHolder.success(subject.getSession().getAttribute("user"));
            } else {
                return ResultHolder.error(Translator.get("login_fail"));
            }
        } catch (ExcessiveAttemptsException e) {
            msg = Translator.get("excessive_attempts");
        } catch (LockedAccountException e) {
            msg = Translator.get("user_locked");
        } catch (DisabledAccountException e) {
            msg = Translator.get("user_has_been_disabled");
        } catch (ExpiredCredentialsException e) {
            msg = Translator.get("user_expires");
        } catch (AuthenticationException e) {
            msg = e.getMessage();
        } catch (UnauthorizedException e) {
            msg = Translator.get("not_authorized") + e.getMessage();
        }
        MSException.throwException(msg);
        return null;
    }
```
3. 真正的登入相关校验
src/main/java/io/metersphere/security/ShiroDBRealm.java
```java

    /**
     * 登录认证
     */
    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken authenticationToken) throws AuthenticationException {
        UsernamePasswordToken token = (UsernamePasswordToken) authenticationToken;
        String login = (String) SecurityUtils.getSubject().getSession().getAttribute("authenticate");

        String userId = token.getUsername();
        String password = String.valueOf(token.getPassword());
		// 判断如果conf/metersphere.properties配置中开启了run.mode=local则无需校验密码
        if (StringUtils.equals("local", runMode)) {
            UserDTO user = getUserWithOutAuthenticate(userId);
            userId = user.getId();
            SessionUser sessionUser = SessionUser.fromUser(user);
            SessionUtils.putUser(sessionUser);
            return new SimpleAuthenticationInfo(userId, password, getName());
        }

        if (StringUtils.equals(login, UserSource.LOCAL.name())) {
            return loginLocalMode(userId, password);
        }

        if (StringUtils.equals(login, UserSource.LDAP.name())) {
            return loginLdapMode(userId, password);
        }

        UserDTO user = getUserWithOutAuthenticate(userId);
        userId = user.getId();
        SessionUser sessionUser = SessionUser.fromUser(user);
        SessionUtils.putUser(sessionUser);
        return new SimpleAuthenticationInfo(userId, password, getName());

    }
```
连接数据库，校验用户信息，是否存在
```java
    private UserDTO getUserWithOutAuthenticate(String userId) {
    	// 调用Mapper查询数据
        UserDTO user = userService.getUserDTO(userId);
        String msg;
        if (user == null) {
            user = userService.getUserDTOByEmail(userId);
            if (user == null) {
                msg = "The user does not exist: " + userId;
                logger.warn(msg);
                throw new UnknownAccountException(Translator.get("user_not_exist") + userId);
            }
        }
        return user;
    }
```
通过用户名查询用户
```java

    public UserDTO getUserDTO(String userId) {

        User user = userMapper.selectByPrimaryKey(userId);
        if (user == null) {
            return null;
        }
        if (StringUtils.equals(user.getStatus(), UserStatus.DISABLED)) {
            throw new DisabledAccountException();
        }
        UserDTO userDTO = new UserDTO();
        BeanUtils.copyProperties(user, userDTO);
        UserRoleDTO userRole = getUserRole(userId);
        userDTO.setUserRoles(Optional.ofNullable(userRole.getUserRoles()).orElse(new ArrayList<>()));
        userDTO.setRoles(Optional.ofNullable(userRole.getRoles()).orElse(new ArrayList<>()));
        return userDTO;
    }

```
### 接口测试模块
1. 直接访问会先判断是否登陆。src/main/java/io/metersphere/controller/LoginController.java
```java
    @GetMapping(value = "/isLogin")
    public ResultHolder isLogin() {
    	//OPS:根据OPS传递过来的消息自动进行一次登陆，如果是新用户则自动创建用户绑定默认角色，关联默认组织，根据OPS的ProjectId创建新的工作空间。
        if (SecurityUtils.getSubject().isAuthenticated()) {
            SessionUser user = SessionUtils.getUser();
            if (StringUtils.isBlank(user.getLanguage())) {
                user.setLanguage(LocaleContextHolder.getLocale().toString());
            }
            return ResultHolder.success(user);
        }
        String ssoMode = env.getProperty("sso.mode");
        if (ssoMode != null && StringUtils.equalsIgnoreCase(SsoMode.CAS.name(), ssoMode)) {
            return ResultHolder.error("sso");
        }
        return ResultHolder.error("");
    }
```
2. 接口测试首页：
	src/main/java/io/metersphere/api/controller/APITestController.java
	src/main/java/io/metersphere/api/controller/APIReportController.java
3.  首页入口代码
```java
    @GetMapping("dashboard/tests")
    public List<DashboardTestDTO> dashboardTests() {
        return apiReportService.dashboardTests(SessionUtils.getCurrentWorkspaceId());
    }
```
4. 访问接口测试调用以下接口
	1. http://192.168.163.41:8081/api/recent/5 所有测试列表
	2. http://192.168.163.41:8081/api/report/recent/5 测试报告列表
	3. http://192.168.163.41:8081/api/list/schedule    运行中的任务列表
	4. http://192.168.163.41:8081/api/report/dashboard/tests  测试日历



### 使用API Keys 进行接口之间的交互

1.  有效期30分钟。src/main/java/io/metersphere/security/ApiKeyHandler.java

```java
	// 签名验证
    public static String getUser(String accessKey, String signature) {
        if (StringUtils.isBlank(accessKey) || StringUtils.isBlank(signature)) {
            return null;
        }
        UserKey userKey = CommonBeanFactory.getBean(UserKeyService.class).getUserKey(accessKey);
        if (userKey == null) {
            throw new RuntimeException("invalid accessKey");
        }
        String signatureDecrypt;
        try {
            signatureDecrypt = CodingUtil.aesDecrypt(signature, userKey.getSecretKey(), accessKey);
        } catch (Throwable t) {
            throw new RuntimeException("invalid signature");
        }
        String[] signatureArray = StringUtils.split(StringUtils.trimToNull(signatureDecrypt), "|");
        if (signatureArray.length < 2) {
            throw new RuntimeException("invalid signature");
        }
        if (!StringUtils.equals(accessKey, signatureArray[0])) {
            throw new RuntimeException("invalid signature");
        }
        long signatureTime;
        try {
            signatureTime = Long.parseLong(signatureArray[signatureArray.length - 1]);
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
        if (Math.abs(System.currentTimeMillis() - signatureTime) > 1800000) {
            //签名30分钟超时
            throw new RuntimeException("expired signature");
        }
        return userKey.getUserId();
    }
```

2.  使用个人API Key 进行接口交互校验

    ```java
    import org.apache.commons.codec.binary.Base64;
    import javax.crypto.Cipher;
    import javax.crypto.spec.IvParameterSpec;
    import javax.crypto.spec.SecretKeySpec;
    
    String accessKey = "${accessKey}";
    String secretKey = "${secretKey}";
    
    public static String aesEncrypt(String src, String secretKey, String iv) throws Exception {
        byte[] raw = secretKey.getBytes("UTF-8");
        SecretKeySpec secretKeySpec = new SecretKeySpec(raw, "AES");
        Cipher cipher = Cipher.getInstance("AES/CBC/PKCS5Padding");
        IvParameterSpec iv1 = new IvParameterSpec(iv.getBytes());
        cipher.init(Cipher.ENCRYPT_MODE, secretKeySpec, iv1);
        byte[] encrypted = cipher.doFinal(src.getBytes("UTF-8"));
        return Base64.encodeBase64String(encrypted);
    }
    
    try {
        //调用加密算法生成签名
        String signature = aesEncrypt(accessKey + "|" + System.currentTimeMillis(), secretKey, accessKey);
        //将签名值存入 signature 变量中
        vars.put("signature",signature);
    } catch (Exception e) {
        e.printStackTrace();
    }
    ```

    


## FQA

### 为什么JMeter 函数没有被解析执行
- 本地使用spring运行起来环境之后，JMeter函数没有解析成功。（已解决，系统读取jmeter.home 失败导致）src/main/java/io/metersphere/api/jmeter/JMeterService.java 

### 如何修改服务的配置文件
- vim /opt/metersphere/.env

### 如何添加节点（非windows机器）
1. 方式1：使用官方一键部署，修改install.conf中MS_MODE=node-controller，如需要修改端口修改配置MS_NODE_CONTROLLER_PORT=8082，修改完之后在资源池添加这台机器的ip和端口即可。
2. 方式2：直接单独部署jar文件，指定配置文件路径即可，需要机器有docker环境。


### 如何在 Windows 上搭建开发环境

1. 源码下载，导入idea。
2. 修改 src\main\java\io\metersphere\Application.java 中加载的配置文件.@PropertySource(value = {"file:c:\\opt\\metersphere\\conf\\metersphere.properties"}, encoding = "UTF-8", ignoreResourceNotFound = true)
3. 修改 src\main\resources\application.properties 中日志配置路径logging.file.path=c:/opt/metersphere/logs/${spring.application.name}
4. 修改 src\main\resources\application.properties 中属性路径jmeter.home=c:/opt/jmeter
5. 修改 src\main\resources\logback.xml 中属性路径 <property file="c:/opt/metersphere/conf/metersphere.properties"/>
6. 下载JMeter 5.2.1版本，将JMeter所有文件拷贝到c:/opt/jmeter 目录下。MS会读取这个目录作为JMeter主目录，然后加载lib执行函数，运行测试等。
7. idea 配远程docker：https://cloud.tencent.com/developer/article/1494921   https://www.cnblogs.com/hei12138/p/ideausedocker.html  







