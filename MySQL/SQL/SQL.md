# SQL

[toc]





## **一、MySQL 基础操作**

### 1. MySQL 服务的启动和停止
**方式一：通过命令行**

```
net start 服务名
net stop 服务名
```

**方式二：计算机——右击——管理——服务**



### 2. MySQL 服务的登录和退出
**方式一：通过 mysql 自带的客户端**

只限于 root 用户

**方式二：通过命令行**

**登录**：

```
mysql 【-h 主机名 -P 端口号】 -u 用户名 -p密码   (这里-p和密码间无空格)
	        (可选的)
```

**退出**：`exit` 或 `Ctrl + C`



### 3. MySQL 的常用命令
1. 查看当前所有的数据库

   ```
   show database;
   ```

2. 打开指定的库

   ```
   use 库名;
   ```

3. 查看当前库的所有表

   ```
   show tables;
   ```

4. 查看其他库的所有表

   ```
   show tables from 库名;
   ```

5. 创建表

   ```
   create table 表名()
   		列名 列类型,
   		列名 列类型,
   		...
   )
   ```

6. 查看表结构

   ```
   desc 表名;
   ```

7. 查看服务器的版本

   方式一：登录到 mysql 服务端，执行：

   ```
   select version()；
   ```

   方式二：没有登录到 mysql 服务端，直接在命令行执行

   ```
   mysql --version	/ mysql -V
   ```

   

### 4. MySQL 的语法规范
1. **不区分大小写**，但建议关键字大写，表名、列名小写。

2. 每条命令最好用**分号结尾**。

3. 每条命令根据需要，可以进行缩进或换行。

4. 注释：

   ```sql
   单行注释：#注释文字
   单行注释：-- 注释文字
   多行注释：/* 注释文字 */
   ```

5.  \` ` 为着重号，可加可不加，是为了区分 MySQL 的保留字与普通字符而引入的符号，一般情况下可以不用，但是如果识别符出现关键字冲突或标识符的写法可能产生歧义的情况下就必须使用。

6. MySQL 中字符和字符串都用单引号

   ![1617810801464](./imgs/1617810801464.png)





## **二、DQL 语言（数据查询语言）**

### 1. 基础查询

#### 1.1 语法
```sql
select 查询列表
from 表名;
```

#### 1.2 特点
1. 查询列表可以是字段、常量、表达式、函数。

2. 查询结果是一个虚拟表。

#### 1.3 示例

1. 查询单个字段

   ```sql
   -- select 字段名 from 表名;
   
   SELECT last_name FROM employees;
   ```

2. 查询多个字段

   ```sql
   -- select 字段名，字段名 from 表名;
   
   SELECT last_name, salary, email FROM employees;
   ```

3. 查询所有字段

   ```sql
   -- select * from 表名
   -- 方式一:
   SELECT
       `employee_id`,
       `first_name`,
       `last_name`,
       `email`,
       `phone_number`,
       `job_id`,
       `salary`,
       `commission_pct`,
       `manager_id`,
       `department_id`,
       `hiredate`
   FROM
   	employees;
   -- 方式二:
   SELECT * FROM employees;
   ```

4. 查询常量

   ```sql
   /*
   select 常量值;
   注意：字符型和日期型的常量值必须用单引号引起来，数值型不需要
   */
   SELECT 100;
   SELECT 'john';
   ```

5. 查询函数

   ```sql
   -- select 函数名(实参列表);
   SELECT VERSION();
   ```

6. 查询表达式

   ```sql
   SELECT 100/1234;
   ```

7. 起别名

   有两种方式起别名：① as    ② 空格

   ```sql
   -- 方式一:
   SELECT 100%98 AS '结果';
   SELECT last_name AS '姓', first_name AS '名' FROM employees;
   -- 方式二:
   SELECT last_name '姓', first_name '名' FROM employees;
   
   -- 案例: 查询salary，显示结果为 out put [建议使用双引号]
   SELECT salary AS "out put" FROM employees;
   ```

8. 去重

   将 `distinct` 关键字放在要去重的字段前面。注意：**distinct 只配合单个字段使用**。

   ```sql
   -- select distinct 字段名 from 表名;
   -- 案例: 查询员工表中涉及到的所有部门编号
   SELECT DISTINCT department_id FROM employees;
   ```

9. +

   仅有一个功能：做加法运算

   ```sql
   -- select 数值+数值;  直接运算
   -- select 字符+数值;  只要其中一方为字符，先试图将字符转换成数值：如果转换成功，则继续运算；否则转换成0，再做运算
   -- select null+值;   只要有null，结果都为null
   ```

10. `concat` 函数

    功能：拼接字符

    ```sql
    -- select concat(字符1，字符2，字符3,...);
    
    -- 案例：查询 员工名和姓 连接成一个字段，并显示为 姓名
    -- 错误做法：
    SELECT 
    	last_name+first_name AS '姓名' 
    FROM employees;
    -- 正确做法：
    SELECT
    	CONCAT(last_name, first_name) AS '姓名'
    FROM employess;
    ```

11. `ifnull` 函数

    功能：判断某字段或表达式是否为 null，如果为 null，返回指定的值；否则返回原本的值。

    ```sql
    -- select ifnull(字段/表达式, 指定值) from 表;
    
    -- 案例：查询员工表中的名字和对应奖金率，如果没有就显示0
    SELECT
    	last_name AS '名字',
    	IFNULL(commission_pct, 0) AS '奖金率'
    FROM employees;
    ```

12. `isnull` 函数

    功能：判断某字段或表达式是否为 null，如果是，则返回 1，否则返回 0。注意要和 `ifnull` 函数区分清楚。



#### 1.4 练习

![1617811006863](./imgs/1617811006863.png)



### 2. 条件查询

#### 2.1 语法

```sql
select 查询列表
from 表名
where 筛选条件;
```

#### 2.2 筛选条件分类
##### （1）简单条件运算符

`> < = <> != >= <= <=>(安全等于)`

<img src="./imgs/1617811218259.png" alt="1617811218259" style="zoom:90%;" align="left" />

##### （2）逻辑运算符

- `&& / and`：一假即假
- `|| / or`：一真即真
- `! / not`：非真为假，非假为真

<img src="./imgs/1617811314253.png" alt="1617811314253" style="zoom:90%;" align="left" />

##### （3）模糊查询
- **like**：一般搭配通配符使用，可以判断字符型或数值型。
- **通配符**：`%` 任意多个字符，`_` 任意单个字符。

<img src="./imgs/1617811348464.png" alt="1617811348464" style="zoom:90%;" align="left" />

- **between and**

<img src="./imgs/1617811370252.png" alt="1617811370252" style="zoom:90%;" align="left" />

- **in**

<img src="./imgs/1617811381708.png" alt="1617811381708" style="zoom:90%;" align="left"/>

- **is null /is not null**：用于判断 null 值

<img src="./imgs/1617811397942.png" alt="1617811397942" style="zoom:90%;" align="left"/>



**is null**  VS  **<=>**：

|         | 普通类型的数值 | null值 | 可读性 |
| :-----: | :------------: | :----: | :----: |
| is null |       ×        |   √    |   √    |
|   <=>   |       √        |   √    |   ×    |



#### 2.3 练习

![1617811769340](./imgs/1617811769340.png)



### 3. 排序查询

#### 3.1 语法
```sql
select 查询列表
from 表
where 筛选条件
order by 排序列表 [asc/desc];
```

#### 3.2 特点
1. `asc` ：升序，如果不写默认升序
   `desc`：降序

2. 排序列表 支持 单个字段、多个字段、函数、表达式、别名

3. `order by` 的位置一般放在查询语句的最后（除 `limit` 语句之外）

#### 3.3 示例

![1617811903684](./imgs/1617811903684.png)

#### 3.4 练习

![1617811911885](./imgs/1617811911885.png)



### 4. 常见函数

#### 4.1 概述
- 概念：类似于 java 中的方法，将一组逻辑语句封装在方法体中，对外暴露方法名。

- 好处：提高重用性和隐藏实现细节。

- 调用：

  ```sql
  select 函数名(实参列表) 【from 表】;
  ```

- 分类：
  	1. 单行函数：如 `concat`、`length`、`ifnull` 等
   	2. 分组函数：功能：做统计使用，又称为统计函数、聚合函数、组函数

#### 4.2 单行函数
##### （1）字符函数
```
concat:  连接
substr:  截取子串
upper:   变大写
lower:   变小写
replace: 替换
length:  获取字节长度【汉字为3个字节】
trim:    去前后空格
lpad:    左填充
rpad:    右填充
instr:   获取子串第一次出现的索引
```

![1617812309983](./imgs/1617812309983.png)

##### （2）数学函数
```
ceil:     向上取整
round:    四舍五入
mod:      取模
floor:    向下取整
truncate: 截断
rand:     获取随机数，返回0-1之间的小数
```

<img src="./imgs/1617812319589.png" alt="1617812319589" align="left" />

##### 3、日期函数
> now：返回当前日期+时间
> year:返回年
> month：返回月
> day:返回日
> date_format:将日期转换成字符
> curdate:返回当前日期
> str_to_date:将字符转换成日期
> curtime：返回当前时间
> hour:小时
> minute:分钟
> second：秒
> datediff:返回两个日期相差的天数
> monthname:以英文形式返回月

![1617812329655](C:\Users\王嘉豪\AppData\Roaming\Typora\typora-user-images\1617812329655.png)

![1617812335449](C:\Users\王嘉豪\AppData\Roaming\Typora\typora-user-images\1617812335449.png)

![1617812352009](C:\Users\王嘉豪\AppData\Roaming\Typora\typora-user-images\1617812352009.png)

##### 4、其他函数

> version 当前数据库服务器的版本
> database 当前打开的数据库
> user当前用户
> password('字符')：返回该字符的密码形式
> md5('字符'):返回该字符的md5加密形式



##### 5、流程控制函数

①if

> if(条件表达式，表达式1，表达式2)：如果条件表达式成立，返回表达式1，否则返回表达式2

![1617812366720](C:\Users\王嘉豪\AppData\Roaming\Typora\typora-user-images\1617812366720.png)

②case情况1

> case 变量或表达式或字段
> when 常量1 then 值1
> when 常量2 then 值2
> ...
> else 值n
> end

![1617812376012](C:\Users\王嘉豪\AppData\Roaming\Typora\typora-user-images\1617812376012.png)

③case情况2

> case 
> when 条件1 then 值1
> when 条件2 then 值2
> ...
> else 值n
> end

![1617812387066](C:\Users\王嘉豪\AppData\Roaming\Typora\typora-user-images\1617812387066.png)

##### 习题：单行函数

![1617812397017](C:\Users\王嘉豪\AppData\Roaming\Typora\typora-user-images\1617812397017.png)



#### 4.3 分组函数
1、分类

> max 最大值
> min 最小值
> sum 和
> avg 平均值
> count 计算个数

2、特点

①语法
select max(字段) from 表名;

②支持的类型
sum和avg一般用于处理数值型
max、min、count可以处理任何数据类型

③以上分组函数都忽略null

④都可以搭配distinct使用，实现去重的统计
select sum(distinct 字段) from 表;

⑤count函数
count(字段)：统计该字段非空值的个数
count(*):统计结果集的行数

案例：查询每个部门的员工个数
1 xx    10
2 dd    20
3 mm    20
4 aa    40
5 hh    40

count(1):统计结果集的行数

效率上：
MyISAM存储引擎，count(*)最高

InnoDB存储引擎，count(*)和count(1)效率>count(字段)

⑥ 和分组函数一同查询的字段，要求是group by后出现的字段

![1617812409329](C:\Users\王嘉豪\AppData\Roaming\Typora\typora-user-images\1617812409329.png)

#### 4.4 练习

![1617812415960](C:\Users\王嘉豪\AppData\Roaming\Typora\typora-user-images\1617812415960.png)



## 6. 分组查询

### 6.1 语法
> select 分组函数，分组后的字段
> from 表
> 【where 筛选条件】
> group by 分组的字段
> 【having 分组后的筛选】
> 【order by 排序列表】

**注意：查询列表必须特殊，要求是分组函数和group by后出现的字段**

### 6.2 特点
1、分组查询中的筛选条件分为两类

|            | 使用关键字 | 筛选的表     | 位置            |
| ---------- | ---------- | ------------ | --------------- |
| 分组前筛选 | where      | 原始表       | group by的前面  |
| 分组后筛选 | having     | 分组后的结果 | group by 的后面 |

**①分组函数做条件肯定是放在having字句中**
②能用分组前筛选的，就优先考虑使用分组前筛选
	
2、group up子句支持单个字段分组、多个字段分组（多个字段之间用逗号隔开没有顺序要求），表达式或函数（用的较少）
3、也可以添加排序（排序放在整个分组查询的最后）

![1617857847781](C:\Users\王嘉豪\AppData\Roaming\Typora\typora-user-images\1617857847781.png)

![1617857857254](C:\Users\王嘉豪\AppData\Roaming\Typora\typora-user-images\1617857857254.png)



### 6.3 习题

![1617857864715](C:\Users\王嘉豪\AppData\Roaming\Typora\typora-user-images\1617857864715.png)



## 7. 连接查询

### 7.1 含义
> 当查询中涉及到了多个表的字段，需要使用多表连接
> select 字段1，字段2
> from 表1，表2,...;

> 笛卡尔乘积：当查询多个表时，没有添加有效的连接条件，导致多个表所有行实现完全连接
> 如何解决：添加有效的**连接条件**

### 7.2 分类

> 按年代分类：
> 	sql92：
> 		等值连接
> 		非等值连接
> 		自连接
>
> 也支持一部分外连接（用于oracle、sqlserver，mysql不支持）
>
> ​	sql99【推荐使用】
> ​		内连接
> ​			等值连接
> ​			非等值连接
> ​			自连接
> ​		外连接
> ​			左外连接
> ​			右外连接
> ​			全外连接（mysql不支持）
> ​		交叉连接





### 7.3 SQL92语法
#### 1、等值连接

> 语法：
> 	select 查询列表
> 	from 表1 别名,表2 别名
> 	where 表1.key=表2.key
> 	【and 筛选条件】
> 	【group by 分组字段】
> 	【having 分组后的筛选】
> 	【order by 排序字段】

> 特点：
> 	① 一般为表起别名
> 	②多表的顺序可以调换
> 	③n表连接至少需要n-1个连接条件
> 	④等值连接的结果是多表的交集部分

![1617858297989](C:\Users\王嘉豪\AppData\Roaming\Typora\typora-user-images\1617858297989.png)

#### 2、非等值连接

> 语法：
> 	select 查询列表
> 	from 表1 别名,表2 别名
> 	where 非等值的连接条件
> 	【and 筛选条件】
> 	【group by 分组字段】
> 	【having 分组后的筛选】
> 	【order by 排序字段】

![1617858316727](C:\Users\王嘉豪\AppData\Roaming\Typora\typora-user-images\1617858316727.png)

![1617858329068](C:\Users\王嘉豪\AppData\Roaming\Typora\typora-user-images\1617858329068.png)

![1617858335978](C:\Users\王嘉豪\AppData\Roaming\Typora\typora-user-images\1617858335978.png)

#### 3、自连接

> 语法：
> 	select 查询列表
> 	from 表 别名1,表 别名2
> 	where 等值的连接条件
> 	【and 筛选条件】
> 	【group by 分组字段】
> 	【having 分组后的筛选】
> 	【order by 排序字段】

![1617858362733](C:\Users\王嘉豪\AppData\Roaming\Typora\typora-user-images\1617858362733.png)

![1617858371879](C:\Users\王嘉豪\AppData\Roaming\Typora\typora-user-images\1617858371879.png)

#### 习题

![1617858379880](C:\Users\王嘉豪\AppData\Roaming\Typora\typora-user-images\1617858379880.png)

### 7.4 SQL99语法

> 语法：
> 	select 查询列表
> 	from 表1 别名 【连接类型】
> 	join 表2 别名 
> 	on 连接条件
> 	【where 筛选条件】
> 	【group by 分组】
> 	【having 筛选条件】
> 	【order by 排序列表】
> 	【limit】

> 分类：
> 内连接（★）: inner
> 外连接
> 	左外（★）: left 【outer】
> 	右外（★）: right 【outer】
> 	全外: full 【outer】
> 交叉连接: cross
>

####  1、内连接

> 语法：
> select 查询列表
> from 表1 别名
> 【inner】 join 表2 别名 
> on 连接条件
> where 筛选条件
> group by 分组列表
> having 分组后的筛选
> order by 排序列表
> limit 子句;

> 特点：
> ①表的顺序可以调换
> ②内连接的结果=多表的交集
> ③n表连接至少需要n-1个连接条件
> ④可添加排序、分组、筛选
> ⑤inner可以省略
> ⑥筛选条件放在where后面，连接条件放在on后面，提高分离性，便于阅读
> ⑦inner join连接和sql92语法中的等值连接效果是一样的，都是查询多表的交集

> 分类：
> 等值连接
> 非等值连接
> 自连接

**等值连接**

![1617858593352](C:\Users\王嘉豪\AppData\Roaming\Typora\typora-user-images\1617858593352.png)

**非等值连接**

![1617858614312](C:\Users\王嘉豪\AppData\Roaming\Typora\typora-user-images\1617858614312.png)

**自连接**

![1617858625766](C:\Users\王嘉豪\AppData\Roaming\Typora\typora-user-images\1617858625766.png)

#### 2、外连接
> 应用场景：用于查询一个表中有，另一个表没有的记录

> ?语法：
> select 查询列表
> from 表1 别名
> left|right|full【outer】 join 表2 别名 on 连接条件
> where 筛选条件
> group by 分组列表
> having 分组后的筛选
> order by 排序列表
> limit 子句;

> 特点：
> ①查询的结果=主表中所有的行，如果从表和它匹配的将显示匹配行，如果从表没有匹配的则显示null
> 		【即：外连接查询结果=内连接结果+主表中需要匹配但从表没有的记录】
> ②left join 左边的就是主表，right join 右边的就是主表
>   full join 两边都是主表
> 		【左外和右外交换两个表的顺序，可以实现同样的效果】
> ③一般用于查询除了交集部分的剩余的不匹配的行
> ④全外连接=内连接的结果+表1中需要匹配但表2没有的记录+表2中需要匹配但表1没有的记录	
>  【full outer join】

![1617858668265](C:\Users\王嘉豪\AppData\Roaming\Typora\typora-user-images\1617858668265.png)



#### 3、交叉连接

> 语法：
> select 查询列表
> from 表1 别名
> cross join 表2 别名;

> 特点：
> 类似于笛卡尔乘积  【m×n】



**图示**

![1617858705422](C:\Users\王嘉豪\AppData\Roaming\Typora\typora-user-images\1617858705422.png)

![1617858711179](C:\Users\王嘉豪\AppData\Roaming\Typora\typora-user-images\1617858711179.png)



**习题**

![1617858717884](C:\Users\王嘉豪\AppData\Roaming\Typora\typora-user-images\1617858717884.png)



## 8. 子查询

### 8.1 含义
> 嵌套在其他语句内部的select语句称为子查询或内查询，
> 外面的语句可以是insert、update、delete、select等，一般select作为外面语句较多
> 外面如果为select语句，则此语句称为外查询或主查询

### 8.2 分类
> **1、按出现位置**
> select后面：
> 		仅仅支持标量子查询
> from后面：
> 		表子查询
> **where或having后面：**
> 		**标量子查询**
> 		**列子查询**
> 		行子查询
> exists后面：
> 		标量子查询
> 		列子查询
> 		行子查询
> 		表子查询

> **2、按结果集的行列**
> 标量子查询（单行子查询）：结果集为一行一列
> 列子查询（多行子查询）：结果集为多行一列
> 行子查询：结果集为多行多列
> 表子查询：结果集为多行多列

### 8.3 示例
#### where或having后面

> 1、标量子查询（单行子查询）
> 2、列子查询（多行子查询）
> 3、行子查询（一行多列）

**特点：**

> ①子查询放在小括号内
> ②子查询一般放在条件的右侧
> ③标量子查询，一般搭配着单行操作符使用
>
> \> < >= <= = <>
>
> 列子查询，一般搭配着多行操作符使用
> in、any/some、all
>
> ④子查询的执行优先于主查询执行，主查询的条件用到了子查询的结果

##### 1、标量子查询
案例：查询最低工资的员工姓名和工资
①最低工资

```
select min(salary) from employees
```

②查询员工的姓名和工资，要求工资=①

```
select last_name,salary
from employees
where salary=(
	select min(salary) from employees
);
```

![1617887231027](C:\Users\王嘉豪\AppData\Roaming\Typora\typora-user-images\1617887231027.png)

![1617887291174](C:\Users\王嘉豪\AppData\Roaming\Typora\typora-user-images\1617887291174.png)

##### 2、列子查询
案例：查询所有是领导的员工姓名
①查询所有员工的 manager_id

```
select manager_id
from employees
```

②查询姓名，employee_id属于①列表的一个

```
select last_name
from employees
where employee_id in(
	select manager_id
	from employees
);
```

![1617887305981](C:\Users\王嘉豪\AppData\Roaming\Typora\typora-user-images\1617887305981.png)

##### 3、行子查询

![1617887335389](C:\Users\王嘉豪\AppData\Roaming\Typora\typora-user-images\1617887335389.png)

#### select后面

![1617887348440](C:\Users\王嘉豪\AppData\Roaming\Typora\typora-user-images\1617887348440.png)

#### from后面

![1617887356219](C:\Users\王嘉豪\AppData\Roaming\Typora\typora-user-images\1617887356219.png)

#### exists后面

![1617887364499](C:\Users\王嘉豪\AppData\Roaming\Typora\typora-user-images\1617887364499.png)

### 8.4 习题

![1617887389394](C:\Users\王嘉豪\AppData\Roaming\Typora\typora-user-images\1617887389394.png)

![1617887399688](C:\Users\王嘉豪\AppData\Roaming\Typora\typora-user-images\1617887399688.png)

### 经典案例

![1617887412148](C:\Users\王嘉豪\AppData\Roaming\Typora\typora-user-images\1617887412148.png)

![1617887431617](C:\Users\王嘉豪\AppData\Roaming\Typora\typora-user-images\1617887431617.png)

![1617887439634](C:\Users\王嘉豪\AppData\Roaming\Typora\typora-user-images\1617887439634.png)

![1617887455477](C:\Users\王嘉豪\AppData\Roaming\Typora\typora-user-images\1617887455477.png)



## 9. 分页查询

### 9.1 应用场景
> 当要查询的条目数太多，一页显示不全

### 9.2 语法

> select 查询列表
> from 表
> limit 【offset，】size;

> 注意：
> offset代表的是起始的条目索引，默认从0开始
> size代表的是显示的条目数

> 公式：
> 假如要显示的页数为page，每一页条目数为size
> select 查询列表
> from 表
> **limit (page-1)*size,size;**

![1617887562858](C:\Users\王嘉豪\AppData\Roaming\Typora\typora-user-images\1617887562858.png)



## 10. 联合查询

### 10.1 含义
> union：合并、联合，将多次查询结果合并成一个结果

### 10.2 语法
> 查询语句1
> union 【all】
> 查询语句2
> union 【all】
> ...

### 10.3 意义
> 1、将一条比较复杂的查询语句拆分成多条语句
> 2、适用于查询多个表的时候，查询的列基本是一致

### 10.4 特点
> 1、要求多条查询语句的查询列数必须一致
> 2、要求多条查询语句的查询的各列类型、顺序最好一致
> 3、union 去重，union all包含重复项



## 查询总结

### 语法
> select 查询列表     ⑦
> from 表1 别名       ①
> 连接类型 join 表2   ②
> on 连接条件         ③
> where 筛选          ④
> group by 分组列表   ⑤
> having 筛选         ⑥
> order by排序列表    ⑧
> limit 起始条目索引，条目数;  ⑨

### 习题

![1617887762393](C:\Users\王嘉豪\AppData\Roaming\Typora\typora-user-images\1617887762393.png)

![1617887768304](C:\Users\王嘉豪\AppData\Roaming\Typora\typora-user-images\1617887768304.png)



# **三、DML语言（数据操作语言：增删改）**

## 1. 插入

### 1.1 方式一
> 语法：
> insert into 表名(字段名,...) values(值,...);

> 特点：
> 1、要求值的类型和字段的类型要一致或兼容
> 2、字段的个数和顺序不一定与原始表中的字段个数和顺序一致
> 但必须保证值和字段一一对应
> 3、假如表中有可以为null的字段，注意可以通过以下两种方式插入null值
> ①字段和值都省略
> ②字段写上，值使用null
> 4、字段和值的个数必须一致
> 5、字段名可以省略，默认所有列

### 1.2 方式二
> 语法：
> insert into 表名 set 字段=值,字段=值,...;



**两种方式的区别：**

> 1.方式一支持一次插入多行，语法如下：
> insert into 表名【(字段名,..)】 values(值，..),(值，...),...;
> 2.方式一支持子查询，语法如下：
> insert into 表名
> 查询语句;

![1617887984174](C:\Users\王嘉豪\AppData\Roaming\Typora\typora-user-images\1617887984174.png)



## 2. 修改

### 2.1 修改单表的记录 
> 语法：update 表名 set 字段=值,字段=值 【where 筛选条件】;

### 2.2 修改多表的记录【补充】
> 语法：
> update 表1 别名 
> left|right|inner join 表2 别名 
> on 连接条件  
> set 字段=值,字段=值 
> 【where 筛选条件】;

![1617888144351](C:\Users\王嘉豪\AppData\Roaming\Typora\typora-user-images\1617888144351.png)



## 3. 删除

### 方式一：使用delete
#### 1、删除单表的记录
> 语法：delete from 表名 【where 筛选条件】【limit 条目数】

#### 2、级联删除[补充]
> 语法：
> delete 别名1,别名2 from 表1 别名 
> inner|left|right join 表2 别名 
> on 连接条件
>  【where 筛选条件】

### 方式二：使用truncate
> 语法：truncate table 表名



**两种方式的区别【面试题】**

> 1.truncate删除后，如果再插入，标识列从1开始
>    delete删除后，如果再插入，标识列从断点开始
> 2.delete可以添加筛选条件
>     truncate不可以添加筛选条件
> 3.truncate效率较高
> 4.truncate没有返回值
> delete可以返回受影响的行数
> 5.truncate不可以回滚
>    delete可以回滚

![1617888285853](C:\Users\王嘉豪\AppData\Roaming\Typora\typora-user-images\1617888285853.png)

## 4. 习题

![1617888317208](C:\Users\王嘉豪\AppData\Roaming\Typora\typora-user-images\1617888317208.png)



# **四、DDL语言（数据定义语言）**

## 1. 库的管理

### 1.1 创建库
> create database 【if not exists】 库名【 character set 字符集名】;

### 1.2 修改库
> alter database 库名 character set 字符集名;

### 1.3 删除库
> drop database 【if exists】 库名;

![1617888423874](C:\Users\王嘉豪\AppData\Roaming\Typora\typora-user-images\1617888423874.png)



## 2. 表的管理

### 2.1 创建表 
> create table 【if not exists】 表名(
> 	字段名 字段类型 【约束】,
> 	字段名 字段类型 【约束】,
> 	。。。
> 	字段名 字段类型 【约束】 
>
> )

![1617888496318](C:\Users\王嘉豪\AppData\Roaming\Typora\typora-user-images\1617888496318.png)

### 2.2 修改表

#### 1、添加列
> alter table 表名 add column 列名 类型 【first|after 字段名】;

#### 2、修改列的类型或约束
> alter table 表名 modify column 列名 新类型 【新约束】;

#### 3、修改列名
> alter table 表名 change column 旧列名 新列名 类型;

#### 4 、删除列
> alter table 表名 drop column 列名;

#### 5、修改表名
> alter table 表名 rename 【to】 新表名;

![1617888540872](C:\Users\王嘉豪\AppData\Roaming\Typora\typora-user-images\1617888540872.png)



### 2.3 删除表
> drop table【if exists】 表名;

![1617888567893](C:\Users\王嘉豪\AppData\Roaming\Typora\typora-user-images\1617888567893.png)

### 2.4 复制表
#### 1、复制表的结构
> create table 表名 like 旧表;

#### 2、 复制表的结构+数据
> create table 表名 
> select 查询列表 from 旧表【where 筛选】;

![1617888614002](C:\Users\王嘉豪\AppData\Roaming\Typora\typora-user-images\1617888614002.png)

### 2.5 习题

![1617888703001](C:\Users\王嘉豪\AppData\Roaming\Typora\typora-user-images\1617888703001.png)



## 3. 数据类型

### 3.1 数值型
#### 1、整型
> tinyint、smallint、mediumint、int/integer、bigint
> 1              2                3                     4                     8

> 特点：
> ①都可以设置无符号和有符号，默认有符号，通过unsigned设置无符号
> ②如果超出了范围，会报out or range异常，插入临界值
> ③长度可以不指定，默认会有一个长度
>    长度代表显示的最大宽度，如果不够则左边用0填充，但需要搭配zerofill，并且默认变为无符号整型

#### 2、浮点型
> 定点数：decimal(M,D) / dec(M,D)
> 浮点数:
> 	float(M,D)   4
> 	double(M,D)  8

> 特点：
> ①M代表整数部位+小数部位的个数，D代表小数部位
> ②如果超出范围，则报out or range异常，并且插入临界值
> ③M和D都可以省略，但对于定点数，M默认为10，D默认为0;如果是float和double，则会根据插入的数值的精度来决定精度
> ④如果精度要求较高，则优先考虑使用定点数

> 原则：
> 所选择的类型越简单越好，能保存数值的类型越小越好

### 3.2 字符型
> char、varchar、binary、varbinary、enum、set、text、blob

> char：固定长度的字符，写法为char(M)，最大长度不能超过M，其中M可以省略，默认为1
> varchar：可变长度的字符，写法为varchar(M)，最大长度不能超过M，其中M不可以省略

![1617888899512](C:\Users\王嘉豪\AppData\Roaming\Typora\typora-user-images\1617888899512.png)

### 3.3 日期型
> year年
> date日期
> time时间
> datetime 日期+时间            8      
> timestamp 日期+时间         4   比较容易受时区、语法模式、版本的影响，更能反映当前时区的真实时间

![1617888917912](C:\Users\王嘉豪\AppData\Roaming\Typora\typora-user-images\1617888917912.png)



## 4.常见的约束

### 4.1 常见的约束
> NOT NULL：非空，该字段的值必填
> UNIQUE：唯一，该字段的值不可重复
> DEFAULT：默认，该字段的值不用手动插入有默认值
> CHECK：检查，mysql不支持
> PRIMARY KEY：主键，该字段的值不可重复并且非空  unique+not null
> FOREIGN KEY：外键，该字段的值引用了另外的表的字段

**主键和唯一**

> 1、区别：
> ①、一个表至多有一个主键，但可以有多个唯一
> ②、主键不允许为空，唯一可以为空
> 2、相同点
> 都具有唯一性
> 都支持组合键，但不推荐

**外键**

> 1、用于限制两个表的关系，从表的字段值引用了主表的某字段值
> 2、外键列和主表的被引用列要求类型一致，意义一样，名称无要求
> 3、主表的被引用列要求是一个key（一般就是主键）
> 4、插入数据，先插入主表
>       删除数据，先删除从表

**可以通过以下两种方式来删除主表的记录**

> #方式一：**级联删除**
> ALTER TABLE stuinfo ADD CONSTRAINT fk_stu_major FOREIGN KEY(majorid) REFERENCES major(id) ON DELETE CASCADE;
>
> #方式二：**级联置空**
> ALTER TABLE stuinfo ADD CONSTRAINT fk_stu_major FOREIGN KEY(majorid) REFERENCES major(id) ON DELETE SET NULL;

![1617889095595](C:\Users\王嘉豪\AppData\Roaming\Typora\typora-user-images\1617889095595.png)

### 4.2 创建表时添加约束
> create table 表名(
> 	字段名 字段类型 not null,#非空
> 	字段名 字段类型 primary key,#主键
> 	字段名 字段类型 unique,#唯一
> 	字段名 字段类型 default 值,#默认
> 	constraint 约束名 foreign key(字段名) references 主表（被引用列）
>
> )

**注意：**

|          | 支持类型       | 可以起约束名       |
| -------- | -------------- | ------------------ |
| 列级约束 | 除了外键       | 不可以             |
| 表级约束 | 除了非空和默认 | 可以，但对主键无效 |

**列级约束可以在一个字段上追加多个，中间用空格隔开，没有顺序要求**

![1617889216474](C:\Users\王嘉豪\AppData\Roaming\Typora\typora-user-images\1617889216474.png)

![1617889222799](C:\Users\王嘉豪\AppData\Roaming\Typora\typora-user-images\1617889222799.png)

### 4.3 修改表时添加或删除约束
#### 1、非空
> 添加非空
> alter table 表名 modify column 字段名 字段类型 not null;
> 删除非空
> alter table 表名 modify column 字段名 字段类型 ;

#### 2、默认
> 添加默认
> alter table 表名 modify column 字段名 字段类型 default 值;
> 删除默认
> alter table 表名 modify column 字段名 字段类型 ;

#### 3、主键
> 添加主键
> alter table 表名 add【 constraint 约束名】 primary key(字段名);
> 删除主键
> alter table 表名 drop primary key;

#### 4、唯一
> 添加唯一
> alter table 表名 add【 constraint 约束名】 unique(字段名);
> 删除唯一
> alter table 表名 drop index 索引名;

#### 5、外键
> 添加外键
> alter table 表名 add【 constraint 约束名】 foreign key(字段名) references 主表（被引用列）;
> 删除外键
> alter table 表名 drop foreign key 约束名;

![1617889286439](C:\Users\王嘉豪\AppData\Roaming\Typora\typora-user-images\1617889286439.png)

### 4.4 自增长列
> 特点：
> 1、不用手动插入值，可以自动提供序列值，默认从1开始，步长为1
> auto_increment_increment
> 如果要更改起始值：手动插入值
> 如果要更改步长：更改系统变量
> set auto_increment_increment=值;
> 2、一个表至多有一个自增长列
> 3、自增长列只能支持数值型
> 4、自增长列必须为一个key

#### 1、创建表时设置自增长列
> create table 表(
> 	字段名 字段类型 约束 auto_increment
> )

#### 2、修改表时设置自增长列
> alter table 表 modify column 字段名 字段类型 约束 auto_increment

#### 3、删除自增长列
> alter table 表 modify column 字段名 字段类型 约束 

![1617889359975](C:\Users\王嘉豪\AppData\Roaming\Typora\typora-user-images\1617889359975.png)



### 4.4 习题

![1617889382069](C:\Users\王嘉豪\AppData\Roaming\Typora\typora-user-images\1617889382069.png)



# **五、TCL语言（事务控制语言）**

## 1. 事务

### 1.1 含义
> 事务：一条或多条sql语句组成一个执行单位，一组sql语句要么都执行要么都不执行

### 1.2 特点（ACID）
> A 原子性：一个事务是不可再分割的整体，要么都执行要么都不执行
> C 一致性：一个事务可以使数据从一个一致状态切换到另外一个一致的状态
>  I  隔离性：一个事务不受其他事务的干扰，多个事务互相隔离的
> D 持久性：一个事务一旦提交了，则永久的持久化到本地

### 1.3 事务的使用步骤 ★
> 了解：
> 隐式（自动）事务：没有明显的开启和结束，本身就是一条事务可以自动提交，比如insert、update、delete
> **显式事务：**具有明显的开启和结束

使用显式事务：
#### ①开启事务
> set autocommit=0;
> start transaction;#可以省略

#### ②编写一组逻辑sql语句
> 注意：sql语句支持的是insert、update、delete

> 设置回滚点：
> savepoint 回滚点名;

#### ③结束事务
> 提交：commit;
> 回滚：rollback;
> 回滚到指定的地方：rollback to 回滚点名;

### 1.4 并发事务
> **1、事务的并发问题是如何发生的？**
> 多个事务 同时 操作 同一个数据库的相同数据时

> **2、并发问题都有哪些？**
> **脏读：**一个事务读取了其他事务还没有提交的数据，读到的是其他事务“更新”的数据
> 不可重复读：一个事务多次读取，结果不一样
> **幻读：**一个事务读取了其他事务还没有提交的数据，只是读到的是 其他事务“插入”的数据

> **3、如何解决并发问题**
> 通过设置隔离级别来解决并发问题

**4、隔离级别**

|                           | 脏读 | 不可重复读 | 幻读 |
| ------------------------- | ---- | ---------- | ---- |
| read uncommitted:读未提交 | ×    | ×          | ×    |
| read committed：读已提交  | √    | ×          | ×    |
| repeatable read：可重复读 | √    | √          | ×    |
| serializable：串行化      | √    | √          | √    |

![1617889765439](C:\Users\王嘉豪\AppData\Roaming\Typora\typora-user-images\1617889765439.png)

![1617889778379](C:\Users\王嘉豪\AppData\Roaming\Typora\typora-user-images\1617889778379.png)

![1617889795576](C:\Users\王嘉豪\AppData\Roaming\Typora\typora-user-images\1617889795576.png)

**mysql 8中查看当前隔离级别：select @@transaction_isolation;**

![1617889812437](C:\Users\王嘉豪\AppData\Roaming\Typora\typora-user-images\1617889812437.png)



# **六、其他**

## 1. 视图

### 1.1含义
> mysql5.1版本出现的新特性，本身是一个虚拟表，它的数据来自于表，通过执行时动态生成。
> 好处：
> 1、简化sql语句
> 2、提高了sql的重用性
> 3、保护基表的数据，提高了安全性

![1617889943050](C:\Users\王嘉豪\AppData\Roaming\Typora\typora-user-images\1617889943050.png)

### 1.2 创建

> create view 视图名
> as
> 查询语句;

![1617889954984](C:\Users\王嘉豪\AppData\Roaming\Typora\typora-user-images\1617889954984.png)

### 1.3 修改

> **方式一：**
> create or replace view 视图名
> as
> 查询语句;
> **方式二：**
> alter view 视图名
> as
> 查询语句

![1617889983270](C:\Users\王嘉豪\AppData\Roaming\Typora\typora-user-images\1617889983270.png)

### 1.4 删除
> drop view 视图1，视图2,...;

### 1.5 查看
> desc 视图名;
> show create view 视图名;

### 1.6 使用
> 1.插入
> insert
> 2.修改
> update
> 3.删除
> delete
> 4.查看
> select

> 注意：视图一般用于查询的，而不是更新的，所以具备以下特点的视图都不允许更新
> ①包含分组函数、group by、distinct、having、union、
> ②join
> ③常量视图
> ④where后的子查询用到了from中的表
> ⑤用到了不可更新的视图

![1617890013115](C:\Users\王嘉豪\AppData\Roaming\Typora\typora-user-images\1617890013115.png)

![1617890028040](C:\Users\王嘉豪\AppData\Roaming\Typora\typora-user-images\1617890028040.png)

### 1.7 视图和表的对比

|      | 关键字 | 是否占用物理空间        | 使用         |
| ---- | ------ | ----------------------- | ------------ |
| 视图 | view   | 占用较小，只保存sql逻辑 | 一般用于查询 |
| 表   | table  | 保存实际的数据          | 增删改查     |



## 2. 变量

![1617890113759](C:\Users\王嘉豪\AppData\Roaming\Typora\typora-user-images\1617890113759.png)

### 2.1 系统变量
> 说明：变量由系统提供的，不用自定义

**语法：**

> ①查看系统变量
> show 【global|session 】variables like ''; 如果没有显式声明global还是session，则默认是session

> ②查看指定的系统变量的值
> select @@【global|session】.变量名; 如果没有显式声明global还是session，则默认是session

> ③为系统变量赋值
> 方式一：
> set 【global|session 】 变量名=值; 如果没有显式声明global还是session，则默认是session
> 方式二：
> set @@global.变量名=值;
> set @@变量名=值；

![1617890226986](C:\Users\王嘉豪\AppData\Roaming\Typora\typora-user-images\1617890226986.png)



#### 1、全局变量
服务器层面上的，必须拥有super权限才能为系统变量赋值，作用域为整个服务器，也就是针对于所有连接（会话）有效

#### 2、会话变量
服务器为每一个连接的客户端都提供了系统变量，作用域为当前的连接（会话）

![1617890233592](C:\Users\王嘉豪\AppData\Roaming\Typora\typora-user-images\1617890233592.png)



### 2.2 自定义变量

**说明：**

![1617890253388](C:\Users\王嘉豪\AppData\Roaming\Typora\typora-user-images\1617890253388.png)

#### 1、用户变量
> 作用域：针对于当前连接（会话）生效

> 位置：begin end里面，也可以放在外面

**使用：**

> ①声明并赋值：
> set @变量名=值;或
> set @变量名:=值;或
> select @变量名:=值;

> ②更新值
> 方式一：
> 	set @变量名=值;或
> 	set @变量名:=值;或
> 	select @变量名:=值;
> 方式二：
> 	select xx into @变量名 from 表;

> ③使用
> select @变量名;

![1617890315724](C:\Users\王嘉豪\AppData\Roaming\Typora\typora-user-images\1617890315724.png)

#### 2、局部变量
> 作用域：仅仅在定义它的begin end中有效

> 位置：只能放在begin end中，而且只能放在第一句

**使用：**

> ①声明
> declare 变量名 类型 【default 值】;
> ②赋值或更新
> 方式一：
> 	set 变量名=值;或
> 	set 变量名:=值;或
> 	select @变量名:=值;
> 方式二：
> 	select xx into 变量名 from 表;
> ③使用
> select 变量名;

![1617890339547](C:\Users\王嘉豪\AppData\Roaming\Typora\typora-user-images\1617890339547.png)

![1617890345770](C:\Users\王嘉豪\AppData\Roaming\Typora\typora-user-images\1617890345770.png)



## 3. 存储过程和函数

> 说明：都类似于java中的方法，将一组完成特定功能的逻辑语句包装起来，对外暴露名字
> 好处：
> 1、提高重用性
> 2、sql语句简单
> 3、减少了和数据库服务器连接的次数，提高了效率

### 3.1 存储过程

#### 1、创建 
> create procedure 存储过程名(参数模式 参数名 参数类型)
> begin
> 		存储过程体
> end

> 注意：
> 1.参数模式：in、out、inout，其中in可以省略
> 2.存储过程体的每一条sql语句都需要用分号结尾

#### 2、调用
> call 存储过程名(实参列表)

> 举例：
> 调用in模式的参数：call sp1（‘值’）;
> 调用out模式的参数：set @name; call sp1(@name);select @name;
> 调用inout模式的参数：set @name=值; call sp1(@name); select @name;

#### 3、查看
> show create procedure 存储过程名;

#### 4、删除
> drop procedure 存储过程名;



![1617890564280](C:\Users\王嘉豪\AppData\Roaming\Typora\typora-user-images\1617890564280.png)

![1617890570322](C:\Users\王嘉豪\AppData\Roaming\Typora\typora-user-images\1617890570322.png)

![1617890578843](C:\Users\王嘉豪\AppData\Roaming\Typora\typora-user-images\1617890578843.png)



#### 习题

![1617890591197](C:\Users\王嘉豪\AppData\Roaming\Typora\typora-user-images\1617890591197.png)



### 3.2 函数

#### 1、创建
> create function 函数名(参数名 参数类型) returns  返回类型
> begin
> 	函数体
> end

> 注意：函数体中肯定需要有return语句

#### 2、调用
> select 函数名(实参列表);

#### 3、查看
> show create function 函数名;

#### 4、删除
> drop function 函数名；

![1617890652533](C:\Users\王嘉豪\AppData\Roaming\Typora\typora-user-images\1617890652533.png)

![1617890658481](C:\Users\王嘉豪\AppData\Roaming\Typora\typora-user-images\1617890658481.png)



## 4.流程控制结构

> 说明：
> 顺序结构：程序从上往下依次执行
> 分支结构：程序按条件进行选择执行，从两条或多条路径中选择一条执行
> 循环结构：程序满足一定条件下，重复执行一组语句

### 4.1 分支结构

**特点：**

> **1、if函数**
> 功能：实现简单双分支
> 语法：
> if(条件，值1，值2)
> 位置：
> 可以作为表达式放在任何位置

> **2、case结构**
> 功能：实现多分支
> 语法1：
> case 表达式或字段
> when 值1 then 语句1;
> when 值2 then 语句2；
> ..
> else 语句n;
> end [case];
>
> 
>
> 语法2：
> case 
> when 条件1 then 语句1;
> when 条件2 then 语句2；
> ..
> else 语句n;
> end [case];
>
> 位置：
>
> 可以放在任何位置，
> 如果放在begin end 外面，作为表达式结合着其他语句使用
> 如果放在begin end 里面，一般作为独立的语句使用

> **3、if结构**
> 功能：实现多分支
> 语法：
> if 条件1 then 语句1;
> elseif 条件2 then 语句2;
> ...
> else 语句n;
> end if;
> 位置：
> 只能放在begin end中

![1617890804877](C:\Users\王嘉豪\AppData\Roaming\Typora\typora-user-images\1617890804877.png)

![1617890810128](C:\Users\王嘉豪\AppData\Roaming\Typora\typora-user-images\1617890810128.png)



### 4.2 循环结构

**位置：**

> 只能放在begin end中

**特点：**

> 都能实现循环结构

**对比：**

> ①这三种循环都可以省略名称，但如果循环中添加了循环控制语句（leave或iterate）则必须添加名称
> ②loop 一般用于实现简单的死循环
>     while 先判断后执行
>     repeat 先执行后判断，无条件至少执行一次



#### 1、while
> 语法：
> 【名称:】while 循环条件 do
> 		循环体
> end while 【名称】;

#### 2、loop
> 语法：
> 【名称：】loop
> 		循环体
> end loop 【名称】;

#### 3、repeat
> 语法：
> 【名称:】repeat
> 		循环体
> until 结束条件 
> end repeat 【名称】;



#### 循环控制语句
> leave：类似于break，用于跳出所在的循环
> iterate：类似于continue，用于结束本次循环，继续下一次



![1617890957231](C:\Users\王嘉豪\AppData\Roaming\Typora\typora-user-images\1617890957231.png)

![1617890974257](C:\Users\王嘉豪\AppData\Roaming\Typora\typora-user-images\1617890974257.png)



## 习题

![1617891000902](C:\Users\王嘉豪\AppData\Roaming\Typora\typora-user-images\1617891000902.png)