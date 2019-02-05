
# 1. 视图
视图：VIEW，虚表，保存有实表的查询结果。

创建方法：
```
CREATE VIEW view_name [(column_list)]
    AS select_statement
    [WITH [CASCADED | LOCAL] CHECK OPTION]
```

查看视图定义：
```
SHOW CREATE VIEW view_name
```

查看视图：
```
show table v_students\G

comm:view
```

删除视图：
```
DROP VIEW [IF EXISTS]
    view_name [, view_name] ...
    [RESTRICT | CASCADE]
```

视图中的数据事实上存储于“基表”中，因此，其修改操作也会针对基表实现；其修改操作受基表限制。
