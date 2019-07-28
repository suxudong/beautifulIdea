# beautifulIdea
Theme for idea

Author:suxudong
Email:89565347x@gmail.com 

在项目中做权限控制时，需要用组织阶层来控制能够访问的数据，
比如A组织的人可以看到其下属组织的人员数据，或者只有A组织是B组织上级的时候才有看B组织人员数据的权利。
根据需求需要构筑DB的表结构，如下（ORG_RANK）

组织ID(PK) 上位组织ID 
ORG_ID HIGH_ORG_ID

根据上面的结构，使用Oracle的树查询语句(start with和connect by)来创建SQL语句，如下：

查询指定组织的直属下层组织：

Sql代码    
select  ORANK.ORG_ID   
  from  ORG_RANK ORANK   
where  ( level  - 1) = 1   
start with  ORANK.ORG_ID = #orgId#   
connect   by   prior  ORANK.ORG_ID = ORANK.HIGH_ORG_ID   
select ORANK.ORG_ID
  from ORG_RANK ORANK
where (level - 1) = 1
start with ORANK.ORG_ID = #orgId#
connect by prior ORANK.ORG_ID = ORANK.HIGH_ORG_ID对以上SQL做性能评定时发现出现严重性能问题，（10层组织，3000条数据时）查询时间1分多钟，下面进行了优化。

1、分析执行计划，发现有Full Table，说明使用索引失败，优化的方法是对HIGH_ORG_ID加上索引。

2、虽然只是查询直属下层的组织，但是上面SQL实际执行时，先查询出指定组织的所有下层组织，

然后再从结果里过滤出直属下层的组织（where (level - 1) = 1）。

上面的分析可以得到证明，因为输入倒数第二层组织的执行时间会比输入最上层组织的执行时间少的多。

优化方法是增加connect by语句的条件（and (level - 1) <= 1），不满足条件的子树不会被查询，会省去很多没用的递归查询。

Sql代码    
select  ORANK.ORG_ID   
  from  ORG_RANK ORANK   
where  ( level  - 1) = 1   
start with  ORANK.ORG_ID = #orgId#   
connect   by   prior  ORANK.ORG_ID = ORANK.HIGH_ORG_ID   
and  ( level  - 1) <= 1   
select ORANK.ORG_ID
  from ORG_RANK ORANK
where (level - 1) = 1
start with ORANK.ORG_ID = #orgId#
connect by prior ORANK.ORG_ID = ORANK.HIGH_ORG_ID
and (level - 1) <= 1判断组织A是组织B的上层组织：

方法一：查询出A的所有下层组织，看其中是否有B；

方法二：查询出B的所有上层组织，看其中是否有A。

只要你头脑里自己描绘出一个树型的组织结构，那么你自然会想到方法二的执行速度会明显比方法一块，

方法二是逆行查询，查到的数据量小。
