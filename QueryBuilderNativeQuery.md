# 原生查询

`Db`类支持原生`SQL`查询操作，主要包括下面两个方法：

> 原生查询仅支持Db类操作，不支持在模型中调用原生查询方法（包括`query`和`execute`方法）。

## `query`方法

`query`方法用于执行`SQL`查询操作，返回查询结果数据集（数组）。

使用示例：

```
Db::query("select * from think_user where status=:id", ['id' => 1]);
```

> 如果你当前采用了分布式数据库，并且设置了读写分离的话，`query`方法默认是在读服务器执行，而不管你的SQL语句是什么。

如果希望从主库读取，可以使用

```
Db::query("select * from think_user where status=:id", ['id' => 1], true);
```

## `execute`方法

`execute`用于更新和写入数据的sql操作，如果数据非法或者查询错误则返回`false` ，否则返回影响的记录数。

使用示例：

```
Db::execute("update think_user set name='thinkphp' where status=1");
```

> 如果你当前采用了分布式数据库，并且设置了读写分离的话，`execute`方法始终是在写服务器执行，而不管你的SQL语句是什么。

## 参数绑定

支持在原生查询的时候使用参数绑定，包括问号占位符或者命名占位符，例如：

```
Db::query("select * from think_user where id=? AND status=?", [8, 1]);
// 命名绑定
Db::execute("update think_user set name=:name where status=:status", ['name' => 'thinkphp', 'status' => 1]);
```

> 注意不支持对表名使用参数绑定