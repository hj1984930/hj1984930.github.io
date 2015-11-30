---
layout: post
title: 期货行情数据gpload并行加载到Greenplum数据库
description: ""
modified: 2015-11-30
tags: [Greenplum,gpload]
image:
  feature: abstract-5.jpg
  
---

###期货行情数据gpload并行加载到Greenplum数据库
- - -


####Greenplum数据库创建期货行情表

创建一个序列，作为id

~~~sql
-- Sequence: hffd.tdb_futureab_seq

-- DROP SEQUENCE hffd.tdb_futureab_seq;

CREATE SEQUENCE hffd.tdb_futureab_seq
  INCREMENT 1
  MINVALUE 1
  MAXVALUE 9223372036854775807
  START 1
  CACHE 1;
ALTER TABLE hffd.tdb_futureab_seq
  OWNER TO gpadmin;
~~~

创建期货行情表

~~~sql
-- Table: hffd.tdb_futureab

-- DROP TABLE hffd.tdb_futureab;

CREATE TABLE hffd.tdb_futureab
(
  id integer NOT NULL DEFAULT nextval('hffd.tdb_futureab_seq'::regclass),
  windcode character(20),
  code character(20),
  date double precision,
  "time" double precision,
  volume double precision,
  turover double precision,
  settle double precision,
  "position" double precision,
  curdelta double precision,
  tradeflag character(50),
  accvolume double precision,
  accturover double precision,
  open double precision,
  high double precision,
  low double precision,
  price double precision,
  askprice double precision,
  askvolume double precision,
  bidprice double precision,
  bidvolume double precision,
  preclose double precision,
  presettle double precision,
  preposition double precision
)
WITH (
  OIDS=FALSE
)
DISTRIBUTED BY (id);
ALTER TABLE hffd.tdb_futureab
  OWNER TO gpadmin;
GRANT ALL ON TABLE hffd.tdb_futureab TO gpadmin;
GRANT SELECT ON TABLE hffd.tdb_futureab TO fitl;
~~~

####编写gpload配置文件

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
         PORT: 8090
         FILE:
           - /fitl/TDB_FURTURE/futureab/2015_csv/*.csv
    - COLUMNS:
           -  WindCode : text
           -  Code : text
           -  Date : float4
           -  Time : float4
           -  Volume : float4
           -  Turover : float4
           -  Settle : float4
           -  Position : float4
           -  CurDelta : float4
           -  TradeFlag : text
           -  AccVolume : float4
           -  AccTurover : float4
           -  Open : float4
           -  High : float4
           -  Low : float4
           -  Price : float4
           -  AskPrice : float4
           -  AskVolume : float4
           -  BidPrice : float4
           -  BidVolume : float4
           -  PreClose : float4
           -  PreSettle : float4
           -  PrePosition : float4
    - FORMAT: csv
    - DELIMITER: ','
    - QUOTE: '"'
    - HEADER: false
    - ERROR_LIMIT: 50
    - ERROR_TABLE: hffd.tdb_futureab_err
   OUTPUT:
    - TABLE: hffd.tdb_futureab
    - MODE: INSERT
~~~

####gpload并行加载数据

~~~bash
gpload -f futureab_2015.yml > 2015ab.log &
~~~

21分钟，完成1亿6千万条数据（2015年1月到10月的股指期货）的并行装载，速度还是挺快的

~~~
2015-11-30 10:11:31|INFO|gpload session started 2015-11-30 10:11:31
2015-11-30 10:11:32|INFO|started gpfdist -p 8090 -P 8091 -f "/fitl/TDB_FURTURE/futureab/2015_csv/*.csv" -t 30
2015-11-30 10:33:23|WARN|207 bad rows
2015-11-30 10:33:23|INFO|running time: 1312.12 seconds
2015-11-30 10:33:24|INFO|rows Inserted          = 167573149
2015-11-30 10:33:24|INFO|rows Updated           = 0
2015-11-30 10:33:24|INFO|data formatting errors = 0
2015-11-30 10:33:24|INFO|gpload succeeded with warnings
~~~