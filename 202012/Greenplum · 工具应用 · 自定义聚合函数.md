# 一、背景
  常见的聚合函数有sum、max、avg、min，但有些场景可以定制聚合函数解决特定的需求。本文给出自定义聚合函数的示例。

# 二、需求描述
实现一个工地搬砖的工资核算系统。

## 1.工地搬砖规则：
i.底薪：1000元

ii.搬一块砖：10元

iii.及时奖：100元

## 2.工作量信息
i）Jack

搬了三天砖，分别10，15，20 ;工资：1000+（10+15+20）*10 +100 = 1550元

ii)Tom

搬了三天砖，分别30，20，40；工资：1000+（30+20+40）*10 +100 = 2000元

# 三、需求实现
## 1、创建person表
CREATE TABLE person

(

        name    varchar(200),

        work              int

);

insert into person values('Jack',10),('Jack',15),('Jack',20);

insert into person values('Tom',30),('Tom',20),('Tom',40);

## 2.创建函数money_add
每天搬砖，劳动报酬进行增加。

CREATE FUNCTION money_add (numeric, numeric, numeric)

RETURNS numeric AS

$$

        SELECT $1 + $2*$3;

$$ LANGUAGE 'sql' 

STRICT;

##3.创建函数money_total
劳动总报酬核算，100为及时奖金。

CREATE FUNCTION money_total(numeric)

RETURNS numeric AS

$$

        SELECT round($1 + 100, -1);

$$ LANGUAGE 'sql'

STRICT;

## 4.创建聚合函数work_money
用于输出工资报表。

CREATE AGGREGATE work_money(numeric, numeric)

(

        INITCOND = 1000,

        STYPE = numeric,

        SFUNC = money_add,

        FINALFUNC = money_total

);

## 5、结果验证
SELECT name,

        work_money(work, 10)

FROM person 

GROUP BY name；

![image](https://github.com/sinwaj/database/blob/main/images/2020-01.png)

