<!-- GFM-TOC -->
* [一、Oracle基础](#一Oracle基础)  
* [二、一切都是对象](#二一切都是对象)  
* [三、操作符](#三操作符)  
* [四、初始化与清理](#四初始化与清理)
* [五、访问权限控制](#五访问权限控制)
* [六、复合类](#六复合类)
* [七、多态](#七多态)
* [八、接口](#八接口)
* [九、数组](#九数组)
* [十、字符串](#十字符串)
* [十一、字符串](#十一内部类)
* [十二、持有对象](#十二持有对象)
* [十三、异常处理](#十三异常处理)
<!-- GFM-TOC -->

# 一、Oracle基础  
  
- 简介  
 - 性能优越  大型数据库中的典范      大型数据库：sybase、Oracle、db2       中型数据库：mysql、SqlServer、infomix       小型数据库：Access、Visual Foxpro   - 是对象关系型的数据库管理系统 (ORDBMS)  
 - 应用广泛，在管理信息系统、企业数据处理、因特网及电子商务等领域使用非常广泛  
 - 在数据**安全性**与**数据完整性**控制方面性能优越  
 - 跨操作系统、跨硬件平台的数据互操作能力  
 - 支持**多用户、大事务量**的事务处理  
 - 可移植性好，特别是可传输表空间的特性   
 ---
- 通过 SQL可以实现与 Oracle 服务器的通信    
 - SQL 是 Structured Query Language（结构化查询语言）的首字母缩写词
 - SQL 是数据库语言，Oracle 使用该语言存储和检索信息
 - 表是主要的数据库对象，用于存储数据
 ---  - SQL 支持下列类别的命令： - 数据定义语言（DDL）：create、alter、drop
 - 数据操纵语言（DML）：insert、delete、update、select
 - 事务控制语言（TCL）：commit、savepoint、rollback
 - 数据控制语言（DCL）：grant、revoke
 ---- 数据类型：字符、数值、日期时间、RAW/LONG RAW、LOB
- 字符数据类型：char、varchar2、long
- Oracle 中伪列就像一个表列，但是它并没有存储在表中
 - 伪列可以从表中查询，但不能插入、更新和删除它们的值
 - 常用的伪列有ROWID和ROWNUM
 - ROWID 是表中行的存储地址，该地址可以唯一地标识数据库中的一行，可以使用 ROWID 伪列快速地定位表中的一行
 - ROWNUM 是查询返回的结果集中行的序号，可以使用它来限制查询返回的行数（有点类似于top 常用于分页）
 ---- 数据定义语言用于改变数据库结构，包括创建、更改和删除数据库对象，用于操纵表结构的数据定义语言命令有： - CREATE TABLE
 - ALTER TABLE
 - TRUNCATE TABLE
 - DROP TABLE
 ---- 利用现有的表创建表：CREATE TABLE <new_table_name> AS SELECT column_names FROM <old_table_name>;

- 选择无重复的行：在SELECT子句，使用DISTINCT关键字 - SELECT DISTINCT sname FROM student;

- 使用列别名：为列表达式提供不同的名称，该别名指定了列标题

- 插入来自其它表中的记录：INSERT INTO student2 SELECT * FROM student;

- 数据控制语言为用户提供权限控制命令，用于权限控制的命令有：
 - GRANT 授予权限
 - REVOKE 撤销已授予的权限
 --- 
- SQL操作符：算术操作符、比较操作符、逻辑操作符、集合操作符、连接操作符
- 连接操作符用于将多个字符串或数据值合并成一个字符串：通过使用连接操作符可以将表中的多个列合并成逻辑上的一行列- SQL 操作符的优先级从高到低的顺序是： - 算术操作符           --------最高优先级
 - 连接操作符
 - 比较操作符
 - NOT 逻辑操作符
 - AND 逻辑操作符
 - OR   逻辑操作符   --------最低优先级 
 ---- SQL函数：单行函数、分组函数、分析函数
- 单行函数：单行函数对于从表中查询的每一行只返回一个值，可以出现在 SELECT 子句中和 WHERE 子句中。单行函数可以大致划分为：
 - 字符函数
 - 日期时间函数
 - 数字函数
 - 转换函数
 - 混合函数

 - Oracel中的length（）方法，字符和汉字同等计算

- 分组函数：基于一组行来返回结果，为每一组行返回一个值，经典如下
 - AVG
 - MIN
 - MAX
 - SUM
 - COUNT
 - GROUP BY
 - HAVING
 ---- 多表查询：等值连接、外连接、自连接、子查询  

 - 等值连接：① select table1.column,table2.column from  table1, table2 where  table1.column1=table2.column2 ；②  select table1.column,table2.column from  table1 inner join table2  on table1.column1=table2.column2；
 - 左（右 right  全 full）外连接：① select table1.column,table2.column from  table1 left  outer  join table2 on table1.column1=table2.column2； ②  select table1.column,table2.column from  table1, table2 where  table1.column1=table2.column2(+)；
 - 自连接：SELECT STUDNET.CLASSID,TEACHER.DEPTNO FROM STUDENT  NATURAL JOIN TEACHER;

- 分析函数： 分析函数用于计算完成聚集的累计排名、序号等，分析函数为每组记录返回多个行。以下三个分析函数用于计算一个行在一组有序行中的排位，序号从1开始：
    - ROW_NUMBER 返回连续的排序，不论值是否相等
    - RANK 具有相等值的行排序相同，序数随后跳跃
    - DENSE_RANK 具有相等值的行排序相同，序号是连续的

 ---- **集合操作符**
 - UNION：合并两个或多个 SELECT 语句的结果集，自动处理重复
 - UNION ALL：和 UNION 命令几乎是等效的，不过 UNION ALL 命令会列出所有的值
 - INTERSECT：返回两个查询的公共行
 - MINUS：返回从第一个查询结果中排除第二个查询中出现的行
- **重命名** 
 - 重命名表：rename table_name1 to table_name2;
 - 重命名列：alter table table_name rename column col_oldname to colnewname ;
 ---- SQL的执行顺序
 - 常见的select、from、where的顺序：1.from  2. where  3. select
 - 完整的select、from、where、group by、having、order by的顺序：1. from  2. where  3. group by  4.having  5. select  6. order by- EXISTS用来判断查询所得的结果中，是否有满足条件的纪录存在。例:select *　from student   where exists(select * from address  where city='杭州');
- SELECT CASE WHEN 的使用
 - CASE   
  WHEN   条件1   THEN   action1     WHEN   条件2   THEN   action2    WHEN   条件3   THEN   action3     …    ELSE actionN    END CASE - 上述例子：  
 select  case         when  substr('20180310',5,2) = '01'  then  '一月份'        when  substr('20180310',5,2) = '02'  then  '二月份'        when  substr('20180310',5,2) = '03'  then  '三月份'        when  substr('20180310',5,2) = '04'  then  '四月份'        else null      end   from dual;  - SELECT CASE SELECTOR 的使用 - CASE selector  
   WHEN value1 THEN action1     WHEN value2 THEN action2     …     ELSE actionN      END [CASE] - 上述例子：  
 select  case substr('20180310',5,2)         when   '01'  then  '一月份'        when    '02'  then  '二月份'        when   '03'  then  '三月份'        when    '04'  then  '四月份'        else null      end   from dual; ---- DECODE
 - 在逻辑编程中，经常用到If – Then –Else 进行逻辑判断。在DECODE的语法中，实际上就是这样的逻辑处理过程
 - DECODE(value, if1, then1,  if2,then2, if3,then3,  . . .  else )
 - Value 代表某个表的任何类型的任意列或一个通过计算所得的任何结果。当每个value值被测试，如果value的值为if1，Decode 函数的结果是then1；如果value等于if2，Decode函数结果是then2；等等。事实上，可以给出多个if/then 配对。如果value结果不等于给出的任何配对时，Decode 结果就返回else
 - ROWNUM
 - 作用：对查询结果，输出前若干条记录
 - 注意：只能与<、<=、between and连用  
  - GROUP BY GROUPING SETS  
 - 可以用GROUP BY GROUPING SETS来进行分组自定义汇总，可以应用它来指定你需要的总数组合 
 - 其格式为： GROUP BY GROUPING SETS ((list), (list) ... )
 - 这里（list）是圆括号中的一个列序列，这个组合生成一个总数。要增加一个总和，必须增加一个(NUlL)分组集   