# 安装

## 

## 常见问题

### 跳过密码验证

1、过滤初始密码

```
grep 'password'  /var/log/mysqld.log
```

红色框框里的就是初始密码![在这里插入图片描述](https://img-blog.csdnimg.cn/20200710152157451.png)

如果密码已经改过了，那么即使找到默认密码也是没有用的，此时就要看第二招了

2、跳过密码认证

```
 vim /etc/my.cnf
```

```
[mysqld]
skip-grant-tables      //指定位置加一行
```

改了配置文件，记得重启服务

```
systemctl restart mysqld       
mysql      //进入到mysql 
mysql> update mysql.user set authentication_string=password('ZG..2020') where user='root';     //更新密码为ZG..2020
mysql> exit
```

消除跳过密码认证，进入正常mysql

### 更改允许登录的host

1. 改表法

   可能是你的帐号不允许从远程登陆，只能在localhost。这个时候只要在localhost的那台电脑，登入mysql后，更改 "mysql" 数据库里的 "user" 表里的 "host" 项，从"localhost"改称"%"

   ```
   mysql>use mysql;
   
   mysql>update user set host = '%' where user = 'root';
   
   mysql>select host, user from user;
   
   mysql>FLUSH PRIVILEGES;
   ```

   这个方式可以成功



​	2.权限授予

​	测试，这个方法暂时不行，使用方法1

```
创建用户'root'@'172.17.0.1'
create user 'root'@'172.17.0.1';
授权'root'@'172.17.0.1'用户拥有所有数据库的所有权限
GRANT ALL PRIVILEGES ON *.* TO 'root'@'172.17.0.1' WITH GRANT OPTION;

update user set host = '127.0.0.1' where user = 'root' and host = '%';

报错，没有grant权限，这里是因为没有创建用户
#GRANT ALL PRIVILEGES ON *.* TO 'root'@'172.17.0.1';
ERROR 1410 (42000): You are not allowed to create a user with GRANT
```

```
FLUSH PRIVILEGES;
```

```
GRANT USAGE ON *.* TO 'root'@'172.17.0.1';
```

### 生成操作日志

- 操作记录

```
首先进入mysql输入指令

show variables like 'gen%';

general_log是开启还是关闭状态，以及这个帐号的general_log文件在哪
如果没有开启，请先设置开启

set global general_log=ON;

查看log：
cat /目录/日志.log
```

- 数据库自己的操作记录

```
show variables like '%log_output%';
默认是FILE，改为TABLE

set  global log_output='TABLE';
1
之后就可以通过以下两句话查看数据库操作记录
查看操作记录：

select * from mysql.general_log;
1
会看到在数据库里已经记录上了日志

因数据库一直记录日志会增加压力，建议用文件记录

set  global log_output='FILE';

truncate table mysql.general_log;
```



# DQL(data query language)

## 多表查询

![1.jpg](https://pic.leetcode-cn.com/ad3df1c4ecc7d2dbe85f92cdde8ec9a731fdd20dc4c5629ecb372b21de26c682-1.jpg)

多表的联结又分为以下几种类型：

1）左联结（left join），联结结果保留左表的全部数据

2）右联结（right join），联结结果保留右表的全部数据

3）内联结（inner join），取两表的公共数据

## 实例

### 	[175. 组合两个表](https://leetcode-cn.com/problems/combine-two-tables/)

表1: `Person`

```
+-------------+---------+
| 列名         | 类型     |
+-------------+---------+
| PersonId    | int     |
| FirstName   | varchar |
| LastName    | varchar |
+-------------+---------+
PersonId 是上表主键
```

表2: `Address`

```
+-------------+---------+
| 列名         | 类型    |
+-------------+---------+
| AddressId   | int     |
| PersonId    | int     |
| City        | varchar |
| State       | varchar |
+-------------+---------+
AddressId 是上表主键
```

编写一个 SQL 查询，满足条件：无论 person 是否有地址信息，都需要基于上述两表提供 person 的以下信息：

```
FirstName, LastName, City, State
```

代码：

```mysql
SELECT FirstName, LastName, City, State
FROM Person p LEFT JOIN Address a
ON p.PersonId = a.PersonId;
```

### 	[176. 第二高的薪水](https://leetcode-cn.com/problems/second-highest-salary/)

`Employee` 表：

```
+-------------+------+
| Column Name | Type |
+-------------+------+
| id          | int  |
| salary      | int  |
+-------------+------+
id 是这个表的主键。
表的每一行包含员工的工资信息。
```

编写一个 SQL 查询，获取并返回 `Employee` 表中第二高的薪水 。如果不存在第二高的薪水，查询应该返回 `null` 。

查询结果如下例所示。

**示例 1：**

```
输入：
Employee 表：
+----+--------+
| id | salary |
+----+--------+
| 1  | 100    |
| 2  | 200    |
| 3  | 300    |
+----+--------+
输出：
+---------------------+
| SecondHighestSalary |
+---------------------+
| 200                 |
+---------------------+
```

**示例 2：**

```
输入：
Employee 表：
+----+--------+
| id | salary |
+----+--------+
| 1  | 100    |
+----+--------+
输出：
+---------------------+
| SecondHighestSalary |
+---------------------+
| null                |
+---------------------+
```

代码：

```mysql
# 解决方案 (1)：子查询 + 分页思想
SELECT 
    (SELECT DISTINCT salary 
    FROM Employee
    ORDER BY salary DESC
    LIMIT 1 OFFSET 1) AS SecondHighestSalary

# 解决方案 (2)：IFNULL 函数
SELECT IFNULL(
    (SELECT DISTINCT salary
    FROM Employee
    ORDER BY salary DESC
    LIMIT 1,1), null) AS SecondHighestSalary

# 解决方案 (3)：max聚合两次比较
SELECT MAX(salary) SecondHighestSalary
FROM Employee
WHERE salary <> (SELECT MAX(salary) FROM Employee)
```

解决方案 (1)：子查询 + 分页思想

解决方案 (2)：IFNULL 函数

解决方案 (3)：max聚合两次比较

- **SQL查询语句中的 limit 与 offset 的区别**

1. limit y 分句表示: 读取 y 条数据（limit n 等价于 limit 0,n）
2. **limit x, y** 分句表示: 跳过 x 条数据，读取 y 条数据
3. **limit y offset x** 分句表示: 跳过 x 条数据，读取 y 条数据

### 	[177. 第N高的薪水](https://leetcode-cn.com/problems/nth-highest-salary/)

难度中等566

SQL架构

表: `Employee`

```
+-------------+------+
| Column Name | Type |
+-------------+------+
| id          | int  |
| salary      | int  |
+-------------+------+
Id是该表的主键列。
该表的每一行都包含有关员工工资的信息。
```

 

编写一个SQL查询来报告 `Employee` 表中第 `n` 高的工资。如果没有第 `n` 个最高工资，查询应该报告为 `null` 。

查询结果格式如下所示。

 

**示例 1:**

```
输入: 
Employee table:
+----+--------+
| id | salary |
+----+--------+
| 1  | 100    |
| 2  | 200    |
| 3  | 300    |
+----+--------+
n = 2
输出: 
+------------------------+
| getNthHighestSalary(2) |
+------------------------+
| 200                    |
+------------------------+
```

**示例 2:**

```
输入: 
Employee 表:
+----+--------+
| id | salary |
+----+--------+
| 1  | 100    |
+----+--------+
n = 2
输出: 
+------------------------+
| getNthHighestSalary(2) |
+------------------------+
| null                   |
+------------------------+
```

排名是数据库中的一个经典题目，实际上又根据排名的具体细节可分为3种场景：

- 连续排名，例如薪水3000、2000、2000、1000排名结果为1-2-3-4，体现同薪不同名，排名类似于编号
- 同薪同名但总排名不连续，例如同样的薪水分布，排名结果为1-2-2-4

- 同薪同名且总排名连续，同样的薪水排名结果为1-2-2-3

不同的应用场景可能需要不同的排名结果，也意味着不同的查询策略。

本题的目标是实现第三种排名方式下的第N个结果，且是全局排名，不存在分组的问题，实际上还要相对简单一些。

**值得一提的是**：在Oracle等数据库中有窗口函数，可非常容易实现这些需求，而MySQL直到8.0版本也引入相关函数。最新OJ环境已更新至8.0版本，可直接使用窗口函数。

思路1：**单表查询**(必会)
由于本题不存在分组排序，只需返回全局第N高的一个，所以自然想到的想法是用order by排序加limit限制得到。需要注意两个细节：

同薪同名且不跳级的问题，解决办法是用group by按薪水分组后再order by
排名第N高意味着要跳过N-1个薪水，由于无法直接用limit N-1，所以需先在函数开头处理N为N=N-1。
注：这里不能直接用limit N-1是因为limit和offset字段后面只接受正整数（意味着0、负数、小数都不行）或者单一变量（意味着不能用表达式），也就是说想取一条，limit 2-1、limit 1.1这类的写法都是报错的。
注：这种解法形式最为简洁直观，但仅适用于查询全局排名问题，如果要求各分组的每个第N名，则该方法不适用；而且也不能处理存在重复值的情况。

代码1

```mysql
CREATE FUNCTION getNthHighestSalary(N INT) RETURNS INT
BEGIN
  SET N = N-1;
  RETURN (
      SELECT IFNULL(
          (SELECT DISTINCT salary 
          FROM employee 
          ORDER BY salary DESC
          LIMIT N, 1),null
      )
  );
END
```

```mysql
CREATE FUNCTION getNthHighestSalary(N INT) RETURNS INT
BEGIN
  Set N = N-1;
  RETURN (
      # Write your MySQL query statement below.
      # 可以用分组去除重复数据，也可以用DISTINCT
      SELECT DISTINCT salary
      FROM Employee
    #   GROUP BY salary
      ORDER BY salary DESC
      LIMIT N, 1
  );
END
```

![image-20220224121628116](http://myimg.go2flare.xyz/img/image-20220224121628116.png)

思路2：子查询
排名第N的薪水意味着该表中存在N-1个比其更高的薪水
注意这里的N-1个更高的薪水是指去重后的N-1个，实际对应人数可能不止N-1个，最后返回的薪水也应该去重，因为可能不止一个薪水排名第N
由于对于每个薪水的where条件都要执行一遍子查询，注定其效率低下

代码2

```mysql
CREATE FUNCTION getNthHighestSalary(N INT) RETURNS INT
BEGIN
  RETURN (
      # Write your MySQL query statement below.
      # 子查询，扫描每条记录，某条记录正好大于n-1条其他记录，就是第n条
      SELECT DISTINCT e.salary
      FROM employee e
      WHERE 
        (SELECT count(DISTINCT salary)  
        FROM employee 
        WHERE salary > e.salary) = N-1
  );
END
```

思路3：自连接
一般来说，**能用子查询解决的问题也能用连接解决**。具体到本题：

**两表自连接**，连接条件设定为表1的salary小于表2的salary
以表1的salary分组，统计表1中每个salary分组后对应表2中salary唯一值个数，即去重
限定步骤2中having 计数个数为N-1，即实现了该分组中表1salary排名为第N个
考虑N=1的特殊情形(特殊是因为N-1=0，计数要求为0)，此时不存在满足条件的记录数，但仍需返回结果，所以连接用left join
如果仅查询薪水这一项值，那么不用left join当然也是可以的，只需把连接条件放宽至小于等于、同时查询个数设置为N即可。因为连接条件含等号，所以一定不为空，用join即可。
注：个人认为无需考虑N<=0的情形，毕竟无实际意义。

代码3

```mysql
CREATE FUNCTION getNthHighestSalary(N INT) RETURNS INT
BEGIN
  RETURN (
      SELECT e1.salary 
      FROM employee e1 JOIN employee e2
      ON e1.salary <= e2.salary #e1从大到小
      GROUP BY e1.salary
      HAVING COUNT(e2.salary) = N
  );
END
```

思路4：笛卡尔积
当然，可以很容易将思路2中的代码改为笛卡尔积连接形式，其执行过程实际上一致的，甚至MySQL执行时可能会优化成相同的查询语句。

代码4

```mysql
CREATE FUNCTION getNthHighestSalary(N INT) RETURNS INT
BEGIN
  RETURN (
      SELECT e1.salary 
      FROM employee e1, employee e2
      WHERE e1.salary <= e2.salary #e1从大到小
      GROUP BY e1.salary
      HAVING COUNT(DISTINCT e2.salary) = N
  );
END
```

查询效率


思路5：自定义变量
以上方法2-4中均存在两表关联的问题，表中记录数少时尚可接受，当记录数量较大且无法建立合适索引时，实测速度会比较慢，用算法复杂度来形容大概是O(n^2)量级（实际还与索引有关）。那么，用下面的自定义变量的方法可实现O(2*n)量级，速度会快得多，且与索引无关。

自定义变量实现按薪水降序后的数据排名，同薪同名不跳级，即3000、2000、2000、1000排名后为1、2、2、3；
对带有排名信息的临时表二次筛选，得到排名为N的薪水；
因为薪水排名为N的记录可能不止1个，用distinct去重
代码5

```mysql
CREATE FUNCTION getNthHighestSalary(N INT) RETURNS INT
BEGIN
  RETURN (
      # Write your MySQL query statement below.
      SELECT 
          DISTINCT salary 
      FROM 
          (SELECT 
                salary, @r:=IF(@p=salary, @r, @r+1) AS rnk,  @p:= salary 
            FROM  
                employee, (SELECT @r:=0, @p:=NULL)init 
            ORDER BY 
                salary DESC) tmp
      WHERE rnk = N
  );
END
```

查询效率


思路6：窗口函数
实际上，在mysql8.0中有相关的内置函数，而且考虑了各种排名问题：

row_number(): 同薪不同名，相当于行号，例如3000、2000、2000、1000排名后为1、2、3、4
rank(): 同薪同名，有跳级，例如3000、2000、2000、1000排名后为1、2、2、4
dense_rank(): 同薪同名，无跳级，例如3000、2000、2000、1000排名后为1、2、2、3
ntile(): 分桶排名，即首先按桶的个数分出第一二三桶，然后各桶内从1排名，实际不是很常用
显然，本题是要用第三个函数。
另外这三个函数必须要要与其搭档over()配套使用，over()中的参数常见的有两个，分别是

partition by，按某字段切分
order by，与常规order by用法一致，也区分ASC(默认)和DESC，因为排名总得有个依据
注：下面代码仅在mysql8.0以上版本可用，最新OJ已支持。

代码6

```mysql
CREATE FUNCTION getNthHighestSalary(N INT) RETURNS INT
BEGIN
  RETURN (
      # Write your MySQL query statement below.
        SELECT 
            DISTINCT salary
        FROM 
            (SELECT 
                salary, dense_rank() over(ORDER BY salary DESC) AS rnk
             FROM 
                employee) tmp
        WHERE rnk = N
  );
END
```

查询效率


至此，可以总结MySQL查询的一般性思路是：

能用单表优先用单表，即便是需要用group by、order by、limit等，效率一般也比多表高

不能用单表时优先用连接，连接是SQL中非常强大的用法，小表驱动大表+建立合适索引+合理运用连接条件，基本上连接可以解决绝大部分问题。但join级数不宜过多，毕竟是一个接近指数级增长的关联效果

能不用子查询、笛卡尔积尽量不用，虽然很多情况下MySQL优化器会将其优化成连接方式的执行过程，但效率仍然难以保证

自定义变量在复杂SQL实现中会很有用，例如LeetCode中困难级别的数据库题目很多都需要借助自定义变量实现

如果MySQL版本允许，某些带聚合功能的查询需求应用窗口函数是一个最优选择。除了经典的获取3种排名信息，还有聚合函数、向前向后取值、百分位等，具体可参考官方指南。以下是官方给出的几个窗口函数的介绍：


最后的最后再补充一点，本题将查询语句封装成一个自定义函数并给出了模板，实际上是降低了对函数语法的书写要求和难度，而且提供的函数写法也较为精简。然而，自定义函数更一般化和常用的写法应该是分三步：

定义变量接收返回值
执行查询条件，并赋值给相应变量
返回结果
例如以解法5为例，如下写法可能更适合函数初学者理解和掌握：

```mysql
CREATE FUNCTION getNthHighestSalary(N INT) RETURNS INT
BEGIN
    # i 定义变量接收返回值
    DECLARE ans INT DEFAULT NULL;  
    # ii 执行查询语句，并赋值给相应变量
    SELECT 
        DISTINCT salary INTO ans
    FROM 
        (SELECT 
            salary, @r:=IF(@p=salary, @r, @r+1) AS rnk,  @p:= salary 
        FROM  
            employee, (SELECT @r:=0, @p:=NULL)init 
        ORDER BY 
            salary DESC) tmp
    WHERE rnk = N;
    # iii 返回查询结果，注意函数名中是 returns，而函数体中是 return
    RETURN ans;
END
```



# 常见问题

mysql -u root -p 回车输入密码进入mysql 


show processlist; 
查看连接数，可以发现有很多连接处于sleep状态，这些其实是暂时没有用的，所以可以kill掉

show variables like "max_connections"; 
查看最大连接数，应该是与上面查询到的连接数相同，才会出现too many connections的情况

set GLOBAL max_connections=1000; 
修改最大连接数，但是这不是一劳永逸的方法，应该要让它自动杀死那些sleep的进程。

show global variables like 'wait_timeout'; 
这个数值指的是mysql在关闭一个非交互的连接之前要等待的秒数，默认是28800s

set global wait_timeout=300; 
修改这个数值，这里可以随意，最好控制在几分钟内 


set global interactive_timeout=500; 
修改这个数值，表示mysql在关闭一个连接之前要等待的秒数，至此可以让mysql自动关闭那些没用的连接，但要注意的是，正在使用的连接到了时间也会被关闭，因此这个时间值要合适

批量kill之前没用的sleep连接，在网上搜索的方法对我都不奏效，因此只好使用最笨的办法，一个一个kill

select concat('KILL ',id,';') from information_schema.processlist where user='root'; 先把要kill的连接id都查询出来
复制中间的kill id;内容到word文档
替换掉符号“|”和回车符（在word中查询^p即可查询到回车符）
把修改过的内容复制回终端，最后按回车执行！




方法二

google了一下，

发现这算属MySQL的一个bug，不管连接是通过hosts还是ip的方式，MySQL都会对DNS做反查，IP到DNS，由于反查的接续速度过慢

（不管是不是isp提供的dns服务器的问题或者其他原因），大量的查询就难以应付，线程不够用就使劲增加线程，但是却得不到释放，所以MySQL会“假死”。

解决的方案很简单，结束这个反查的过程，禁止任何解析。

打开mysql的配置文件（my.cnf），在[mysqld]下面增加一行：

skip-name-resolve

重新载入配置文件或者重启MySQL服务即可。