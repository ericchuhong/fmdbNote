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

上面的这些方法每一个都有与之相对应的根据索引获取数据的方法 ：

    {type}ForColumnIndex:

上面这些方法则是根据列的名字：


    {type}ForColumn:(name)


    FMResultSet *resultSet = [self.db execute Query:@“select * from t_student;”];
    
     //根据条件查询
    FMResultSet *resultSet = [self.db executeQuery:@“select * from t_student where id<?;”@(14)];
    
     //遍历结果集合   
    
    while ([resultSet  next])
    
       {
    int idNum = [resultSet intForColumn:@“id”]；
    
    NSString *name = [resultSet
    objectForColumn:@“name”];
    
    int age = [resultSet intForColumn:@“age”];
      }

## 事务

FMDatabase可以通过调用开始/结束事务的方法来实现批量语句操作。

## 多语句和批量操作
使用 FMDatabase的 **executeStatements:withResultBlock:**

    NSString *sql = @"create table bulktest1 (id integer primary key autoincrement, x text);"
     "create table bulktest2 (id integer primary key autoincrement, y text);"
     "create table bulktest3 (id integer primary key autoincrement, z text);"
     "insert into bulktest1 (x) values ('XXX');"
     "insert into bulktest2 (y) values ('YYY');"
     "insert into bulktest3 (z) values ('ZZZ');";
    
    success = [db executeStatements:sql];
    
    sql = @"select count(*) as count from bulktest1;"
       "select count(*) as count from bulktest2;"
       "select count(*) as count from bulktest3;";
    
    success = [self.db executeStatements:sql withResultBlock:^int(NSDictionary *dictionary) {
    NSInteger count = [dictionary[@"count"] integerValue];
    XCTAssertEqual(count, 1, @"expected one record for dictionary %@", dictionary);
    return 0;
    }];

## 数据清洗
当时用SQL语句时，如果还没有插入数据时，不要去执行清洗数据。


    INSERT INTO myTable VALUES (?, ?, ?, ?)

？ 问号是一个占位符，会被后面的值或对象代替。这个方法可以接收许多不同类型的参数（NSArray, NSDictionary, or 或者va_list），这些参数都会被自动专业为需要的值 。

> NSInteger 之类的变量应该 转化为 NSNumber 对象，可以在前面 加上 “@” 或者使用方法  [NSNumber numberWithInt:identifier]  转换
> 相同的 插入NULL值得话也需要转化成 Objecttive - C对象 [NSNull null]  。可以使用 comment ?: [NSNull null] 语法，来插入一个字符串

另外，我们可以使用命名好的参数语法

    INSERT INTO authors (identifier, name, date, comment) VALUES (:identifier, :name, :date, :comment)

参数必须以 冒号“：”为前缀，所以字典的key 不要带有前缀

    NSDictionary *arguments = @{@"identifier": @(identifier), @"name": name, @"date": date, @"comment": comment ?: [NSNull null]};
    BOOL success = [db executeUpdate:@"INSERT INTO authors (identifier, name, date, comment) VALUES (:identifier, :name, :date, :comment)" withParameterDictionary:arguments];
    if (!success) {
    NSLog(@"error = %@", [db lastErrorMessage]);
    }

---

## 使用 FMDatabaseQueue 与 线程安全

如果在多线程里面为去创建一个FMDatabase 对象是很糟糕的选择，可能会造成你的内存爆炸或者让你的Mac 直接宕机。
> 所以请不要创建一个 FMDatabase 对象之后再多线程中使用它

我们可以使用FMDataseQueue 到多线程中。它会同步和协调多线程的访问。

###使用 

    FMDatabaseQueue *queue = [FMDatabaseQueue databaseQueueWithPath:aPath];
    [queue inDatabase:^(FMDatabase *db) {
    [db executeUpdate:@"INSERT INTO myTable VALUES (?)", @1];
    [db executeUpdate:@"INSERT INTO myTable VALUES (?)", @2];
    [db executeUpdate:@"INSERT INTO myTable VALUES (?)", @3];
    
    FMResultSet *rs = [db executeQuery:@"select * from foo"];
    while ([rs next]) {
    …
    }
    }];

也可以把代码包装起来，传到FMDatabaseQueue的事务处理块中

    [queue inTransaction:^(FMDatabase *db, BOOL *rollback) {
    [db executeUpdate:@"INSERT INTO myTable VALUES (?)", @1];
    [db executeUpdate:@"INSERT INTO myTable VALUES (?)", @2];
    [db executeUpdate:@"INSERT INTO myTable VALUES (?)", @3];
    
    if (whoopsSomethingWrongHappened) {
    *rollback = YES;
    return;
    }
    // etc…
    [db executeUpdate:@"INSERT INTO myTable VALUES (?)", @4];
    }];

## 自定义 基于 blocks的sqlite方法
> 参考 在 main.m 的方法 **-makeFunctionNamed:**

摘要就到这里结束了 

--2016年1月14日
