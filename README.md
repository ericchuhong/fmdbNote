# fmdbNote
fmdb的学习摘要

### FMDB 学习摘要
-----
使用如下代码加入到Cocoapods上
    
    pod 'FMDB'

###使用

- FMDatabase - 表示单独的一个数据库，可以用来执行SQL语句的实例对象
- FMResultSet - 表示执行完SQL语句的查询之后存放结果数据的集合
- FMdatabaseQueue - 可以支持数据库的多线程操作，是线程安全操作类型的


### 创建数据库
每一个数据库都是根据一个系统路径创建，可以使用以下三种方式之一：

1. 一个系统硬盘路径，如果没有则会创建

1.  @“”。则fmdb会在临时文件目录下创建这个数据库如果断开连接，数据库文件被删除。

1. NULL，则它会创建一个内存中的数据库，数据库开连接时，数据库文件被删除。



    FMDatabase *db = [FMDatabase databaseWithPath:@"/tmp/tmp.db"];


每次进行与数据库的交互时，都需提前打开数据库，文件不存在或者访问权限不足是则会失败：

打开数据库：

    if (![db open]) {
    [db release];
    return;
    }


关闭数据库：


    [db close];

## 执行更新
除了 **SELECT**之外的数据数据定义语言几乎都是更新语句：**CREATE, UPDATE, INSERT, ALTER, COMMIT, BEGIN, DETACH, DELETE, DROP, END, EXPLAIN, VACUUM, and REPLACE statements (plus many more). ，** 
所以你的语句一旦不是**SELECT** ，它就是更新语句。

- **返回值**：为**BOOL**类型 ,YES为执行成功，NO则遇到错误
- **-lastErrorMessage**：如果错误，使用此命令可以查看详细的错误信息

## 执行查询
**SELECT** 语句是通过 -executeQuery ... 方法执行查询的。查询后天的结果会通过 FMResultSet 对象返回。如果失败则通过 -lastErrorMessage 和 -lastErrorCode方法来获取更多的错误信息。

    FMResultSet *s = [db executeQuery:@"SELECT * FROM myTable"];
    while ([s next]) {
    //retrieve values for each record
    }

需要不断通过 -[FMResulSet next] 来获取下一个返回的数据对象

    FMResultSet *s = [db executeQuery:@"SELECT COUNT(*) FROM myTable"];
    if ([s next]) {
    	int totalCount = [s intForColumnIndex:0];
    }
通过适当的格式来接收数据：

- intForColumn:
- longForColumn:
- longLongIntForColumn:
- boolForColumn:
- doubleForColumn:
- stringForColumn:
- dateForColumn:
- dataForColumn:
- dataNoCopyForColumn:
- UTF8StringForColumnName:
- objectForColumnName:
