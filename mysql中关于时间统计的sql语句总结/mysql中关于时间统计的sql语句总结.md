在之前写VR360时有一个统计页面（[https://vr.beifengtz.com/p/statistics.html](https://vr.beifengtz.com/p/statistics.html)），在此页面的数据统计时用到了很多mysql中日期函数和时间统计sql语句，当时也是参考了一些资料才写出来的。在平时开发中，涉及到统计数据、报表甚至大数据计算时一定会使用这些日期函数，其他关系数据库也是类似的，我是以mysql为例，比较简单还免费嘛。话不多说，下面直接列出常用的时间统计sql语句，记录下来方便以后学习巩固。

# 常用时间函数

1. DAYOFWEEK(date)

返回 date 的星期索引(1 = Sunday, 2 = Monday, ... 7 = Saturday)。索引值符合 ODBC 的标准。

        mysql> SELECT DAYOFWEEK(’1998-02-03’); 
            -> 3 

2. WEEKDAY(date)   

返回 date 的星期索引(0 = Monday, 1 = Tuesday, ... 6 = Sunday)：   

    mysql> SELECT WEEKDAY(’1998-02-03 22:23:00’); 
         -> 1 
    mysql> SELECT WEEKDAY(’1997-11-05’); 
         -> 2

3. DAYOFMONTH(date)   

返回 date 是一月中的第几天，范围为 1 到 31：   

    mysql> SELECT DAYOFMONTH(’1998-02-03’); 
         -> 3 

4. DAYOFYEAR(date)   

返回 date 是一年中的第几天，范围为 1 到 366：   

    mysql> SELECT DAYOFYEAR(’1998-02-03’); 
         -> 34 

5. MONTH(date)   

返回 date 中的月份，范围为 1 到 12：  

    mysql> SELECT MONTH(’1998-02-03’); 
         -> 2

6. DAYNAME(date)   

返回 date 的星期名：   

    mysql> SELECT DAYNAME("1998-02-05"); 
         -> ’Thursday’ 

7. MONTHNAME(date)   

返回 date 的月份名： 

    mysql> SELECT MONTHNAME("1998-02-05"); 
         -> ’February’ 
    
8. QUARTER(date)   

返回 date 在一年中的季度，范围为 1 到 4：  

    mysql> SELECT QUARTER(’98-04-01’); 
         -> 2 

9.  WEEK(date)   

    WEEK(date,first)   

    对于星期日是一周中的第一天的场合，如果函数只有一个参数调用，返回 date 为一年的第几周，返回值范围为 0 到 53 (是的，可能有第 53 周的开始)。两个参数形式的 WEEK() 允许你指定一周是否以星期日或星期一开始，以及返回值为 0-53 还是 1-52。 这里的一个表显示第二个参数是如何工作的： 

值     含义 

0     一周以星期日开始，返回值范围为 0-53 

1      一周以星期一开始，返回值范围为 0-53 

2      一周以星期日开始，返回值范围为 1-53 

3      一周以星期一开始，返回值范围为 1-53 (ISO 8601) 


    mysql> SELECT WEEK(’1998-02-20’); 
         -> 7 
    mysql> SELECT WEEK(’1998-02-20’,0); 
         -> 7 
    mysql> SELECT WEEK(’1998-02-20’,1); 
         -> 8 
    mysql> SELECT WEEK(’1998-12-31’,1); 
         -> 53 

注意，在版本 4.0 中，WEEK(#,0) 被更改为匹配 USA 历法。 注意，如果一周是上一年的最后一周，当你没有使用 2 或 3 做为可选参数时，MySQL 将返回 0： 

    mysql> SELECT YEAR(’2000-01-01’), WEEK(’2000-01-01’,0); 
         -> 2000, 0 
    mysql> SELECT WEEK(’2000-01-01’,2); 
         -> 52

10. YEAR(date) 

返回 date 的年份，范围为 1000 到 9999： 

    mysql> SELECT YEAR(’98-02-03’); 
         -> 1998 

11. YEARWEEK(date) 

    YEARWEEK(date,first) 

返回一个日期值是的哪一年的哪一周。第二个参数的形式与作用完全与 WEEK() 的第二个参数一致。注意，对于给定的日期参数是一年的第一周或最后一周的，返回的年份值可能与日期参数给出的年份不一致： 


    mysql> SELECT YEARWEEK(’1987-01-01’); 
         -> 198653 

注意，对于可选参数 0 或 1，周值的返回值不同于 WEEK() 函数所返回值(0)， WEEK() 根据给定的年语境返回周值。 
HOUR(time) 
返回 time 的小时值，范围为 0 到 23： 

    mysql> SELECT HOUR(’10:05:03’); 
         -> 10 

12. HOUR(time) 

返回 time 的小时值，范围为 0 到 23： 

    mysql> SELECT HOUR(’10:05:03’); 
         -> 10

13. MINUTE(time) 

返回 time 的分钟值，范围为 0 到 59： 

    mysql> SELECT MINUTE(’98-02-03 10:05:03’); 
         -> 5 

14. SECOND(time) 

返回 time 的秒值，范围为 0 到 59： 

    mysql> SELECT SECOND(’10:05:03’); 
         -> 3 

15. PERIOD_ADD(P,N) 

增加 N 个月到时期 P(格式为 YYMM 或 YYYYMM)中。以 YYYYMM 格式返回值。 注意，期间参数 P 不是 一个日期值：

    mysql> SELECT PERIOD_ADD(9801,2); 
         -> 199803 

16. PERIOD_DIFF(P1,P2) 

返回时期 P1 和 P2 之间的月数。P1 和 P2 应该以 YYMM 或 YYYYMM 指定。 注意，时期参数 P1 和 P2 不是 日期值：

    mysql> SELECT PERIOD_DIFF(9802,199703); 
         -> 11

17. 
DATE_ADD(date,INTERVAL expr type) 

DATE_SUB(date,INTERVAL expr type)

ADDDATE(date,INTERVAL expr type)

SUBDATE(date,INTERVAL expr type) 

这 些函数执行日期的算术运算。ADDDATE() 和 SUBDATE() 分别是 DATE_ADD() 和 DATE_SUB() 的同义词。 在 MySQL 3.23 中，如果表达式的右边是一个日期值或一个日期时间型字段，你可以使用 + 和 - 代替 DATE_ADD() 和 DATE_SUB()(示例如下)。 参数 date 是一个 DATETIME 或 DATE 值，指定一个日期的开始。expr 是一个表达式，指定从开始日期上增加还是减去间隔值。expr 是一个字符串；它可以以一个 “-” 领头表示一个负的间隔值。type 是一个关键词，它标志着表达式以何格式被解释。


# 常用统计SQL

1. 查询一天内的数据
```
select * from table where to_days(column_time) = to_days(now());

select * from table where date(column_time) = curdate();
```

2. 查询一周内的数据
```
select * from table   where DATE_SUB(CURDATE(), INTERVAL 7 DAY) <= date(column_time);
```

3. 查询一月内的数据
```
select * from table where DATE_SUB(CURDATE(), INTERVAL 1 MONTH) <= date(column_time);
```

4. 查询上一周的数据
```
select * from visit_log_db where week(column_time) = WEEK(now())-1;
```

5. 查询选择所有 column_time 值在最后 30 天内的记录。   
```
SELECT something FROM tbl_name WHERE TO_DAYS(NOW()) - TO_DAYS(column_time) <= 30; 
```

6. 查询本年度数据
```
SELECT * 
FROM table 
WHERE year( FROM_UNIXTIME( column_time ) ) = year( curdate( ))
```

7. 查询数据附带季度数
```
SELECT *, quarter( FROM_UNIXTIME( `column_time` ) ) 
FROM `table`
```

8. 查询本季度的数据
```
SELECT * 
FROM table 
WHERE quarter( FROM_UNIXTIME( column_time ) ) = quarter( curdate( ))
```

9. N天内数据
```
SELECT * 
FROM table 
WHERE TO_DAYS(NOW()) - TO_DAYS(column_time) <= N
```

10. 查询'06-03'到'07-08'这个时间段内的数据
```
Select * From table Where 
DATE_FORMAT(column_time,'%m-%d') >= '06-03' and DATE_FORMAT(column_time,'%m-%d') 
<= '07-08';
```
11. 统计某日、某周、某月数据量
```
select count(*) from `table` where `date`='{某天}' 
select count(*) from `table` where date_fo rmat(` date`,'%V')='{某周}' 
select count(*) from `table` where date_format(`date`,'%c')='{某月}'
```
12. 统计每天的访问数量(假设table表为访问日志表)
```
SELECT
COUNT(1) AS countNumber
DATE_FORMAT(column_time,'%Y-%m-%d') AS dateTime
FROM table
GROUP BY DATE_FORMAT(column_time,'%Y-%m-%d')
```

13. 同居每周的数据量
```
select count(*) as countNumber,week(column_time) as weekflg 
from projects 
where year(column_time) =2019 
group by weekflg
```

# format函数
mysql中DATE_FORMAT(date, format)函数可根据format字符串格式化日期或日期和时间值date，返回结果串。
也可用DATE_FORMAT( ) 来格式化DATE 或DATETIME 值，以便得到所希望的格式。根据format字符串格式化date值:

>函数的参数说明:
* %S, %s 两位数字形式的秒（ 00,01, . . ., 59）
* %i 两位数字形式的分（ 00,01, . . ., 59）
* %H 两位数字形式的小时，24 小时（00,01, . . ., 23）
* %h, %I 两位数字形式的小时，12 小时（01,02, . . ., 12）
* %k 数字形式的小时，24 小时（0,1, . . ., 23）
* %l 数字形式的小时，12 小时（1, 2, . . ., 12）
* %T 24 小时的时间形式（hh : mm : s s）
* %r 12 小时的时间形式（hh:mm:ss AM 或hh:mm:ss PM）
* %p AM 或P M
* %W 一周中每一天的名称（ Sunday, Monday, . . ., Saturday）
* %a 一周中每一天名称的缩写（ Sun, Mon, . . ., Sat）
* %d 两位数字表示月中的天数（ 00, 01, . . ., 31）
* %e 数字形式表示月中的天数（ 1, 2， . . ., 31）
* %D 英文后缀表示月中的天数（ 1st, 2nd, 3rd, . . .）
* %w 以数字形式表示周中的天数（ 0 = Sunday, 1=Monday, . . ., 6=Saturday）
* %j 以三位数字表示年中的天数（ 001, 002, . . ., 366）
* % U 周（0, 1, 52），其中Sunday 为周中的第一天
* %u 周（0, 1, 52），其中Monday 为周中的第一天
* %M 月名（January, February, . . ., December）
* %b 缩写的月名（ January, February, . . ., December）
* %m 两位数字表示的月份（ 01, 02, . . ., 12）
* %c 数字表示的月份（ 1, 2, . . ., 12）
* %Y 四位数字表示的年份
* %y 两位数字表示的年份
* %% 直接值“%”