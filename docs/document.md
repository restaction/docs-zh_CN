# API文档阅读指南

这是写给API的使用者的一份指南，主要介绍如何阅读Flask-Restaction框架生成的API文档，这里
不会有复杂的概念，十分钟之内就能掌握。


## 整体布局

侧边栏分为四部分:

1. Desc

    关于整个API的介绍，其中会包含全局共有的数据格式和错误状态等信息
    
2. Meta

    一份JSON格式的文本，包含了整个API的元数据
    
3. Roles

    显示所有角色和他对应的权限
    
4. Resources

    所有的接口信息，这是文档的主体内容
    

## 路由规则

所有接口分为许多的Resource，每个Resource下面有多个Action。

Resource
:   表示一类资源。

Action
:   对资源的操作，例如 get, post, delete, get_list, post_login。
    
一个接口通常用`Resource.Action`这种格式表示，例如: `user.post_login`。
    
将`Resource.Action`转换成URL表示，只需将`.`换成`/`，再把Action下划线`_`前面的部分去掉。
    
例如: `user.post_login`就转换成`user/login`。
    
如果API不是部署在网站根目录下，例如部署在`/api`下， 则再加上这个前缀，变成`/api/user/login`。

!!! note
    对于`GET,DELETE`请求，请求数据放在URL后面，`POST,PUT,PATCH`请求数据在请求体中，
    `Content-Type` 为 `application/json`。


## 数据格式

数据格式是用来描述请求参数和响应内容的JSON文本。

文档中的数据格式和实际数据的格式是完全一致的，`{}`表示对象，`[]`表示列表。

**共享的数据格式:**  
在Desc部分会有一块"Shared"内容，里面描述的就是全局共享的数据格式。    
在每个Resource下面可能也会有一块"Shared"内容，里面描述的就是资源内共享的数据格式。   


## 接口描述

### 以登录接口为例

在User这个资源中"Shared"部分有这些内容，@user描述的是资源内共享的一个数据格式。

@user
```
{
    "date_create?datetime&optional": "创建时间",
    "date_modify?datetime&optional": "修改时间",
    "id?str": "用户ID",
    "lastlogin_date?datetime&optional": "最近登录时间",
    "role?str": "角色"
}
```

在登录接口中，Input和Output描述请求参数和响应内容，Error描述可能返回的错误。

Input
```
{
    "account?str": "用户ID或邮箱",
    "password?str": "密码"
}
```
请求参数是account和password，都是字符串类型。

Output
```
@user
```
响应内容就是"Shared"部分@user中的数据格式。

Error
```
{
    "403.UserNotFound": "帐号不存在",
    "403.WrongPassword": "密码错误"
}
```
如果帐号不存在，会返回下面这样的错误信息，其他错误返回的格式也一样:
```
{
    "status": 403
    "error": "UserNotFound"
    "message": "帐号不存在"
}
```

### 混合数据格式

以获取作者文章列表接口为例。

@pagging
```
{
    "page?int&min=1&default=1": "第几页",
    "per_page?int&min=1&max=100&default=10": "每页数量"
}
```

Input
```
{
    "$self@pagging": "分页",
    "author?str": "作者",
    "tag?str&optional": "标签"
}
```

这里`$self@pagging`表示将@pagging混入这个结构中，Input等价于:
```
{
    "page?int&min=1&default=1": "第几页",
    "per_page?int&min=1&max=100&default=10": "每页数量",
    "author?str": "作者",
    "tag?str&optional": "标签"
}
```


## 字段描述

字段描述的格式为 `name?validator&param=value&...`

name: 字段名称     
validator: 校验器，可理解为数据类型，常用的有str,int,datetime等等    
param: 参数，常用的有optional表示这个字段可选和default描述字段默认值    

例如:
```
"lastlogin_date?datetime&optional": "最近登录时间"
```
表示: 字段名称lastlogin_date，类型datetime，可选。
