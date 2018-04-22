#多表查询

创建时间:2018.04.22

---

##*链表查询*

###1.交叉查询

```sql
    SELECT * FROM table1,table2 
```
返回的是两张表记录的笛卡尔积。

###2.内连接(inner join)

隐式查询：不使用关键字
```sql
    SELECT * FROM t1,t2
            WHRER t1.id=t2.t1Id
```
            
或者(非隐式查询):
```sql
   SELECT * FROM t1 inner join t1
        on t1.id=t2.t1Id
 
```

###3.外连接(outter join)

1.左外:返回满足连接条件的记录,同时返回左表中不满足条件的记录。
```sql
    SELECT * FROM t1 left outter join t2
        on t1.id=t2.t1Id
```
       
2.右外连：返回满足连接条件的记录,同时返回右表中不满足条件的记录。
    
3.全外连
  
--- 
    
##子查询/嵌套查询

```sql
SELECT * FROM t1
    where id in(
        SELECT id FROM t2
            where id=#{id}     
  )
``` 

---

##统计查询

```sql 
select count(*) from t
```

```sql
select sum(colume_name)/count(*) from t
```

```sql
seletc avg(colume_name) from t
```

查询购买了哪些商品 ，并且没类总价大于100的商品
```sql
select product,sum(price) as total from order
    group by product
having total>100
```

##分页查询

```sql
select * from t limit (offset,nums)
```







