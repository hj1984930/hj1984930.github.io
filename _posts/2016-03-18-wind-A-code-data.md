---
layout: post
title: GreenplumDataBase存储万得宏汇高频数据
description: ""
modified: 2016-03-18
tags: [wind, greenplum]
image:
  feature: abstract-11.jpg
  
---

###GreenplumDataBase存储万得宏汇高频数据

- - -

###说明

* 全部A股
* 数据一：含十档买卖盘的tick数据
* 数据二：逐笔成交
* 数据三：委托队列
* 数据四：逐笔委托
* 所有数据均以csv格式文件存储
* 将csv数据导入gpdb

###GPDB中创建表和视图

* 由于数据量大，需要按照时间创建分区表，按列压缩存储

1.含十档买卖盘的tick数据表

~~~sql
-- Table: hffd.tdb_tickab

-- DROP TABLE hffd.tdb_tickab;

CREATE TABLE hffd.tdb_tickab
(
  id bigint NOT NULL DEFAULT nextval('hffd.tdb_tickab_seq'::regclass),
  windcode character(20), --万得代码
  code character(20),     --交易所代码
  date date,              --日期
  "time" numeric,         --时间
  price numeric,          --成交价
  volume numeric,         --成交量
  turover numeric,        --成交额（元）
  matchitem numeric,      --
  interest numeric,       --
  tradeflag character(50),--成交标志
  bsflag character(50),   --BS标志
  accvolume numeric,      --当日累计成交量
  accturover numeric,     --当日成交额（元）
  askprice10 numeric,     --五档叫卖价
  askvolume10 numeric,    --五档叫卖量
  bidprice10 numeric,     --五档叫买价
  bidvolume10 numeric,    --五档叫买量
  askaveprice numeric,    --平均叫卖价
  bidaveprice numeric,    --平均叫买价
  totalaskvolume numeric, --叫卖量
  totalbidvolume numeric, --叫买量
  open numeric,           --开盘价
  high numeric,           --最高价
  low numeric,            --最低价
  preclose numeric        --前收盘
)
WITH (APPENDONLY=true, COMPRESSLEVEL=1, ORIENTATION=column, COMPRESSTYPE=quicklz, 
  OIDS=FALSE
)
DISTRIBUTED BY (id)
PARTITION BY RANGE (date)
( START (date '2013-01-01') INCLUSIVE
   END (date '2017-01-01') EXCLUSIVE
   EVERY (INTERVAL '1 mon') );
ALTER TABLE hffd.tdb_tickab
  OWNER TO gpadmin;
GRANT ALL ON TABLE hffd.tdb_tickab TO gpadmin;
GRANT SELECT ON TABLE hffd.tdb_tickab TO fitl;

-- View: tdb_tickab

-- DROP VIEW tdb_tickab;

CREATE OR REPLACE VIEW tdb_tickab AS 
 SELECT tdb_tickab.id, tdb_tickab.windcode, tdb_tickab.code, tdb_tickab.date, tdb_tickab."time", tdb_tickab.price, tdb_tickab.volume, tdb_tickab.turover, tdb_tickab.matchitem, tdb_tickab.interest, tdb_tickab.tradeflag, tdb_tickab.bsflag, tdb_tickab.accvolume, tdb_tickab.accturover, tdb_tickab.askprice10, tdb_tickab.askvolume10, tdb_tickab.bidprice10, tdb_tickab.bidvolume10, tdb_tickab.askaveprice, tdb_tickab.bidaveprice, tdb_tickab.totalaskvolume, tdb_tickab.totalbidvolume, tdb_tickab.open, tdb_tickab.high, tdb_tickab.low, tdb_tickab.preclose
   FROM hffd.tdb_tickab;

ALTER TABLE tdb_tickab
  OWNER TO fitl;
~~~

2.逐笔成交数据

~~~sql
-- Table: hffd.tdb_transaction

-- DROP TABLE hffd.tdb_transaction;

CREATE TABLE hffd.tdb_transaction
(
  id bigint NOT NULL DEFAULT nextval('hffd.tdb_transaction_seq'::regclass),
  windcode character(20),      --万得代码
  code character(20),          --交易所代码
  "date" date,                 --日期
  "time" integer,              --时间
  functioncode character(50),  --成交代码
  orderkind character(50),     --委托代码
  bsflag character(1),         --BS标志
  price numeric,               --成交价格
  volume integer,              --成交数量
  askorder integer,            --叫卖序号
  bidorder numeric             --叫买序号
)
WITH (APPENDONLY=true, COMPRESSLEVEL=1, ORIENTATION=column, COMPRESSTYPE=quicklz, 
  OIDS=FALSE
)
DISTRIBUTED BY (id)
PARTITION BY RANGE (date)
( START (date '2013-01-01') INCLUSIVE
   END (date '2017-01-01') EXCLUSIVE
   EVERY (INTERVAL '1 mon') );
ALTER TABLE hffd.tdb_transaction
  OWNER TO gpadmin;
GRANT ALL ON TABLE hffd.tdb_transaction TO gpadmin;
GRANT SELECT ON TABLE hffd.tdb_transaction TO fitl;

-- View: tdb_transaction

-- DROP VIEW tdb_transaction;

CREATE OR REPLACE VIEW tdb_transaction AS 
 SELECT tdb_transaction.id, tdb_transaction.windcode, tdb_transaction.code, tdb_transaction.date, tdb_transaction."time", tdb_transaction.functioncode, tdb_transaction.orderkind, tdb_transaction.bsflag, tdb_transaction.price, tdb_transaction.volume, tdb_transaction.askorder, tdb_transaction.bidorder
   FROM hffd.tdb_transaction;

ALTER TABLE tdb_transaction
  OWNER TO fitl;
~~~

3.委托队列

~~~sql
-- Table: hffd.tdb_orderqueue

-- DROP TABLE hffd.tdb_orderqueue;

CREATE TABLE hffd.tdb_orderqueue
(
  id bigint NOT NULL DEFAULT nextval('hffd.tdb_orderqueue_seq'::regclass),
  windcode character(20),  --万得代码
  code character(20),      --交易所代码
  date date,               --日期
  "time" numeric,          --时间
  side character(50),      --买卖方向
  price numeric,           --成交价格
  orderitems numeric,      --订单数量
  abitems numeric,         --明细个数
  abvolume numeric         --订单明细
)
WITH (APPENDONLY=true, COMPRESSLEVEL=1, ORIENTATION=column, COMPRESSTYPE=quicklz, 
  OIDS=FALSE
)
DISTRIBUTED BY (id)
PARTITION BY RANGE (date)
( START (date '2013-01-01') INCLUSIVE
   END (date '2017-01-01') EXCLUSIVE
   EVERY (INTERVAL '1 mon') );
ALTER TABLE hffd.tdb_orderqueue
  OWNER TO gpadmin;
GRANT ALL ON TABLE hffd.tdb_orderqueue TO gpadmin;
GRANT SELECT ON TABLE hffd.tdb_orderqueue TO fitl;

-- View: tdb_orderqueue

-- DROP VIEW tdb_orderqueue;

CREATE OR REPLACE VIEW tdb_orderqueue AS 
 SELECT tdb_orderqueue.id, tdb_orderqueue.windcode, tdb_orderqueue.code, tdb_orderqueue.date, tdb_orderqueue."time", tdb_orderqueue.side, tdb_orderqueue.price, tdb_orderqueue.orderitems, tdb_orderqueue.abitems, tdb_orderqueue.abvolume
   FROM hffd.tdb_orderqueue;

ALTER TABLE tdb_orderqueue
  OWNER TO fitl;
~~~

4.逐笔委托

~~~sql
-- Table: hffd.tdb_order

-- DROP TABLE hffd.tdb_order;

CREATE TABLE hffd.tdb_order
(
  id bigint NOT NULL DEFAULT nextval('hffd.tdb_order_seq'::regclass),
  windcode character(20),             --万得代码
  code character(20),                 --交易所代码
  date date,                          --日期
  "time" numeric,                     --时间
  ordernumber character varying(20),  --交易所委托号
  orderkind character(20),            --委托类型
  functioncode character(1),          --委托代码 B，S，C
  price numeric,                      --委托价格
  volume numeric                      --委托数量
)
WITH (APPENDONLY=true, COMPRESSLEVEL=1, ORIENTATION=column, COMPRESSTYPE=quicklz, 
  OIDS=FALSE
)
DISTRIBUTED BY (id)
PARTITION BY RANGE (date)
( START (date '2013-01-01') INCLUSIVE
   END (date '2017-01-01') EXCLUSIVE
   EVERY (INTERVAL '1 mon') );
ALTER TABLE hffd.tdb_order
  OWNER TO gpadmin;
GRANT ALL ON TABLE hffd.tdb_order TO gpadmin;
GRANT SELECT ON TABLE hffd.tdb_order TO fitl;

-- View: tdb_order

-- DROP VIEW tdb_order;

CREATE OR REPLACE VIEW tdb_order AS 
 SELECT tdb_order.id, tdb_order.windcode, tdb_order.code, tdb_order.date, tdb_order."time", tdb_order.ordernumber, tdb_order.orderkind, tdb_order.functioncode, tdb_order.price, tdb_order.volume, tdb_order.tdate
   FROM hffd.tdb_order;

ALTER TABLE tdb_order
  OWNER TO fitl;
GRANT ALL ON TABLE tdb_order TO fitl;
~~~

###导入数据库

* 使用gpload批量导入

1.含买卖盘的tickab

~~~yaml
VERSION: 1.0.0.1
DATABASE: fitl
USER: gpadmin
HOST: mdw
PORT: 5432
GPLOAD:
   INPUT:
    - SOURCE:
         LOCAL_HOSTNAME:
           - mdw
         PORT: 8097
         FILE:
           - /fitl/TDB_WIND/tickab/2015/csv/*
    - COLUMNS:
           - WindCode : text
           - Code : text
           - Date : date
           - Time : numeric
           - Price : numeric
           - Volume : numeric
           - Turover : numeric
           - MatchItem : numeric
           - Interest : numeric
           - TradeFlag : text
           - BSFlag : text
           - AccVolume : numeric
           - AccTurover : numeric
           - AskPrice10 : numeric
           - AskVolume10 : numeric
           - BidPrice10 : numeric
           - BidVolume10 : numeric
           - AskAvePrice : numeric
           - BidAvePrice : numeric
           - TotalAskVolume : numeric
           - TotalBidVolume : numeric
           - Open : numeric
           - High : numeric
           - Low : numeric
           - PreClose : numeric
    - FORMAT: csv
    - DELIMITER: ','
    - QUOTE: '"'
    - HEADER: false
    - ERROR_LIMIT: 2000
    - ERROR_TABLE: hffd.tdb_tickab_err
   OUTPUT:
    - TABLE: hffd.tdb_tickab
    - MODE: INSERT
~~~

2. transaction

~~~yaml
VERSION: 1.0.0.1
DATABASE: fitl
USER: gpadmin
HOST: mdw
PORT: 5432
GPLOAD:
   INPUT:
    - SOURCE:
         LOCAL_HOSTNAME:
           - mdw
         PORT: 9080
         FILE:
           - /fitl/TDB_WIND/transaction/2015/csv/*
    - COLUMNS:
           -  WindCode : text
           -  Code : text
           -  Date : date
           -  Time : numeric
           -  FunctionCode : text
           -  OrderKind : text
           -  BSFlag : text
           -  Price : float4
           -  Volume : int
           -  AskOrder : int
           -  BidOrder : numeric
    - FORMAT: csv
    - DELIMITER: ','
    - QUOTE: '"'
    - HEADER: false
    - ERROR_LIMIT: 5000
    - ERROR_TABLE: hffd.tdb_transaction_err
   OUTPUT:
    - TABLE: hffd.tdb_transaction
    - MODE: INSERT
~~~

3. orderqueue

~~~yaml
VERSION: 1.0.0.1
DATABASE: fitl
USER: gpadmin
HOST: mdw
PORT: 5432
GPLOAD:
   INPUT:
    - SOURCE:
         LOCAL_HOSTNAME:
           - mdw
         PORT: 9085
         FILE:
           - /fitl/TDB_WIND/orderqueue/2015/csv/*
    - COLUMNS:
           -  WindCode : text
           -  Code : text
           -  Date : date
           -  Time : numeric
           -  Side : text
           -  Price : numeric
           -  OrderItems : numeric
           -  ABItems : numeric
           -  ABVolume : numeric
    - FORMAT: csv
    - DELIMITER: ','
    - QUOTE: '"'
    - HEADER: false
    - ERROR_LIMIT: 5000
    - ERROR_TABLE: hffd.tdb_orderqueue_err
   OUTPUT:
    - TABLE: hffd.tdb_orderqueue
    - MODE: INSERT
~~~
4. order

~~~yaml
VERSION: 1.0.0.1
DATABASE: fitl
USER: gpadmin
HOST: mdw
PORT: 5432
GPLOAD:
   INPUT:
    - SOURCE:
         LOCAL_HOSTNAME:
           - mdw
         PORT: 8092
         FILE:
           - /fitl/TDB_WIND/order/2015/csv/*
    - COLUMNS:
         - WindCode : text
         - Code : text
         - Date : date
         - Time : numeric
         - OrderNumber : text
         - OrderKind : text
         - FunctionCode : text
         - Price : numeric
         - Volume : numeric
    - FORMAT: csv
    - DELIMITER: ','
    - QUOTE: '"'
    - HEADER: false
    - ERROR_LIMIT: 10000
    - ERROR_TABLE: hffd.tdb_order_err
   OUTPUT:
    - TABLE: hffd.tdb_order
    - MODE: INSERT
~~~