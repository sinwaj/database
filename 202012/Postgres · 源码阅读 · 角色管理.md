# 一、背景
 Postgres采用基于角色访问控制机制，角色和用户的区别是用户缺省有LOGIN权限。权限分系统权限和对象权限。本文主要介绍系统权限管理。

# 二、系统权限属性
系统权限涉及使用数据库的权限，包括：登录LOGIN、超级用户（SUPERUSER）、创建数据库（CREATEDB）、创建角色（CREATEROLE）、口令（PASSWORD）、继承（INHERIT）、连接限制（CONNECTION LIMIT）。

# 三、角色相关的系统表
## 1.pg_authid
![image 表1 pg_authid](https://github.com/sinwaj/database/blob/main/images/2020-02.png)
 
2.pg_auth_members
![image 表2 成员关系](https://github.com/sinwaj/database/blob/main/images/2020-03.png)

# 四、数据结构
## 1、CreateRoleStmt
typedef struct CreateRoleStmt

{

NodeTag	type;

RoleStmtType stmt_type;	//角色类型ROLE/USER/GROUP，枚举类型

char	  *role;	//角色名

List	  *options;	//角色属性列表DefElem结构体

} CreateRoleStmt;

## 2、DefElem

typedef struct DefElem

{

NodeTag	type;

char	  *defnamespace;	//节点对应的命名空间

char	  *defname;//节点对应的属性

Node	  *arg;	//表示值或类型名

DefElemAction defaction;	// SET/ADD/DROP 

int	location;	/* token location, or -1 if unknown */

} DefElem;

#五、创建角色流程
1.角色类型判断

2.提取options参数，并赋值给对应DefElem变量

3.DefElem变量类型转换

4.isSuper/replication/byPassrls/createrole权限判断

5.系统保留名判断

6.有效期格式转换

7.密码检查

8.构造new_record数据

9.密码处理，不能为空

10.构造tuple

11.升级的pg_authid_oid赋值

12.insert pg_authid表

13.AddRoleMems处理

# 六、源码位置
/backend/commands/user.c

