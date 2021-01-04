# MeterSphere整合OPS方案

## OPS与MS对应关系

-   OPS---user.name		--------> 	MS---user.id
-   OPS---project.name   ------->      MS---workspace.name。工作空间管理。增删改查、及人员与工作空间关联。

-   OPS---权限管理   -------> MS---role.test_manager。权限管理（测试经理：可对接口测试、性能测试进行所有操作。）

## 方案一 Cookies + API Keys

- 分为两类：涉及到用户新增、工作空间新增、修改工作空间时使用admin的帐号用API Keys进行OPS与MS交互。其余操作使用当前登入的用户id进行操作，比如新增项目、查看接口列表、执行测试、查看报告、添加用例等。
- 用户数据迁移：第一次要批量将OPS目前现在存在用户调用MS接口循环创建一次，否则可能会出现用户关联工作空间时找不到用户的情况。可以使用postman循环创建所有用户。
- 接口返回值统一说明：{"success": true},  true表示成功,false失败。
### 交互流程
1. OPS用户登入后使用OPS的user.name + 统一密码Autotest#123调用MS登入接口“/signin”判断用户是否存在,如果不存在则使用Admin的API Keys 调用新增用户接口“/user/special/add”，创建一个“只读用户、默认工作空间”的用户。

- 用户_登入接口：POST {{URL}}/signin
```java
{
    "username":"dd.shan8",
    "password":"Autotest#123"
}
```

- 用户_新增-只读用户-默认工作空间： POST {{URL}}/user/special/add
```java
{
    "id": "dd.shan8",
    "name": "dd.shan8",
    "email": "dd.shan8@.com",
    "phone": "15606690056",
    "password": "Autotest#123",
    "roles": [
        {
            "id": "test_viewer",
            "ids": [
                "8c8f13c0-3852-11eb-a44c-005056b431aa"
            ]
        }
    ],
    "orgList": [
        {
            "id": "8c8ecd5b-3852-11eb-a44c-005056b431aa",
            "name": "默认组织",
            "description": "系统默认创建的组织"
        }
    ],
    "wsList": [
        {
            "id": "8c8f13c0-3852-11eb-a44c-005056b431aa",
            "organizationId": "8c8ecd5b-3852-11eb-a44c-005056b431aa",
            "name": "默认工作空间",
            "description": "系统默认创建的工作空间",
            "createTime": 1607321276000,
            "updateTime": 1607321276000
        }
    ]
}
```
- 返回值："success": true 或 false

2.  OPS用户创建项目时使用Admin的API Keys 调用MS接口“/workspace/special/add” 主动创建MS的工作空间，并将工作空间Id保存到OPS库中的third_ms表中。

    -   工作空间_添加：POST {{URL}}/workspace/special/add

        ```java
        {
            "organizationId": "8c8ecd5b-3852-11eb-a44c-005056b431aa",
            "name": "OPS工作空间-名称不能重复1",
            "description": "工作空间描述"
        }
        ```

    -   表结构（待定）

        -   | ops_project_name | ms_user_id     | ms_workspace_name | ms_workspace_id    |
            | ---------------- | -------------- | ----------------- | ------------------ |
            | OA项目           | dd.shan        | OA项目            | xxxxxxx            |
            | 项目名称唯一     | 用户id不可修改 | 工作空间名称唯一  | 工作空间id不可修改 |
            |                  |                |                   |                    |

        

3.   OPS用户在项目管理中关联用户到项目时调用MS Admin Keys 进行用户与工作空间的绑定。通过查询third_ms.ops_project_name查询到工作空间ID，进行工作空间用户的关联，角色默认。

    -   工作空间_添加成员到此工作空间：POST {{URL}}/user/special/ws/member/add

        ```java
        {
            "userIds": [
                "dd.shan8"
            ],
            "roleIds": [
                "test_manager"
            ],
            "workspaceId": "adff05ea-5f79-4123-9736-77fc4b61dc06"
        }
        ```

4.  OPS项目删除、项目名称修改时需要同步调用MS的Admin Keys 进行更新，并写更新third_ms表中的子段值。

    -   工作空间_删除： GET  {{URL}}/workspace/special/delete/:workspaceId

    -   工作空间_更新：POST {{URL}}/workspace/special/update

        ```java
        {
            "id": "e4bdb1a9-2282-4f52-848f-266b611ef7ab",
            "name": "API-空间1234",
            "organizationId": "8c8ecd5b-3852-11eb-a44c-005056b431aa"
        }
        ```

        

5.  OPS点击接口测试后使用OPS中user.name+默认密码进行登入MS,拿到MS返回的Cookies值，保存下来。然后通过third.ms表查询出此项目的workspaceId，在调用切换工作空间接口“{{URL}}/user/switch/source/ws/{workspaceId}“之后开始加载接口测试页面。

    -   工作空间切换： GET {{URL}}/user/switch/source/ws/:workspaceId


## 疑问
1. MS的登入如果开启配置文件中的run.mode=local则不需要做密码验证。暂不清楚是否有其他影响。
2. MS登入成功之后与系统的交互是保存的在cookie中的，那OPS如何拿到cookie与MS进行API的交互呢？
3. 