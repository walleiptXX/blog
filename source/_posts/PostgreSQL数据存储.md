title: PostgreSQL数据存储
author: Walleipt.Wang
tags:
  - postreSQL
  - 数据库存储
categories: []
date: 2018-02-07 14:32:00
---
数据库从核心层面来说就是个为了方便开发人员进行数据的读写操作文件系统。
既然是文件系统那他的主要功能不外乎读和写两个方面，那么也就涉及到了数据是怎么存储的，如何检索相关数据。


#### 数据表与文件的映射关系
- **文件路径公式: /data/base/db/table**

- **查询映射关系**
```
select pg_relation_filepath('friends');

select oid,relfilenode,relname from pg_class where relname='friends';

oid2name -d manu_db
```
- **查看文件: hexdump file**

        


#### 文件装载到内存(Shared memory)
- **缓存结构**
	- **Buffer Descriptors**
    ```
    BufferDescription
    _ BufferTag
      _ RelFileNode (spcNode/dbNode/relNode-空间/库/表)
      _ ForkNumber  (后缀)
      _ BlockNumber (分割文件编号)
    ```
	- **Shared Buffers**
    ```
    SharedBuffers
    _ page
      _ Page header
      _ special
      _ Item
      _ Tuple
    ```
    
- **分配(alloc)和替换(replacement)** 
	- **问题**
    
    在将硬盘中的数据加载到内存时，通常考虑到的问题是如何高效运用缓存数据提高数据提取速度。那么这里就涉及到那些内容应该优先放入缓存。

	- **分配**
    
    如果有空闲空间直接进行分配
    
    - **替换（clock sweep 置换算法）**
    ![](https://raw.githubusercontent.com/walleiptXX/blogImgs/master/201802081104333410.png)
    对使用次数低的进行优先排除，使用次数最高5次；
    环形扫描，如果扫到了你，你的usage_count--,直到遇到第一usage_count变成0的buffer
    
    


- **缓存状态和缓存命中率** 
	
    什么是缓存命中率
    
	- **buffercache状态视图**
    
    ```
--创建pg_buffercache视图(psql)
create extension pg_buffercache ;
    
    
--查询buffercache信息：
select * from pg_buffercache;
    
    
--统计排序usagecount,buffercache信息：
select usagecount,count(*),isdirty from pg_buffercache
group by isdirty, usagecount
order by isdirty, usagecount ;
    
    ```

	- **缓存命中率视图(pg_stat_database,pg_statio_user_tables,pg_statio_user_tables)**
    
    ```
--命中率视图统计（包括系统表）
select 
	blks_hit as hit,blks_read as miss , 
	blks_hit::numeric / (blks_read + blks_hit ) as ratio
from pg_stat_database
where datname = 'walleipt_db' ;


--命中率视图统计（非系统表）
SELECT 
	(sum(heap_blks_hit) + sum(heap_blks_read)) AS TABLE_CNT,
	round(sum(heap_blks_hit) / (sum(heap_blks_hit) + sum(heap_blks_read)) * 100 ,3)as TABLE_RATIO,
	(sum(idx_blks_hit) + sum(idx_blks_read)) AS INDEX_CNT,
	round(sum(idx_blks_hit) / (sum(idx_blks_hit) + sum(idx_blks_read)) * 100,3) as INDEX_RATIO/**,
	(sum(toast_blks_hit) + sum(toast_blks_read)) AS TOAST_CNT,
	round(sum(toast_blks_hit) / (sum(toast_blks_hit) + sum(toast_blks_read)) * 100,3) as TOAST_RATIO,
	(sum(tidx_blks_hit) + sum(tidx_blks_read)) AS TOASIDX_CNT ,
	round(sum(tidx_blks_hit) / (sum(tidx_blks_hit) + sum(tidx_blks_read))*100,3) as TOASTIDX_RATIO*/
FROM pg_statio_user_tables ;


--命中率视图统计（用户表维度命中率）
SELECT relname, heap_blks_read, heap_blks_hit,
	round(heap_blks_hit::numeric/(heap_blks_hit + heap_blks_read), 3)
FROM   pg_statio_user_tables
WHERE  heap_blks_read > 0
ORDER  BY 2 DESC ;


    ```

- **page(block)** 