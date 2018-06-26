---
layout:     post                    # 使用的布局（不需要改）
title:      SQL中实现关系代数中的除运算浅析         # 标题
subtitle:                          #副标题
date:       2018-5-4 18:00              # 时间
author:     Ming.J                      # 作者
header-img: img/blog-header1.jpg    #这篇文章标题背景图片
catalog: true                       # 是否归档
tags:                               #标签
    - 数据库
    - 学习
    - 笔记
---

> ## 准备工作

- 先给出构造测试表的初始化代码，有兴趣的小伙伴可以跑一跑试试
  ```sql
  drop table R;
  drop table S;
  create table R (
    X integer,
    Y varchar(5)
  );

  create table S (
    Y varchar(5),
    Z integer
  );

  insert into R (X, Y) values (
    1, 'A'
  );
  insert into R (X, Y) values (
    2, 'B'
  );
  insert into R (X, Y) values (
    3, 'C'
  );
  insert into R (X, Y) values (
    1, 'B'
  );
  insert into R (X, Y) values (
    1, 'C'
  );
  insert into R (X, Y) values (
    2, 'A'
  );

  insert into S (Y, Z) values (
    'A', 3
  );
  insert into S (Y, Z) values (
    'B', 3
  );
  insert into S (Y, Z) values (
    'C', 3
  );
  ```
- 现在整体的表结构如下
  - R
    ![R表详情](http://qjm253.cn/img/database/database_double_exists01.png)
  - S
    ![S表详情](http://qjm253.cn/img/database/database_double_exists02.png)
- 解释
  - 其中X,Y,Z都可以代表一个或多个字段
  - Y为两张表中的相同字段

> ## 关系代数除运算（division）扫盲

- R÷S
  - 对于某个R关系中的X的某个具体值x映射到Y的集合，如果它包含关系S中Y的集合，那么这个x就会出现在结果集当中
  - 像上面构造的两张表R和S，R÷S的结果就是'A'了

OK，所以说了和没说差不多，我还是不懂什么是关系代数中的除运算_(:з」∠)_
...那下面就来举一个形象一点的栗子ლ(╹◡╹ლ)
- 这是栗子
  - R表示学生的选课信息，其中X表示学号，Y表示课程号
  - S表示课程的信息，其中Y表示课程号，Z表示学分
  - 好现在，上面的两种表就变成下面这样了
    - R
      ![选课信息](http://qjm253.cn/img/database/database_double_exists04.png)
    - S
      ![课程信息](http://qjm253.cn/img/database/database_double_exists03.png)
  - ok，现在我们要找出一部分学生的学号，他们选了所有的课
  - R÷S ==> 选了所有课的学生（很显然，上面只有学号为1的学生选了所有的课）
  - 结果如下：
    ![选了所有课的学生](http://qjm253.cn/img/database/database_double_exists05.png)

> ## SQL实现
> 网上常见的方式就是用双重not exists来实现，咋一看不是很好理解，笔者这里对其进行稍详尽点的分步分析

- ### 最终代码
  ```sql
  SELECT DISTINCT R1.X
  FROM R R1
  WHERE NOT EXISTS(
    SELECT *
    FROM S
    WHERE NOT EXISTS(
      SELECT *
      FROM R R2
      WHERE R2.X = R1.X AND R2.Y = S.Y
    )
  );
  ```
- ### 分析
  - 我们先来对里面一层的not exists进行分析
    - 执行下面代码：
      ```sql
      SELECT *
      FROM S
      WHERE NOT EXISTS(
        SELECT *
        FROM R R2
        WHERE R2.X = 1 AND R2.Y = S.Y
      );

      SELECT *
      FROM S
      WHERE NOT EXISTS(
        SELECT *
        FROM R R2
        WHERE R2.X = 2 AND R2.Y = S.Y
      );
      ```
    - 第一个执行结果为空，第二个执行结果如下
      ![学号为2的同学没选的课](http://qjm253.cn/img/database/database_double_exists06.png)
    - 分析一下可以知道，上面的SQL完成的任务是“**对于某个学号的学生，求得该学生未选的课程列表**”
  - OK，我们加上外层的not exists
    ```sql
    SELECT DISTINCT R1.X
    FROM R R1
    WHERE NOT EXISTS(
      SELECT *
      FROM S
      WHERE NOT EXISTS(
        SELECT *
        FROM R R2
        WHERE R2.X = R1.X AND R2.Y = S.Y
      )
    );
    ```
    - 从上面的分析我们知道，内层的SQL求的是对于一个具体的学生，其未选课程的列表
    - 那么外层加上一个not exists，整个SQL的含义就是求没有未选课程的学生，换句话说，就是求选了所有课的学生
    - 当然，第一行的DISTINCT也是很有必要的，去除了重复的X，你可以试试，不加，看一下结果，会发现每一个结果都会出现S表中Y集合的大小那么多次。
