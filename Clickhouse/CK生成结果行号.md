### Clickhouse查询生成行号

##### 1、rowNumberInAllBlocks()：返回行所在结果集中的序列号。此函数仅考虑受影响的Block。

返回的行号为SQL中过滤条件执行后，过滤出的所有数据的中行号。这个结果中所给出的行号为在结果集中存储的顺序的行号。为自然排序的序号
如果需要按照某个字段排序后的行号则需要以第一次查询结果为子查询，使用子查询的结果表重新在进行查询生成行号。
例如：
**SQL-1**

```sql
select rowNumberInAllBlocks() + 1 as ranks,
       field1                     as field1
from table1
WHERE field2 = 'condition1'
  and field3 = 'condition2'
  and field4 = 'condition3'
  and field5 = 'condition4'
order by ranks asc limit 100;
```

**SQL-2**

```sql
select field1
from table1
WHERE field2 = 'condition1'
  and field3 = 'condition2'
  and field4 = 'condition3'
  and field5 = 'condition4' limit 100;
```

以上根据SQL-1和SQL-2查询出来的结果相同。说明行号为过滤结果集中自然存储结果

如果需要按照其中一个字段排序则需要使用的子查询如SQL-3所示
**SQL-3**

```sql
select rowNumberInAllBlocks() + 1 as ranks, field1
from (select field1
      from table1
      WHERE field2 = 'condition1'
        and field3 = 'condition2'
        and field4 = 'condition3'
        and field5 = 'condition4'
      order by field1) as res;
```

如上SQL表明了rowNumberInAllBlocks函数获取的行号为当前结果集中行所在的行号。

另外
如果使用了SQL-3这样的场景，且有使用limit字段则需要将SQL-3当做子查询的结果集进行再次查询，且在查询中需要使用where条件，若无where条件单单只有limit则只有查询结果等同于SQL-3执行结果
需要在最外层的SQL中追加条件。例如SQL-4
**SQL-4**

```sql
select ranks, field1, field2
from (select rowNumberInAllBlocks() + 1 as ranks, field1, field2
      from (select field1, field2
            from table1
            WHERE field2 = 'condition1'
              and field3 = 'condition2'
              and field4 = 'condition3'
              and field5 = 'condition4'
            order by field1) as res) as rowtable
where field2 = 'condition1' limit 10,10;
```

##### 2、row_number(),rank(),dense_rank() 使用row_Number生成行号,此函数需要分区函数配合使用partition by

- ROW_NUMBER()函数表示：从1开始对当前行赋值序号。
- RANK()函数表示：对数据进行排序，最大排序号与数据条数相等。
- DENSE_RANK()函数表示：对数据进行排序，最大排序号小于数据条数。
- PARTITION BY 函数表示：根据那些字段对查询的结果进行分组。
- ORDER BY 函数表示：在进行集合函数计算时，根据分组内的那些字段进行排序。

```sql
select row_number() over w as ranks, field1, field2
from table1
where field2 = 'condition1'
  and field3 = 'condition2'
  and field4 = 'condition3'
  and field5 = 'condition4'
window w as (partition by field2,field3,field4,field5 order by field1)
limit 100;

select row_number() over (partition by field2,field3,field4,field5 order by field1) as ranks, field1, field2
from table1
where field2 = 'condition1'
  and field3 = 'condition2'
  and field4 = 'condition3'
  and field5 = 'condition4' limit 100;
```

以上两个SQL是等价的。
partition by field2,field3,field4,field5 order by field1 这个句表示，根据字段field2,field3,field4,field5分组，并在组内按照field1进行升序排序
以上两个SQL生成的序号是连续的，不同存在相同的序号。每个序号都是唯一的。


```sql
select rank() over w as ranks, dense_rank() over w as ranks2,row_number() over w as num,field1
from table1
where field2 = 'condition1'
  and field3 = 'condition2'
  and field4 = 'condition3'
  and field5 = 'condition4'
window w as (partition by field2,field3,field4,field5 order by field1)
limit 100;
```
执行后得到结果:    
| ranks | ranks2 | num |field1|
|--|--|--|--|
| 1 | 1 | 1 | 397794.00 |
| 2 | 2 | 2 | 387720.00 |
| 3 | 3 | 3 | 119417.40 |
| 4 | 4 | 4 | 114608.22 |
| 5 | 5 | 5 | 114360.92 |
| 5 | 5 | 6 | 114360.92 |
| 5 | 5 | 7 | 114360.92 |
| 8 | 6 | 8 | 113218.00 |
| 9 | 7 | 9 | 100414.06 |
| 10 | 8 | 10 | 87609.00 |
| 11 | 9 | 11 | 73428.20 |
| 12 | 10 | 12 | 63354.96 |
| 13 | 11 | 13 | 61383.30 |

从上面的结果可知    
- RANKS()函数-生成的排序号在排序值相同的时候会赋予相同的序号且下个序号会跳至与下一个数量相等的序号    
- DENSE_RANKS()函数-生成排序号，在排序值相同的时候会赋予相同的序号，下个排序号与上一个排序号是连续的，不会跳过中间的数量。    
- ROW_NUMBER()函数-生成序号，可以理解为单纯的数条目数，与排序字段无关。按照当前的数据从前到后一次编号。    
