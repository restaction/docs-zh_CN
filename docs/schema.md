# YAML与Schema语法

JSON的语法是YAML语法的子集，因此大部分的JSON文件都可以被YAML的解析器解析。
由于YAML的运作主要依赖缩进来决定结构，且字符串不需要双引号，写出的Schema会更加精简，更适合写在文档字符串中。

[Validr](https://github.com/guyskk/validr)中使用JSON格式表示Schema，
框架中则用的是YAML，是为了适合不同的使用环境。

学习Schema语法需要先学会基本的JSON和YAML语法，YAML语法比JSON语法略复杂一些，可以通过 [YAML 语言教程](http://www.ruanyifeng.com/blog/2016/07/yaml.html)对YAML语法进行初步了解。

之后可以看一下Validr中的[Schema语法](https://github.com/guyskk/validr/blob/master/Isomorph-JSON-Schema-zh-cn.md)，
Schema语法是基于JSON的，并且与实际数据结构相同，很容易掌握。

然后再转换到YAML格式的Schema，新手学习过程中可能会遇到一些问题，
这里总结了一些与Schema相关的语法，方便上手。


## YAML解析器

YAML中有一些特殊符号，`@` 是YAML的保留符号，`&` 用于表示 [锚](http://pyyaml.org/wiki/PyYAMLDocumentation#Aliases)，
这两个与Schema语法有冲突，框架对YAML解析器进行了修改，将 `@`，`&` 视为普通字符串，同时不再支持YAML的锚语法。


## 列表

注意: `'-'` 后面要有一个空格或换行。

简单的列表:

    # tags
    - &unique&minlen=1
    - str

嵌套的列表:

    # time_table
    # 星期一
    - - 上午写BUG
      - 下午改BUG
      - 晚上又写BUG
    # 星期二
    - - 上午改昨天的BUG
      - 下午又写一堆BUG
      - 晚上改不完的BUG
    # ...

    # schema_of_time_table
    - &minlen=7&maxlen=7 # 一周七天
    - - &minlen=3&maxlen=3 # 每天有三个时间段
      - str&optional # 这个时间段的安排


## 字典

注意: `':'` 后面要有一个空格或换行。

简单的字典:

    user:
        id?int: user id
        name?str: user name

嵌套的字典:

    friends:
        best:
            $self: best friend
            id?int: user id
            name?str: user name
        bad:
            $self: bad friend
            id?int: user id
            name?str: user name

    friends:
        best@user: best friend
        bad@user: bad friend


## 复杂的嵌套

列表里面是字典:

    - my friends
    - id?int: user id
      name?str: user name

    - my friends
    - @user

字典里面有列表:

    friends:
        - my friends
        - id?int: user id
          name?str: user name

    friends:
        - my friends
        - @user


## 引用

    $shared:
        userid: int
        tags:
            - &unique&minlen=1
            - str
        user:
            id?int: user id
            name?str: user name

下面的可以引用上面的:

    $shared:
        userid: int
        user:
            id@userid: user id
            name?str: user name

## 混合

    $shared:
        paging:
            page_num?int&min=1&default=1: 第几页
            page_size?int&min=1&default=10: 每页的数量
        query:
            $self@paging: 查询参数
            tag?str: 标签
            date?date: 日期
