### 一、聚合函数

聚合函数表示对值集合进行操作的组（集合）函数

如：

1. 华为手机价格的平均值

   ```mysql
   SELECT AVG(price) FROM `products` WHERE brand = '华为';
   ```

2. 计算所有手机的平均分

   ```mysql
   SELECT AVG(score) FROM `products`;
   ```

3. 手机中最低和最高分数

   ```mysql
   SELECT MAX(score) FROM `products`;
   SELECT MIN(score) FROM `products`;
   ```

4. 计算总投票人数

   ```mysql
   SELECT SUM(voteCnt) FROM `products`;
   ```

5. 计算所有条目的数量

   ```mysql
   SELECT COUNT(*) FROM `products`;
   ```

6. 华为手机的个数

   ```mysql
   SELECT COUNT(*) FROM `products` WHERE brand = '华为';
   ```

   在`COUNT`函数中，*表示统计所有的数据，如果传入具体的字段，则会统计结果中该字段不为空的总数，如：

   ```mysql
   SELECT COUNT(url) FROM `products` WHERE brand = '华为';#查询url不为空的华为手机总数
   ```

   在统计的过程中，如果想要剔除某些字段重复的数据，可以使用`DISTINCT`：

   ```mysql
   SELECT COUNT(DISTINCT PRICE) FROM `products` WHERE brand = '华为';
   ```

### 二、Group By

#### 2.1 Group By使用

事实上聚合函数相当于默认将所有的数据分成了一组：

- 前面使用avg还是max等，都是将所有的结果看成一组来计算的；
- 那么如果希望划分多个组：比如华为、苹果、小米等手机分别的平均价格，应该怎么来做呢？
- 这个时候可以使用GROUP BY；

GROUP BY通常和聚合函数一起使用：

- 表示先对数据进行分组，再对每一组数据，进行聚合函数的计算；

我们现在来提一个需求：

- 根据品牌进行分组；

- 计算各个品牌中：商品的个数、平均价格；也包括：最高价格、最低价格、平均评分；

  ```mysql
  SELECT brand,  COUNT(*) as count, AVG(price) as avgPrice, MAX(price) as maxPrice,
  MIN(price) as minPrice,AVG(score) as avgScore FROM `products` GROUP BY brand;
  ```

#### 2.2 Group By约束

如果希望给Group By查询到的结果添加一些约束，那么可以使用：HAVING。

比如：在按品牌分组查询并统计平均价格、平均评分得到的结果中，筛选平均价格在4000以下，并且平均分在7以上的品牌：

```mysql
SELECT brand, AVG(price) as avgPrice, AVG(score) as avgScore FROM `products` GROUP BY brand HAVING avgPrice < 4000 and avgScore > 7;
```

需求：查询平均分大于7.5分的手机，按品牌进行分类，求平均价格：

1. 先查询评分大于7.5的手机

   ```mysql
   SELECT * FROM `products` WHERE score > 7.5
   ```

2. 对结果进行价格的平均计算并按品牌分类

   ```mysql
   SELECT brand, AVG(price) FROM `products` WHERE score > 7.5 GROUP BY brand
   ```

### 三、多表

#### 3.1 创建多表

假如在上面的商品表中，对应的品牌还需要包含其他的信息：

- 比如品牌的官网，品牌的世界排名，品牌的市值等等；

- 如果直接在商品中去体现品牌相关的信息，会存在一些问题：

  - 一方面，products表中应该表示的都是商品相关的数据，应该又另外一张表来表示brand的数据；
  - 另一方面，多个商品使用的品牌是一致时，会存在大量的冗余数据；

- 所以，可以将所有的批评数据，单独放到一张表中，创建一张品牌的表：

  ```mysql
  CREATE TABLE IF NOT EXISTS `brand`(
  id INT PRIMARY KEY AUTO_INCREMENT,
  name VARCHAR(20) NOT NULL,
  website VARCHAR(100),
  worldRank INT
  );
  ```

- 插入数据：

  ```mysql
  INSERT INTO `brand` (name, website, worldRank) VALUES ('华为', 'www.huawei.com', 1);
  INSERT INTO `brand` (name, website, worldRank) VALUES ('小米', 'www.mi.com', 10);
  INSERT INTO `brand` (name, website, worldRank) VALUES ('苹果', 'www.apple.com', 5);
  INSERT INTO `brand` (name, website, worldRank) VALUES ('oppo', 'www.oppo.com', 15);
  INSERT INTO `brand` (name, website, worldRank) VALUES ('京东', 'www.jd.com', 3);
  INSERT INTO `brand` (name, website, worldRank) VALUES ('Google', 'www.google.com', 8);
  ```

#### 3.2 创建外键

将两张表联系起来，我们可以将products中的brand_id关联到brand中的id：

- 如果是创建表添加外键约束，我们需要在创建表的()最后添加如下语句

  ```mysql
  FOREIGN KEY (brand_id) REFERENCES brand(id)
  ```

- 如果是表已经创建好，额外添加外键：

  ```mysql
  ALTER TABLE `products` ADD FOREIGN KEY (brand_id) REFERENCES brand(id);
  ```

- 现在可以将products中的brand_id关联到brand中的id的值：

  ```mysql
  UPDATE `products` SET `brand_id` = 1 WHERE `brand` = '华为';
  UPDATE `products` SET `brand_id` = 4 WHERE `brand` = 'OPPO';
  UPDATE `products` SET `brand_id` = 3 WHERE `brand` = '苹果';
  UPDATE `products` SET `brand_id` = 2 WHERE `brand` = '小米';
  ```


#### 3.3 更新外键

如果products中引用的外键被更新了或者删除了，这个时候会出现什么情况呢？

我们来进行一个更新操作：比如将华为的id更新为100：

```mysql
UPDATE `brand` SET id = 100 WHERE id = 1;
```

这个时候执行代码是报错的，因为在设置外键时没有设置ACTION，所以取默认值，ACTION可设置的值有：

- RESTRICT（默认属性）：当更新或删除某个记录时，会检查该记录是否有关联的外键记录，有的话会报错的，不允许更新或删除；
- NO ACTION：和RESTRICT是一致的，是在`SQL`标准中定义的；
- CASCADE：当更新或删除某个记录时，会检查该记录是否有关联的外键记录，有的话：
  - 更新：那么会更新对应的记录；
  - 删除：那么关联的记录会被一起删除掉；
- SET NULL：当更新或删除某个记录时，会检查该记录是否有关联的外键记录，有的话，将对应的值设置为NULL；

所以如果想更新或删除引用的外键的值，必须更改ACTION，更改已存在的外键ACTION分为三个步骤：

1. 查看表结构：

   ```mysql
   SHOW CREATE TABLE `products`;
   ```

   ![查看外键名称](https://gitee.com/Topcvan//img-storage/raw/master//node/%E6%9F%A5%E7%9C%8B%E5%A4%96%E9%94%AE%E5%90%8D%E7%A7%B0.png)

   这个时候，可以知道外键的名称是`products_ibfk_1`。

2. 第二步：删除之前的外键

   ```mysql
   # 删除之前的外键
   ALTER TABLE `products` DROP FOREIGN KEY products_ibfk_1;
   ```

3. 第三步：添加新的外键，并且设置新的action

   ```mysql
   ALTER TABLE `products` ADD FOREIGN KEY (brand_id) 
   REFERENCES brand(id) 
   ON UPDATE CASCADE;
   ```

   在添加ACTION时，一般将UPDATE设置为CASCADE，DELETE为默认值

#### 3.4 多表查询

如果希望查询到产品的同时，显示对应的品牌相关的信息，因为数据是存放在两张表中，所以这个时候就需要进行多表查询。

如果直接通过查询语句希望在多张表中查询到数据，这个时候是什么效果呢？

```mysql
SELECT * FROM `products`, `brand`;
```

会发现一共有648条数据，这个数据量是如何得到的呢？

- 第一张表的108条* 第二张表的6条数据；
- 也就是说第一张表中每一个条数据，都会和第二张表中的每一条数据结合一次；
- 这个结果称之为笛卡尔乘积，也称之为直积，表示为X*Y；

但是事实上很多的数据是没有意义的，比如华为和苹果、小米的品牌结合起来的数据就是没有意义的，可不可以进行筛选呢？

- 使用where来进行筛选；

  ```mysql
  SELECT * FROM `products`, `brand` WHERE `products`.brand_id = `brand`.id;
  ```

- 这个表示查询到笛卡尔乘积后的结果中，符合`products.brand_id = brand.id`条件的数据过滤出来；

事实上对多表查询的分步操作，可以使用`SQL JOIN`来完成，`SQL JOIN`主要有四种常用的连接方式：

1. 左连接
   如果我们希望获取到的是左边所有的数据（以左表为主）：

   - 这个时候就表示无论左边的表是否有对应的brand_id的值对应右边表的id，左边的数据都会被查询出来；

   - 这个也是开发中使用最多的情况，它的完整写法是LEFT [OUTER] JOIN，但是OUTER可以省略的；
     ![左连接](https://gitee.com/Topcvan//img-storage/raw/master//node/%E5%B7%A6%E8%BF%9E%E6%8E%A5.png)

     ```mysql
     # 查询所有的手机(包括没有品牌信息的)以及对应的品牌信息
     SELECT * FROM `products` LEFT JOIN `brand` ON `products`.brand_id = `brand`.id;
     # 查询没有对应品牌信息的手机
     SELECT * FROM `products` LEFT JOIN `brand` ON `products`.brand_id = `brand`.id
     WHERE brand.id IS NULL;
     ```

2. 右连接
   如果我们希望获取到的是右边所有的数据（以由表为主）：

   - 这个时候就表示无论左边的表中的brand_id是否有和右边表中的id对应，右边的数据都会被查询出来；

   - 右连接在开发中没有左连接常用，它的完整写法是RIGHT [OUTER] JOIN，但是OUTER可以省略的；

     ![右连接](https://gitee.com/Topcvan//img-storage/raw/master//node/%E5%8F%B3%E8%BF%9E%E6%8E%A5.png)

     ```mysql
     # 查询所有的品牌(包括没有手机数据的)以及对应的手机数据
     SELECT * FROM `products` RIGHT JOIN `brand` ON `products`.brand_id = `brand`.id;
     # 查询没有对应手机数据的品牌信息
     SELECT * FROM `products` RIGHT JOIN `brand` ON `products`.brand_id = `brand`.id
     WHERE products.id IS NULL;
     ```

3. 内连接
   事实上内连接是表示左边的表和右边的表都有对应的数据关联：

   ![内连接](https://gitee.com/Topcvan//img-storage/raw/master//node/%E5%86%85%E8%BF%9E%E6%8E%A5.png)

   内连接有其他的写法：CROSS JOIN或者JOIN都可以；

   ```mysql
   SELECT * FROM `products` INNER JOIN `brand` ON `products`.brand_id = `brand`.id;
   ```

   我们会发现它和之前的下面写法是一样的效果：

   ```mysql
   SELECT * FROM `products`, `brand` WHERE `products`.brand_id = `brand`.id;
   ```

   但是他们代表的含义并不相同：

   - `SQL`语句一：内连接，代表的是在两张表连接时就会约束数据之间的关系，来决定之后查询的结果；
   - `SQL`语句二：where条件，代表的是先计算出笛卡尔乘积，在笛卡尔乘积的数据基础之上进行where条件的筛选；

4. 全连接
   `SQL`规范中全连接是使用FULL JOIN，但是`MySQL`中并没有对它的支持，需要使用UNION 来实现：

   - 全连接1：

     ![全连接1](https://gitee.com/Topcvan//img-storage/raw/master//node/%E5%85%A8%E8%BF%9E%E6%8E%A51.png)

     ```mysql
     (SELECT * FROM `products` LEFT JOIN `brand` ON `products`.brand_id = `brand`.id)
     UNION
     (SELECT * FROM `products` RIGHT JOIN `brand` ON `products`.brand_id = `brand`.id);
     ```

     相当于将左连接和右连接的查询结果取并集，`MySQL`会过滤重复的数据

   - 全连接2：

     ![全连接2](https://gitee.com/Topcvan//img-storage/raw/master//node/%E5%85%A8%E8%BF%9E%E6%8E%A52.png)

     ```mysql
     (SELECT * FROM `products` LEFT JOIN `brand` ON `products`.brand_id = `brand`.id WHERE `brand`.id IS NULL)
     UNION
     (SELECT * FROM `products` RIGHT JOIN `brand` ON `products`.brand_id = `brand`.id WHERE `products`.id IS NULL);
     ```

### 四、多对多关系表

在开发中我们还会遇到多对多的关系：

- 比如学生可以选择多门课程，一个课程可以被多个学生选择；
- 这种情况在开发中应该如何处理呢？

先建立好两张表：

- 创建学生表

  ```mysql
  CREATE TABLE IF NOT EXISTS `students`(
  id INT PRIMARY KEY AUTO_INCREMENT,
  name VARCHAR(20) NOT NULL,
  age INT
  );
  ```

- 创建课程表

  ```mysql
  CREATE TABLE IF NOT EXISTS `courses`(
  id INT PRIMARY KEY AUTO_INCREMENT,
  name VARCHAR(20) NOT NULL,
  price DOUBLE NOT NULL
  );
  ```

- 插入数据

  ```mysql
  INSERT INTO `students` (name, age) VALUES('why', 18);
  INSERT INTO `students` (name, age) VALUES('tom', 22);
  INSERT INTO `students` (name, age) VALUES('lilei', 25);
  INSERT INTO `students` (name, age) VALUES('lucy', 16);
  INSERT INTO `students` (name, age) VALUES('lily', 20);
  
  INSERT INTO `courses` (name, price) VALUES ('英语', 100);
  INSERT INTO `courses` (name, price) VALUES ('语文', 666);
  INSERT INTO `courses` (name, price) VALUES ('数学', 888);
  INSERT INTO `courses` (name, price) VALUES ('历史', 80);
  ```

还需要一个关系表来记录两张表中的数据关系：

- 创建关系表

  ```mysql
  CREATE TABLE IF NOT EXISTS `students_select_courses`(
  id INT PRIMARY KEY AUTO_INCREMENT,
  student_id INT NOT NULL,
  course_id INT NOT NULL,
  FOREIGN KEY (student_id) REFERENCES students(id) ON UPDATE CASCADE,
  FOREIGN KEY (course_id) REFERENCES courses(id) ON UPDATE CASCADE
  );
  ```

  如果需要防止插入学生id和课程id同时重复的数据，可将学生id和课程id作为联合主键：

  ```mysql
  CREATE TABLE IF NOT EXISTS `students_select_courses`(
      student_id INT NOT NULL,
      course_id INT NOT NULL,
      FOREIGN KEY (student_id) REFERENCES students(id) ON UPDATE CASCADE,
      FOREIGN KEY (course_id) REFERENCES courses(id) ON UPDATE CASCADE,
      PRIMARY KEY (student_id, course_id)
  );
  ```

- 插入选课数据

  ```mysql
  # why 选修了 英文和数学
  INSERT INTO `students_select_courses` (student_id, course_id) VALUES (1, 1);
  INSERT INTO `students_select_courses` (student_id, course_id) VALUES (1, 3);
  # lilei选修了 语文和数学和历史
  INSERT INTO `students_select_courses` (student_id, course_id) VALUES (3, 2);
  INSERT INTO `students_select_courses` (student_id, course_id) VALUES (3, 3);
  INSERT INTO `students_select_courses` (student_id, course_id) VALUES (3, 4);
  ```

#### 4.1 查询多条数据

- 查询所有有选课的学生选择的所有课程

  此时需要将学生表和关系表进行内连接：

  ```mysql
  SELECT * FROM students stu JOIN student_select_course ssc ON stu.id = ssc.student_id
  ```

  得到所有有选课的学生以及对应的选课id信息

  然后将查询结果与课程表进行内连接：

  ```mysql
  SELECT * FROM students stu JOIN student_select_course ssc ON stu.id = ssc.student_id
  JOIN courses cs ON ssc.course_id = cs.id
  ```

  查询结果：

  ![学生选课情况](https://gitee.com/Topcvan//img-storage/raw/master//node/%E5%AD%A6%E7%94%9F%E9%80%89%E8%AF%BE%E6%83%85%E5%86%B5.png)

  但在结果中只需要学生的信息字段和课程信息字段，不需要关系表的信息字段，所以可以添加展示的字段：

  ```mysql
  SELECT stu.id id, stu.name stuName, stu.age stuAge, cs.id, csId,
  cs.name csName, cs.price csPrice
  FROM students stu 
  JOIN student_select_course ssc ON stu.id = ssc.student_id
  JOIN courses cs ON ssc.course_id = cs.id
  ```

  查询结果：

  ![学生选课情况1](https://gitee.com/Topcvan//img-storage/raw/master//node/%E5%AD%A6%E7%94%9F%E9%80%89%E8%AF%BE%E6%83%85%E5%86%B51.png)

- 查询所有的学生选课情况
  此时查询的结果应该包含所有学生，包括未选课的，所以需要使用左连接：

  ```mysql
  SELECT 
  stu.id studentId, stu.name studentName, cs.id courseId, cs.name courseName, cs.price coursePrice
  FROM `students` stu
  LEFT JOIN `students_select_courses` ssc
  ON stu.id = ssc.student_id
  LEFT JOIN `courses` cs 
  ON ssc.course_id = cs.id; 
  ```

#### 4.2 查询单个学生数据

- 先将学生表与关系表建立连接，为防止查询的学生未选课，选择左连接：

  ```mysql
  SELECT 
  stu.id studentId, stu.name studentName, cs.id courseId, cs.name courseName, cs.price coursePrice
  FROM `students` stu
  LEFT JOIN `students_select_courses` ssc
  ON stu.id = ssc.student_id
  ```

- 然后将该结果再与课程表建立左连接：

  ```mysql
  SELECT 
  stu.id studentId, stu.name studentName, cs.id courseId, cs.name courseName, cs.price coursePrice
  FROM `students` stu
  LEFT JOIN `students_select_courses` ssc
  ON stu.id = ssc.student_id
  LEFT JOIN `courses` cs 
  ON ssc.course_id = cs.id
  ```

- 最后将连接查询的结果再按`id`进行条件筛选：

  ```mysql
  SELECT 
  stu.id studentId, stu.name studentName, cs.id courseId, cs.name courseName, cs.price coursePrice
  FROM `students` stu
  LEFT JOIN `students_select_courses` ssc
  ON stu.id = ssc.student_id
  LEFT JOIN `courses` cs 
  ON ssc.course_id = cs.id
  WHERE stu.id = 1; 
  ```

#### 4.3 查询未选课学生及未被选课程

- 未选课学生

  1. 先将学生表与关系表进行左连接：

     ```mysql
     SELECT 
     stu.id studentId, stu.name studentName, cs.id courseId, cs.name courseName, cs.price coursePrice
     FROM `students` stu
     LEFT JOIN `students_select_courses` ssc
     ON stu.id = ssc.student_id
     ```

  2. 再将该查询结果与课程表进行左连接

     ```mysql
     SELECT 
     stu.id studentId, stu.name studentName, cs.id courseId, cs.name courseName, cs.price coursePrice
     FROM `students` stu
     LEFT JOIN `students_select_courses` ssc
     ON stu.id = ssc.student_id
     LEFT JOIN `courses` cs
     ON ssc.course_id = cs.id
     ```

  3. 最后，将连接查询结果按条件进行筛选：

     ```mysql
     SELECT 
     stu.id studentId, stu.name studentName, cs.id courseId, cs.name courseName, cs.price coursePrice
     FROM `students` stu
     LEFT JOIN `students_select_courses` ssc
     ON stu.id = ssc.student_id
     LEFT JOIN `courses` cs
     ON ssc.course_id = cs.id
     WHERE cs.id IS NULL;
     ```

- 未被选择的课程：

  与查询未选课学生类似，不过需要以课程表为主体，所以使用右连接：

  ```mysql
  SELECT 
  stu.id studentId, stu.name studentName, cs.id courseId, cs.name courseName, cs.price coursePrice
  FROM `students` stu
  RIGHT JOIN `students_select_courses` ssc
  ON stu.id = ssc.student_id
  RIGHT JOIN `courses` cs
  ON ssc.course_id = cs.id
  WHERE stu.id IS NULL;
  ```

### 五、字段合并转化

前面学习的查询语句，查询到的结果通常是一张表，比如查询手机+品牌信息：

```mysql
SELECT * FROM products LEFT JOIN brand ON products.brand_id = brand.id;
```

结果：

![商品品牌信息结果](https://gitee.com/Topcvan//img-storage/raw/master//node/%E5%95%86%E5%93%81%E5%93%81%E7%89%8C%E4%BF%A1%E6%81%AF%E7%BB%93%E6%9E%9C.png)

#### 5.1 将多个字段转化为`JSON`

但是在真实开发中，实际上红色圈起来的部分应该放入到一个对象中，那么可以使用下面的查询方式：

这个时候我们要用`JSON_OBJECT`

```mysql
SELECT products.id as id, products.title as title, products.price as price, products.score as score, 
JSON_OBJECT('id', brand.id, 'name', brand.name, 'rank', brand.phoneRank, 'website', brand.website) as brand
FROM products LEFT JOIN brand ON products.brand_id = brand.id;
```

查询结果：

![brand转json](https://gitee.com/Topcvan//img-storage/raw/master//node/brand%E8%BD%ACjson.png)

这种转化一般是用在一对多关系表中

#### 5.2 多对多转成数组

在多对多关系中，一般希望查询得到的是一个数组。比如一个学生的多门课程信息，应该是放到一个数组中的；数组中存放的是课程信息的一个个对象；

这个时候要将`JSON_ARRAYAGG`和`JSON_OBJECT`结合来使用：

```mysql
SELECT stu.id, stu.name, stu.age, 
JSON_ARRAYAGG(JSON_OBJECT('id', cs.id, 'name', cs.name)) as courses 
FROM students stu
LEFT JOIN students_select_courses ssc ON stu.id = ssc.student_id
LEFT JOIN courses cs ON ssc.course_id = cs.id
GROUP BY stu.id;
```

### 六、node中操作数据库

前面所有的操作都是在GUI工具中，通过执行`SQL`语句来获取结果的，但真实开发中肯定是通过代码来完成所有的操作的。

在node中可以借助于第三方库`MySQL`以在代码中执行`SQL`语句。

安装：`npm install mysql2`

#### 6.1 `MySQL`简单使用

mysql2的使用过程如下：

- 第一步：创建连接（通过`createConnection`），并且获取连接对象；

- 第二步：执行`SQL`语句即可（通过query）；

  ```javascript
  const mysql = require('mysql2')
  
  const connection = mysql.createConnection({
      host: 'localhost',
      database: 'coderhub',
      user: 'root',
      password: '2518114515'
  })
  
  const statement = `SELECT * FROM products`
  
  connection.query(statement, (err, result, fields) => {
      console.log(result)
  })
  ```

- 第三步：关闭连接

  在执行完`SQL`语句后，连接继续保持，如果想关闭连接，需要调用`connection.end()`来关闭连接，并且可以监听`error`事件来添加关闭过程中抛出的错误处理函数；此外，还可以使用`connection.destory()`来强制关闭连接，此时无法监听错误事件

  ```javascript
  connection.query(statement, (err, result, fields) => {
      console.log(result)
      connection.end()
  })
  
  connection.on('error', (err) => {
      console.log(err)
  })
  ```

#### 6.2 预编译语句

Prepared Statement（预编译语句）具有以下优点：

- 提高性能：将创建的语句模块发送给`MySQL`，然后`MySQL`编译（解析、优化、转换）语句模块，并且存储它但是不执行，之后在真正执行时会给?提供实际的参数才会执行；如果再次执行该语句，它将会从LRU（Least Recently Used）Cache中获取获取，省略了编译statement的时间来提高性能；

- 防止`SQL`注入：之后传入的值不会像模块引擎那样编译，那么一些`SQL`注入的内容不会被执行；如`or 1 = 1`不会被执行；

  ```javascript
  const statement = `SELECT * FROM products WHERE price < ? AND score > ?`
  
  connection.execute(statement, [6000, 7],(err, result, fields) => {
      console.log(result)
  })
  ```

#### 6.3 连接池

前面是创建了一个连接（connection），但是如果有多个请求的话，该连接很有可能正在被占用，那么是否需要每次一个请求都去创建一个新的连接呢？

事实上，`mysql2`提供了连接池（connection pools）；连接池可以在需要的时候自动创建连接，并且创建的连接不会被销毁，会放到连接池中，后续可以继续使用；可以在创建连接池的时候设置LIMIT，也就是最大创建个数；

```javascript
const pool = mysql.createPool({
    host: 'localhost',
    database: 'coderhub',
    user: 'root',
    password: '4185151451',
    connectionLimit: 10
})

const statement = `
	SELECT * FROM products WHERE price < ? AND score > ?;
`
pool.execute(statement, [6000, 7], (err, res, fields) => {
    console.log(res)
    pool.end()
})
```

#### 6.4 Promise方式

目前在JavaScript开发中更习惯`Promise`和`await`、`async`的方式，`mysql2`同样是支持的：

```javascript
pool.promise().execute(statement, [6000, 7]).then([res, fields] => {
    console.log(res)
}).catch(err => console.log(err))
```

#### 6.5 `ORM`

对象关系映射（英语：Object Relational Mapping，简称ORM，或O/RM，或O/R mapping），是一种程序
设计的方案：

- 从效果上来讲，它提供了一个可在编程语言中，使用虚拟对象数据库的效果；比如在Java开发中经常使用的`ORM`包括：`Hibernate`、`MyBatis`；Node当中的`ORM`通常使用的是`sequelize`;

- 如果希望将`Sequelize`和`MySQL`一起使用，那么需要先安装这两个东西：

  - `mysql2`：`sequelize`在操作`mysql`时使用的是`mysql2`；

  - `sequelize`：使用它来让对象映射到表中；

    ```javascript
    npm install sequelize mysql2
    ```

1. 基本使用

   `Sequelize`连接数据库：

   - 第一步：创建一个`Sequelize`的对象，并且指定数据库、用户名、密码、数据库类型、主机地址等；

     ```javascript
     const { Sequelize } = require('sequelize')
     
     const sequelize = new Sequelize('coderhub', 'root', '92151145', {
         host: 'localhost',
         dialect: 'mysql'
     })
     ```

   - 第二步：测试连接是否成功

     ```javascript
     sequelize.authenticate().then(() => console.log('连接成功'))
                             .catch(err => console.log('连接失败', err))
     ```

2. 单表操作

   在用`Sequelize`操作数据库时。还需导入`DataType`和`Model`：

   ```javascript
   const { Sequelize, DataTypes, Model } = require('sequelize')
   
   const sequelize = new Sequelize('coderhub', 'root', '92151145', {
       host: 'localhost',
       dialect: 'mysql'
   })
   ```

   将数据库中的表`products`映射到类`Product`中：

   ```javascript
   class Product extends Model {}
   Product.init({
       id: {
           type: DataTypes.INTERGER,
           primaryKey: true,
           autoIncrement: true
       },
       title: {
           type: DataTypes.STRING,
           allowNotNull: false
       },
       price: DataTypes.DOUBLE,
       score: DataTypes.DOUBLE
   }, {
       tableName: 'products',
       createAt: false, // 查询所有字段时会默认加上这两个字段,需要显式设置为false
       updateAt: false,
       sequelize // 将该类与打开的数据库连接起来
   })
   ```

   查询操作：

   ```javascript
   async function queryProducts() {
       // 查询表中所有内容
       const result = await Product.findAll()
       console.log(result)
       // 条件查询
       const result1 = await Product.findAll({
       	where: {
               price: {
                   [Op.gte]: 5000 // 查找价格大于5000的手机,需要在sequelize中引入Op
                                  // Op.gte表示大于等于(greater than or equal to)
               }
           }
   	})
       console.log(result1)
   }
   ```

   插入操作：

   ```javascript
   async function createProducts() {
       const result = await Product.create({
           title: '华为Nova',
           price: 3688,
           score: 5.5
       })
       console.log(result)
   }
   ```

   更新操作：

   ```javascript
   async function updateProducts() {
       const result = await Product.update({
           price: 3600
       }, {
           where: {
               id: 1
           }
       })
       console.log(result)
   }
   ```

3. 一对多操作：

   首先映射`brand`表到`Brand`类中：

   ```javascript
   class Brand extends Model {}
   Brand.init({
       id: {
           type: DataTypes.INTERGER,
           primaryKey: true,
           autoIncrement: true
       }, 
       name: {
           type: DataTypes.STRING,
           allowNotNull: false
       }, 
       website: DataTypes.STRING,
       phoneRank: DataTypes.INTERGER
   }, {
       tableName: 'brand',
       createAt: false,
       updateAt: false,
       sequelize
   })
   ```

   给Product添加外键字段，需要在Product初始化前将Brand初始化，否则无法将Brand联系起来：

   ```javascript
   class Product extends Model {}
   Product.init({
       id: {
           type: DataTypes.INTERGER,
           primaryKey: true,
           autoIncrement: true
       },
       title: {
           type: DataTypes.STRING,
           allowNotNull: false
       },
       price: DataTypes.DOUBLE,
       score: DataTypes.DOUBLE,
       brandId: {
           field: 'brand_id',
           type: DataTypes.INTERGER,
           reference: {
               model: Brand,
               key: 'id'
           }
       }
   }, {
       tableName: 'products',
       createAt: false, // 查询所有字段时会默认加上这两个字段,需要显式设置为false
       updateAt: false,
       sequelize // 将该类与打开的数据库连接起来
   })
   ```

   将两张表联系在一起：

   ```javascript
   Product.belongsto(Brand, {
       foreignKey: 'brandId'
   })
   ```

   联表查询：

   ```javascript
   async function queryProducts() {
       const result = Product.findAll({
           include: {
               model: Brand
           }
       })
   }
   ```

   在查询结果中，`brand`表中的信息会以数组形式存储在Brand字段中

4. 多对多操作

   将学生表、课程表和关系表初始化：

   ```javascript
   // Student
   class Student extends Model {}
   Student.init({
     id: {
       type: DataTypes.INTEGER,
       primaryKey: true,
       autoIncrement: true
     },
     name: {
       type: DataTypes.STRING,
       allowNotNull: false
     },
     age: DataTypes.INTEGER
   }, {
     tableName: 'students',
     createdAt: false,
     updatedAt: false,
     sequelize
   });
   
   // Course
   class Course extends Model {}
   Course.init({
     id: {
       type: DataTypes.INTEGER,
       primaryKey: true,
       autoIncrement: true
     },
     name: {
       type: DataTypes.STRING,
       allowNotNull: false
     },
     price: DataTypes.DOUBLE
   }, {
     tableName: 'courses',
     createdAt: false,
     updatedAt: false,
     sequelize
   });
   
   // StudentCourse
   class StudentCourse extends Model {}
   StudentCourse.init({
     id: {
       type: DataTypes.INTEGER,
       primaryKey: true,
       autoIncrement: true
     },
     studentId: {
       type: DataTypes.INTEGER,
       references: {
         model: Student,
         key: 'id'
       },
       field: 'student_id'
     },
     courseId: {
       type: DataTypes.INTEGER,
       references: {
         model: Course,
         key: 'id'
       },
       field: 'course_id'
     }
   }, {
     tableName: 'students_select_courses',
     createdAt: false,
     updatedAt: false,
     sequelize
   });
   ```

   建立多对多联系：

   ```javascript
   Student.belongsToMany(Course, {
     through: StudentCourse,
     foreignKey: 'studentId',
     otherKey: 'courseId'
   });
   
   Course.belongsToMany(Student, {
     through: StudentCourse,
     foreignKey: 'courseId',
     otherKey: 'studentId'
   });
   ```

   查询：

   ```javascript
   async function queryProducts() {
     const result = await Student.findAll({
       include: {
         model: Course
       }
     });
     console.log(result);
   }
   ```

   查询到的结果中，课程信息以数组形式包含在Course字段中

   