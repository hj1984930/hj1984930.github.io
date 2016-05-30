---
layout: post
title: 关联账户识别
description: ""
modified: 2016-05-30
tags: [python, greenplum, database]
image:
  feature: account.png
  
---

## 关联账户识别

### 思路

- 模拟一年股指期货报撤单数据
- 计算任意两个账户之间是否是关联账户
- 计算所有账户组合是否关联账户


### 模拟数据

- 20万用户，3万活跃用户
- 4千万报撤单
- 1年的数据

### 关联账户计算

- 输入参数：任意两个账户ID
- 输出参数：是否关联账户
- 数据：1年共计4千万条
- 耗时：800-900毫秒


### 计算所有组合

- 组合数量：4.5亿个组合
- 共计耗时：3.6亿秒，即4千天

### 并行计算

- 组合数量平均分成4千份
- Greeplum中同时提交，并行计算
- 估计需要1-2天

### 计算任意两个账户ID的关联关系
~~~sql
WITH client_t1_orders AS (
        SELECT clientid, count(*) AS total_orders
        FROM if
        WHERE clientid = '137849'
        GROUP BY clientid
     ), client_t2_orders AS (
        SELECT clientid, count(*) AS total_orders
        FROM if
        WHERE clientid = '174243'
        GROUP BY clientid
     ),order_compare AS(
     SELECT t1.tradingday,t1.clientid as clientid_t1,t2.clientid as clientid_t2,
     EXTRACT(EPOCH FROM (t1.inserttime -  t2.inserttime )),
     CASE WHEN EXTRACT(EPOCH FROM (t1.inserttime -  t2.inserttime )) < 60
     AND  EXTRACT(EPOCH FROM (t1.inserttime -  t2.inserttime ))  > -60
     THEN 1 ELSE 0 
     END  AS same_orders
     FROM if t1
     INNER JOIN if t2
     on t1.tradingday = t2.tradingday 
     and t1.instrumentid = t2.instrumentid
     and t1.direction = t2.direction 
     and t1.clientid  = 137849
     and t2.clientid = 174243
     )
SELECT t1.total_orders,round(t2.same_orders*1.0/t1.total_orders,4) as rate
FROM(
SELECT min(total_orders) AS total_orders
FROM
    (SELECT total_orders
     FROM client_t1_orders
     UNION ALL 
     SELECT total_orders
     FROM client_t1_orders
     )tt
     )t1
CROSS JOIN (SELECT clientid_t1 ,clientid_t2,sum(same_orders) AS same_orders
            FROM order_compare 
            GROUP BY clientid_t1 ,clientid_t2
  
)t2

~~~