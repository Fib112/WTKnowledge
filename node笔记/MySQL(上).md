### 一、终端操作数据库

#### 1.1 连接数据库

如果想要操作数据，需要先和数据建立一个连接，最直接的方式就是通过终端来连接；

有两种方式来连接，这两种方式的区别在于输入密码是直接输入，还是另起一行以密文的形式输入；

- 方式一：
  `mysql -uroot -p<password>`
- 方式二：
  `mysql -uroot -p
  Enter password: <password>`

#### 1.2 显示数据库

一个数据库软件中，可以包含很多个数据库，可以通过这条命令查看所有数据库：`show databases;`

`MySQL`默认的数据库：

- `infomation_schema`：信息数据库，其中包括`MySQL`在维护的其他数据库、表、列、访问权限等信息；

- `performance_schema`：性能数据库，记录着`MySQL Server`数据库引擎在运行过程中的一些资源消耗相关的信息；

- `mysql`：用于存储数据库管理者的用户信息、权限信息以及一些日志信息等；

- `sys`：相当于是一个简易版的`performance_schema`，将性能数据库中的数据汇总成更容易理解的形式；

#### 1.3 创建数据库-表

- 在终端直接创建一个属于自己的新的数据库`coderhub`（一般情况下一个新的项目会对应一个新的数据库）：`create database coderhub;`

- 使用我们创建的数据库`coderhub`：`use coderhub;`

- 在数据库中，创建一张表：

  `create table user(
          name varchar(20),
          age int,
          height double
  );`

  `use`后括号中为表的字段以及每个字段使用的类型，如：`varchar(20) int double`等

- 插入数据
  `insert into user (name, age, height) values ('why', 18, 1.88);`
  `insert into user (name, age, height) values ('kobe', 40, 1.98);`

### 二、GUI工具

在终端操作数据库有很多不方便的地方：

- 语句写出来没有高亮，并且不会有任何的提示；
- 复杂的语句分成多行，格式看起来并不美观，很容易出现错误；
- 终端中查看所有的数据库或者表非常的不直观和不方便；
- ......

所以在开发中，可以借助于一些GUI工具连接并操作数据库

### 三、`SQL`语句分类与操作

#### 3.1 `SQL`分类

常见的`SQL`语句可以分成四类：

- `DDL`（Data Definition Language）：数据定义语言；可以通过`DDL`语句对数据库或者表进行：创建、删除、修改等操作；
- `DML`（Data Manipulation Language）：数据操作语言；可以通过`DML`语句对表进行：添加、删除、修改等操作；
- `DQL`（Data Query Language）：数据查询语言；可以通过`DQL`从数据库中查询记录；
- `DCL`（Data Control Language）：数据控制语言；对数据库、表格的权限进行相关访问控制操作；

#### 3.2 数据库操作

1. 查看当前数据库：

   - 查看所有的数据：
     `SHOW DATABASES;`

   - 使用某一个数据库：
     `USE coderhub;`

   - 查看当前正在使用的数据库：
     `SELECT DATABASE();`

2. 创建新的数据库：

   - 创建数据库语句：
     `CREATE DATABASE bilibili;`

     但这样创建数据库在该数据库已经存在的前提下会抛出错误，导致程序崩溃，所以可以添加限定条件

   - 不存在时创建数据库：

     `CREATE DATABASE IF NOT EXISTS bilibili;`

   - 创建时指定字符集以及排序规则：

     `CREATE DATABASE IF NOT EXISTS bilibili
     DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci;`

3. 删除数据库：

   - 删除数据库语句：

     `DROP DATABASE bilibili;`

     但这样删除数据库在数据库不存在时同样会抛出错误，所以也要添加限制条件

   - 存在时删除数据库：

     `DROP DATABASE	 IF EXISTS bilibili;`

4. 修改数据库：

   - 修改数据库的字符集和排序规则
     `ALTER DATABASE bilibili CHARACTER SET = utf8 COLLATE = utf8_unicode_ci;`

#### 3.3 数据表的操作

1. 查看数据表

   - 查看所有的数据表：`SHOW TABLES;`

   - 查看某一个表结构，如表的字段及类型等：`DESC user;`

   - 查看创建表时所用的`SQL`语句：

     ```sql
     SHOW CREATE TABLE `students`;
     ```

2. 创建数据表

   ```sql
   CREATE TABLE IF NOT EXISTS `users`(
   name VARCHAR(20),
   age INT,
   height DOUBLE
   );
   ```

   在创建表时，将表的名字用``包裹起来，是防止表明与某些关键字冲突，表的字段也可以用这种方法防止冲突

3. 删除数据表

   ```sql
   DROP TABLE IF EXISTS `USER`;
   ```

### 三、`SQL`数据类型

不同的数据会划分为不同的数据类型，在数据库中也是一样。`MySQL`支持的数据类型有：数字类型，日期和时间类型，字符串（字符和字节）类型，空间类型和`JSON`数据类型。

#### 3.1 数字类型

`MySQL`的数字类型有很多：

1. 整数数字类型：`INTEGER`，`INT`，`SMALLINT`，`TINYINT`，`MEDIUMINT`，`BIGINT`；

   | 类型        | 字节 | 有符号最小值 | 无符号最小值 | 有符号最大值 | 无符号最大值 |
   | ----------- | ---- | ------------ | ------------ | ------------ | ------------ |
   | `TINYINT`   | 1    | -128         | 0            | 127          | 255          |
   | `SMALLINT`  | 2    | -32768       | 0            | 32767        | 65535        |
   | `MEDIUMINT` | 3    | -8388608     | 0            | 8388607      | 16777215     |
   | `INT`       | 4    | -2147483648  | 0            | 2147483647   | 4294967295   |
   | `BIGINT`    | 8    | -2^63^       | 0            | 2^63^-1      | 2^64^-1      |

2. 浮点数字类型：`FLOAT`，`DOUBLE`（FLOAT是4个字节，DOUBLE是8个字节）；

3. 精确数字类型：`DECIMAL`，`NUMERIC`（DECIMAL是NUMERIC的实现形式）；

   `DECIMAL`用法：`DECIMAL(a, b)`其中a为储存字节长度，b为保留小数位数

#### 3.2 日期类型

`MySQL`的日期类型也很多：

1. `YEAR`以`YYYY`格式显示值：

   范围 1901到2155，和0000。

2. `DATE`类型用于具有日期部分但没有时间部分的值：

   DATE以格式`YYYY-MM-DD`显示值；支持的范围是 '1000-01-01' 到'9999-12-31'；

3. `DATETIME`类型用于包含日期和时间部分的值：

   `DATETIME`以格式`'YYYY-MM-DD hh:mm:ss'`显示值；支持的范围是`1000-01-01 00:00:00`到`9999-12-31 23:59:59`;

4. `TIMESTAMP`数据类型被用于同时包含日期和时间部分的值：

   `TIMESTAMP`以格式`'YYYY-MM-DD hh:mm:ss'`显示值；但是它的范围是`UTC`的时间范围：`'1970-01-01 00:00:01'`到`'2038-01-19 03:14:07'`;

5. 另外：`DATETIME`或`TIMESTAMP` 值可以包括在高达微秒（6位）精度的后小数秒一部分；比如`DATETIME`表示的范围可以是`'1000-01-01 00:00:00.000000'`到`'9999-12-31 23:59:59.999999'`;

#### 3.3 字符串类型

`MySQL`的字符串类型表示方式如下：

1. `CHAR`类型在创建表时为固定长度，长度可以是0到255之间的任何值；在被查询时，会删除后面的空格;

2. `VARCHAR`类型的值是可变长度的字符串，长度可以指定为0到65535之间的值；在被查询时，不会删除后面的空格；

3. `BINARY`和`VARBINARY`  类型用于存储二进制字符串，存储的是字节字符串；

   https://dev.mysql.com/doc/refman/8.0/en/binary-varbinary.html

4. `BLOB`用于存储大的二进制类型；

5. `TEXT`用于存储大的字符串类型

### 四、表约束

#### 4.1 主键(PRIMARY KEY)

在一张表中，为了区分每一条记录的唯一性，必须有一个字段是永远不会重复，并且不会为空的，这个字段通常会将它设置为主键：

- 主键是表中唯一的索引；
- 并且必须是NOT NULL的，如果没有设置NOT NULL，那么`MySQL`也会隐式的设置为NOT NULL；
- 主键也可以是多列索引，PRIMARY KEY(key_part, ...)，一般称之为联合主键；
- 开发中主键字段应该是和业务无关的，尽量不要使用业务字段来作为主键；

#### 4.2 唯一(UNIQUE)

某些字段在开发中希望是唯一的，不会重复的，比如手机号码、身份证号码等，这个字段可以使用UNIQUE来约束：

- 使用UNIQUE约束的字段在表中必须是不同的；
- 对于所有引擎，UNIQUE 索引允许NULL包含的列具有多个值NULL，也就是默认情况下`UNIQUE`唯一允许重复的值就是NULL，除非显示地约束为`NOT NULL`

#### 4.3 不能为空(NOT NULL)

某些字段要求用户必须插入值，不可以为空，这个时候可以使用NOT NULL 来约束；

#### 4.4 默认值(DEFAULT)

某些字段希望在没有设置值时给予一个默认值，这个时候可以使用DEFAULT来完成；

#### 4.5 自动递增(AUTO_INCREMENT)

某些字段希望不设置值时可以进行递增，比如用户的id，这个时候可以使用AUTO_INCREMENT来完成；一般自增的字段都是数字类型

### 五、表操作

#### 5.1 完整地创建一个表

```mysql
CREATE TABLE IF NOT EXISTS `users`(
	id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(20) NOT NULL,
    age INT DEFAULT 0,
    phoneNumber VARCHAR(20) UNIQUE DEFAULT '',
    createTime TIMESTAMP
)
```

在以上代码中，创建了一个名为`users`并包含以下字段的表：

1. `id`，int类型，设置为主键(隐式限定为NOT NULL)，并且自增
2. `name`，varchar类型且最大长度为20字节，非空
3. `age`，int类型，默认值为0
4. `phoneNumber`，varchar类型且最大长度为20字节，唯一，默认值为空字符串
5. `createTime`，timestamp类型

#### 5.2 修改表

1. 修改表名

   ```mysql
   ALTER TABLE `users` RENAME TO `user`;
   ```

   将`users`改为`user`

2. 添加新的列

   ```mysql
   ALTER TABLE `user` ADD `updateTime` TINMESTAMP;
   ```

   在添加新的字段时必须添加字段类型

3. 修改字段名称

   ```mysql
   ALTER TABLE `user` CHANGE `phoneNumber` `telNumber` VARCHAR(20)
   ```

   在修改字段名称时也要添加字段类型

4. 修改字段类型

   ```mysql
   ALTER TABLE `user` MODIFY `name` VARCHAR(30)
   ```

   将`name`字段类型修改为`VARCHAR(30)`

5. 删除某个字段

   ```mysql
   ALTER TABLE `user` DROP `age`;
   ```

   将`age`字段删除

6. 根据一张表的结构创建一张表

   ```mysql
   CREATE TABLE `user1` LIKE `user`
   ```

   新建的表的结构和`user`完全一致，包括字段类型和表约束，但不会复制`user`中的内容

7. 根据一张表中的所有内容，创建一张表

   ```mysql
   CREATE TABLE `user2` (SELECT * FROM `user`);
   ```

   新建的表会复制`user`中的所有内容，但没有复制表约束

#### 5.3 `DML`语句

`DML`：Data Manipulation Language（数据操作语言），用于对表进行增删改

1. 插入数据

   ```mysql
   INSERT INFO `user` VALUES (110, 'why', '020-110110', '2020-10-10', '2020-11-11')
   ```

   但在实际开发中一般不会这样插入，因为`id`一般是自动生成的，一般实际开发中的写法如下：

   ```mysql
   INSERT INFO `user` (name, telPhone, createTime, updateTime)
               VALUES ('KOBE', '020-111111', '2020-10-10', '202011-11')
   ```

   此时，`kobe`的`id`虽然没有写，但会根据上一条数据自动递增为111；

   但有时`createTime`和`updateTime`不会填写，也是自动生成的，会写成这样：

   ```mysql
   INSERT INFO `user` (name, telPhone)
               VALUES ('lilei', '020-561451')
   ```

   但这样会导致写入的`lilei`的`createTime`和`updateTime`会为NULL，此时可以更新这两个字段的类型：

   ```mysql
   ALTER TABLE `user` MODIFY `createTime` TIMESTAMP DEFAULT CURRENT_TIMESTAMP;
   ALTER TABLE `user` MODIFY `updateTime` TIMESTAMP DEFAULT CURRENT_TIMESTAMP 
                                                    ON UPDATE CURRENT_TIMESTAMP;
   ```

   此时插入时会插入当前时间，在更新时更新`updateTime`

2. 删除数据

   - 删除表中所有数据

     ```mysql
     DELETE FROM `user`;
     ```

   - 删除表中符合条件的数据

     ```mysql
     DELETE FROM `user` WHERE id = 110;
     ```

     删除`id`为110的数据

3. 更新数据

   - 更新所有数据

     ```mysql
     UPDATE `user` SET name = 'lily', telPhone = '020-515141';
     ```

   - 更新符合条件的数据

     ```mysql
     UPDATE `user` SET name = 'lily', telPhone = '020-895181' WHERE id = 115;
     ```

#### 5.4 `DQL`语句

`DQL`：Data Query Language（数据查询语言）用于对数据库数据的查询

- SELECT用于从一个或者多个表中检索选中的行（Record）。

查询的格式如下：
`SELECT select_expr [, select_expr]...
        [FROM table_references]
        [WHERE where_condition]
        [ORDER BY expr [ASC | DESC]]
        [LIMIT {[offset,] row_count | row_count OFFSET offset}]
        [GROUP BY expr]
        [HAVING where_condition]`

- 基本查询

  1. 查询所有的数据并且显示所有的字段：

     ```mysql
     SELECT * FROM `products`;
     ```

  2. 查询title、brand、price：

     ```mysql
     SELECT title, brand, price FROM `products`;
     ```

  3. 我们也可以给字段起别名，别名一般在多张表或者给客户端返回对应的key时会使用到：

     ```mysql
     SELECT title as t, brand as b, price as p FROM `products`;
     ```

- `WHERE`条件查询

  1. 比较运算符

     - 查询价格小于1000的手机

       ```mysql
       SELECT * FROM `products` WHERE price < 1000;
       ```

     - 查询价格大于等于2000的手机

       ```mysql
       SELECT * FROM `products` WHERE price >= 2000;
       ```

     - 价格等于3399的手机

       ```mysql
       SELECT * FROM `products` WHERE price = 3399;
       ```

     - 价格不等于3399的手机

       ```mysql
       SELECT * FROM `products` WHERE price != 3399;
       ```

     - 查询华为品牌的手机

       ```mysql
       SELECT * FROM `products` WHERE `brand` = '华为';
       ```

  2. 逻辑运算符

     - 查询价格范围价格为1000到2000的手机

       ```mysql
       SELECT * FROM `products` WHERE price > 1000 AND price < 2000;
       // 或者
       SELECT * FROM `products` WHERE price > 1000 && price < 2000;
       // 或 BETWEEN AND 包含等于
       SELECT * FROM `products` WHERE price BETWEEN 1000 AND 2000;
       ```

     - 询所有的华为手机或者价格小于1000的手机

       ```mysql
       SELECT * FROM `products` WHERE brand = '华为' or price < 1000
       ```

     - 查询品牌是华为，并且小于2000元的手机

       ```mysql
       SELECT * FROM `products` WHERE `brand` = '华为' and `price` < 2000;
       SELECT * FROM `products` WHERE `brand` = '华为' && `price` < 2000
       ```

     - 查询`url`为`null`的结果

       ```mysql
       SELECT * FROM `products` WHERE url IS NULL;
       ```

     - 查询`url`不为`null`的结果

       ```mysql
       SELECT * FROM `products` WHERE url IS NOT NULL;
       ```

  3. 模糊查询，使用`LIKE`关键字，结合两个特殊的符号：

     - `%`表示匹配任意个的任意字符；

     - `_`表示匹配一个的任意字符；

       1. 查询所有以v开头的title

          ```mysql
          SELECT * FROM `products` WHERE title LIKE 'v%';
          ```

       2. 查询带M的title

          ```mysql
          SELECT * FROM `products` WHERE title LIKE '%M%';
          ```

       3. 查询带M的title必须是第三个字符

          ```mysql
          SELECT * FROM `products` WHERE title LIKE '__M%
          ```

  4. `IN`，表示取多个值中的一个即可

     如：查询品牌为小米和华为的手机

     ```mysql
     SELECT * FROM `products` WHERE brand in ('华为', '小米');
     ```

- 查询结果排序
  当查询到结果的时候，希望将结果按照某种方式进行排序，这个时候使用的是ORDER BY；
  ORDER BY有两个常用的值：

  - `ASC`：升序排列；
  - `DESC`：降序排列；

  将查询结果按照价格升序排列：

  ```mysql
  SELECT * FROM `products` WHERE brand = '华为' or price < 1000 ORDER BY price ASC;
  ```

  在价格相同时按照评分降序排列：

  ```mysql
  SELECT * FROM `products` WHERE brand = '华为' or price < 1000
                                                  ORDER BY price ASC, score DESC;
  ```

- 分页查询

  当数据库中的数据非常多时，一次性查询到所有的结果进行显示是不太现实的：

  在真实开发中，我们都会要求用户传入`offset`、`limit`或者`page`等字段；它们的目的是让我们可以在数据库中进行分页查询；

  它的用法有

  1. LIMIT limit OFFSET offset

     ```mysql
     SELECT * FROM `products` LIMIT 30 OFFSET 0; #偏移为0,查询30条数据
     SELECT * FROM `products` LIMIT 30 OFFSET 30; #偏移为30,查询30条数据
     SELECT * FROM `products` LIMIT 30 OFFSET 60; #偏移为60,查询30条数据
     ```

  2. LIMIT offset, limit

     ```mysql
     SELECT * FROM `products` LIMIT 90, 30; #偏移为90,查询30条数据
     ```

