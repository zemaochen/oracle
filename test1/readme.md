# Oracle
# 实验一：分析SQL执行计划，执行SQL语句的优化指导

## 实验内容：
- 对Oracle12c中的HR人力资源管理系统中的表进行查询与分析。
- 首先运行和分析教材中的样例：本训练任务目的是查询两个部门('IT'和'Sales')的部门总人数和平均工资，以下两个查询的结果是一样的。但效率不相同。
- 设计自己的查询语句，并作相应的分析，查询语句不能太简单。

## 教材中的查询语句

- 查询1：

```SQL
SELECT d.department_name，count(e.job_id)as "部门总人数"，
avg(e.salary)as "平均工资"
from hr.departments d，hr.employees e
where d.department_id = e.department_id
and d.department_name in ('IT'，'Sales')
GROUP BY department_name;
```
- 执行结果及优化指导  

![查询一结果图片](https://github.com/zemaochen/Oracle/blob/master/images/QQ%E5%9B%BE%E7%89%8720181016164946.png)  
![查询一优化指导图片](https://github.com/zemaochen/Oracle/blob/master/images/%E5%9B%BE%E7%89%871.png)
- 查询执行计划输出
```sql
-------------------------------------------------------------------------------
EXPLAIN PLANS SECTION
-------------------------------------------------------------------------------

1- Original
-----------
Plan hash value: 3808327043

 
---------------------------------------------------------------------------------------------------
| Id  | Operation                     | Name              | Rows  | Bytes | Cost (%CPU)| Time     |
---------------------------------------------------------------------------------------------------
|   0 | SELECT STATEMENT              |                   |     1 |    23 |     5  (20)| 00:00:01 |
|   1 |  HASH GROUP BY                |                   |     1 |    23 |     5  (20)| 00:00:01 |
|   2 |   NESTED LOOPS                |                   |    19 |   437 |     4   (0)| 00:00:01 |
|   3 |    NESTED LOOPS               |                   |    20 |   437 |     4   (0)| 00:00:01 |
|*  4 |     TABLE ACCESS FULL         | DEPARTMENTS       |     2 |    32 |     3   (0)| 00:00:01 |
|*  5 |     INDEX RANGE SCAN          | EMP_DEPARTMENT_IX |    10 |       |     0   (0)| 00:00:01 |
|   6 |    TABLE ACCESS BY INDEX ROWID| EMPLOYEES         |    10 |    70 |     1   (0)| 00:00:01 |
---------------------------------------------------------------------------------------------------
 
Query Block Name / Object Alias (identified by operation id):
-------------------------------------------------------------
 
   1 - SEL$1
   4 - SEL$1 / D@SEL$1
   5 - SEL$1 / E@SEL$1
   6 - SEL$1 / E@SEL$1
 
Predicate Information (identified by operation id):
---------------------------------------------------
 
   4 - filter("D"."DEPARTMENT_NAME"='IT' OR "D"."DEPARTMENT_NAME"='Sales')
   5 - access("D"."DEPARTMENT_ID"="E"."DEPARTMENT_ID")
 
Column Projection Information (identified by operation id):
-----------------------------------------------------------
 
   1 - (#keys=1) "DEPARTMENT_NAME"[VARCHAR2,30], COUNT("E"."SALARY")[22], COUNT(*)[22], 
       SUM("E"."SALARY")[22]
   2 - (#keys=0) "D"."DEPARTMENT_ID"[NUMBER,22], "DEPARTMENT_NAME"[VARCHAR2,30], 
       "E"."SALARY"[NUMBER,22]
   3 - (#keys=0) "D"."DEPARTMENT_ID"[NUMBER,22], "DEPARTMENT_NAME"[VARCHAR2,30], 
       "E".ROWID[ROWID,10]
   4 - "D"."DEPARTMENT_ID"[NUMBER,22], "DEPARTMENT_NAME"[VARCHAR2,30]
   5 - "E".ROWID[ROWID,10]
   6 - "E"."SALARY"[NUMBER,22]
 
Note
-----
   - this is an adaptive plan

2- Using New Indices
--------------------
Plan hash value: 916193473

 
---------------------------------------------------------------------------------------------------
| Id  | Operation                     | Name              | Rows  | Bytes | Cost (%CPU)| Time     |
---------------------------------------------------------------------------------------------------
|   0 | SELECT STATEMENT              |                   |     2 |    46 |     2   (0)| 00:00:01 |
|   1 |  SORT GROUP BY NOSORT         |                   |     2 |    46 |     2   (0)| 00:00:01 |
|   2 |   NESTED LOOPS                |                   |    19 |   437 |     2   (0)| 00:00:01 |
|   3 |    NESTED LOOPS               |                   |    20 |   437 |     2   (0)| 00:00:01 |
|   4 |     INLIST ITERATOR           |                   |       |       |            |          |
|*  5 |      INDEX RANGE SCAN         | IDX$$_00220001    |     2 |    32 |     1   (0)| 00:00:01 |
|*  6 |     INDEX RANGE SCAN          | EMP_DEPARTMENT_IX |    10 |       |     0   (0)| 00:00:01 |
|   7 |    TABLE ACCESS BY INDEX ROWID| EMPLOYEES         |    10 |    70 |     1   (0)| 00:00:01 |
---------------------------------------------------------------------------------------------------
 
Query Block Name / Object Alias (identified by operation id):
-------------------------------------------------------------
 
   1 - SEL$1
   5 - SEL$1 / D@SEL$1
   6 - SEL$1 / E@SEL$1
   7 - SEL$1 / E@SEL$1
 
Predicate Information (identified by operation id):
---------------------------------------------------
 
   5 - access("D"."DEPARTMENT_NAME"='IT' OR "D"."DEPARTMENT_NAME"='Sales')
   6 - access("D"."DEPARTMENT_ID"="E"."DEPARTMENT_ID")
 
Column Projection Information (identified by operation id):
-----------------------------------------------------------
 
   1 - (#keys=1) "DEPARTMENT_NAME"[VARCHAR2,30], COUNT("E"."SALARY")[22], COUNT(*)[22], 
       SUM("E"."SALARY")[22]
   2 - (#keys=0) "DEPARTMENT_NAME"[VARCHAR2,30], "E"."SALARY"[NUMBER,22]
   3 - (#keys=0) "DEPARTMENT_NAME"[VARCHAR2,30], "E".ROWID[ROWID,10]
   4 - "D"."DEPARTMENT_ID"[NUMBER,22], "DEPARTMENT_NAME"[VARCHAR2,30]
   5 - "D"."DEPARTMENT_ID"[NUMBER,22], "DEPARTMENT_NAME"[VARCHAR2,30]
   6 - "E".ROWID[ROWID,10]
   7 - "E"."SALARY"[NUMBER,22]
```
- 查询2：
```SQL
SELECT d.department_name，count(e.job_id)as "部门总人数"，
avg(e.salary)as "平均工资"
FROM hr.departments d，hr.employees e
WHERE d.department_id = e.department_id
GROUP BY department_name
HAVING d.department_name in ('IT'，'Sales');
```
- 执行结果及优化指导  

![查询二结果图片](https://github.com/zemaochen/Oracle/blob/master/images/QQ%E5%9B%BE%E7%89%8720181016164931.png)  
![查询二优化指导图片](https://github.com/zemaochen/Oracle/blob/master/images/%E5%9B%BE%E7%89%872.png)
- 查询执行计划输出    
```sql
1- Original
-----------
Plan hash value: 2128232041

 
----------------------------------------------------------------------------------------------
| Id  | Operation                      | Name        | Rows  | Bytes | Cost (%CPU)| Time     |
----------------------------------------------------------------------------------------------
|   0 | SELECT STATEMENT               |             |     1 |    23 |     7  (29)| 00:00:01 |
|*  1 |  FILTER                        |             |       |       |            |          |
|   2 |   HASH GROUP BY                |             |     1 |    23 |     7  (29)| 00:00:01 |
|   3 |    MERGE JOIN                  |             |   106 |  2438 |     6  (17)| 00:00:01 |
|   4 |     TABLE ACCESS BY INDEX ROWID| DEPARTMENTS |    27 |   432 |     2   (0)| 00:00:01 |
|   5 |      INDEX FULL SCAN           | DEPT_ID_PK  |    27 |       |     1   (0)| 00:00:01 |
|*  6 |     SORT JOIN                  |             |   107 |   749 |     4  (25)| 00:00:01 |
|   7 |      TABLE ACCESS FULL         | EMPLOYEES   |   107 |   749 |     3   (0)| 00:00:01 |
----------------------------------------------------------------------------------------------
 
Query Block Name / Object Alias (identified by operation id):
-------------------------------------------------------------
 
   1 - SEL$1
   4 - SEL$1 / D@SEL$1
   5 - SEL$1 / D@SEL$1
   7 - SEL$1 / E@SEL$1
 
Predicate Information (identified by operation id):
---------------------------------------------------
 
   1 - filter("DEPARTMENT_NAME"='IT' OR "DEPARTMENT_NAME"='Sales')
   6 - access("D"."DEPARTMENT_ID"="E"."DEPARTMENT_ID")
       filter("D"."DEPARTMENT_ID"="E"."DEPARTMENT_ID")
 
Column Projection Information (identified by operation id):
-----------------------------------------------------------
 
   1 - "DEPARTMENT_NAME"[VARCHAR2,30], COUNT("E"."SALARY")[22], COUNT(*)[22], 
       SUM("E"."SALARY")[22]
   2 - (#keys=1) "DEPARTMENT_NAME"[VARCHAR2,30], COUNT("E"."SALARY")[22], 
       COUNT(*)[22], SUM("E"."SALARY")[22]
   3 - (#keys=0) "D"."DEPARTMENT_NAME"[VARCHAR2,30], "E"."SALARY"[NUMBER,22]
   4 - "D"."DEPARTMENT_ID"[NUMBER,22], "D"."DEPARTMENT_NAME"[VARCHAR2,30]
   5 - "D".ROWID[ROWID,10], "D"."DEPARTMENT_ID"[NUMBER,22]
   6 - (#keys=1) "E"."DEPARTMENT_ID"[NUMBER,22], "E"."SALARY"[NUMBER,22]
   7 - "E"."SALARY"[NUMBER,22], "E"."DEPARTMENT_ID"[NUMBER,22]

-------------------------------------------------------------------------------
```
- 分析  
    ***从查询执行计划可以知道。  
    查询一：Cost=5，Rows=20 Predicate Information（谓词信息）中有一次索引搜索access，一次全表搜索filter。  
    查询二：Cost=7，Rows=106 Predicate Information中有一次搜索access，两次搜索filter  
    比较可以看出查询一的大部分参数都优于查询二。从分析两个查询的sql语句来看，查询一是先过滤后汇总，
    参与汇总与计算的数据量少，而查询二是先汇总后过滤，参与汇总与计算的数据量多。所以相比之下查询一更优。***
    ## 自定义查询语句
    ```SQL
    SELECT d.department_name,count(e.job_id)as "部门总人数",
    avg(e.salary)as "平均工资"
    FROM hr.departments d,hr.employees e
    WHERE d.department_id = e.department_id
    and d.department_name = 'IT' or d.department_name = 'Sales'
    GROUP BY department_name
    ```
    
    执行结果：
    
    ![测试图片](https://github.com/zemaochen/Oracle/blob/master/images/3.png)
 - 执行计划：  
   ![测试图片](https://github.com/zemaochen/Oracle/blob/master/images/3_执行计划.png)
    
- 优化建议  
   在进行连接的时候会造成开销很大的笛卡尔积操作，可以进一步优化。
   
