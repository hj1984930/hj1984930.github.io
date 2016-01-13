---
layout: post
title: 沪深300指数计算(实时)
description: ""
modified: 2016-01-11
tags: [python, spring-xd]
image:
  feature: abstract-11.jpg
  
---

###沪深300指数计算(实时)

- - -

###环境说明
* SpringXD1.2.0
* Greenplum Database
* Centos6.6
* Python2.7
* kafka


###依赖环境配置

SpringXD安装见[SpringXD分布式安装部署](https://hj1984930.github.io/springxd-install/)

~~~
yum install postgresql-devel -y
~~~

~~~python
python get-pip.py
pip install psycopg2
pip install pandas
~~~

###计算流程说明

* 股票行情数据实时通过C++接口写入kafka消息队列
* SpringXD读取kafka的行情数据，经过python扩展脚本进行实时计算
* 计算结果写入文件、kafka和数据库

###SpringXD中创建stream

说明：该stream从kafka中读取股票实时行情数据，数据流进python模块，进行沪深300指数计算，计算结果写到文件中和kafka中

~~~
stream create --name hs300 --definition "kafka --zkconnect=10.2.29.4:2181 --topic=hjtest --outputType=text/plain | shell --command='/usr/local/bin/python /opt/hs300.py > /opt/test/hs300.log' --outputType=text/plain | file --dir=/opt/test/ --name=hs300" --deploy
~~~


###沪深300指数计算代码

说明：

~~~python
# coding=utf-8
# @huangjie
# 2016.1.13
# hs300.py
import sys
import psycopg2 as pg
import sys
import pandas.io.sql as psql
import pandas
import json
import time
import traceback
from pykafka import KafkaClient
import os

#=====================
# Write data to stdout
#=====================
def send(data):
  sys.stdout.write(data)
  sys.stdout.flush()

#=====================
# HS300 index
#=====================
def hs300(data):
  base_mcap = 12126502796.981754
  try:
    client = KafkaClient(hosts="10.2.29.54:9092, 10.2.29.115:9092")
    topic = client.topics['hs300']
  except:
    error_log = open("/opt/springXD_python_error.txt",'a')
    traceback.print_exc(file = error_log)
    error_log.flush()
    error_log.close()
  try:
  	conn = pg.connect(database="fitl", user="fitl", password="xxxx", host="10.2.28.234", port="5432")
  	cursor = conn.cursor()
  	dataframe = psql.read_sql("SELECT code,cnname,enname,market from hffd.hs300", conn)
  	dic = dataframe.to_dict('list')
  	hs300_list = dic['code']
  	df = psql.read_sql("SELECT tdate,code,lclose,tcap,mcap,cap,rate from hffd.hs300_sample_stocks",conn)
  	df_dic = df.to_dict('list')
  	df_index = df_dic['code']
  	df_300_data = pandas.DataFrame(df_dic,index=df_index,columns=['tdate','code','lclose','tcap','mcap','cap','rate'])
  	js_data = json.loads(data)
  	if js_data:
  	  code = js_data['code']
  	  code = eval(repr(code)[1:]).replace(" ","")
  	  if code in hs300_list:
  	  	df_300_data.set_value(code,'lclose',js_data['price'])
  	  	hs300_index = sum(df_300_data['cap'] * df_300_data['lclose'] * df_300_data['rate']/base_mcap)
  	  	hs300_index = round(hs300_index,2)
                with topic.get_sync_producer() as producer:
                  hs300_js = {'hs300':hs300_index}
                  hs300_js = json.dumps(hs300_js)
                  producer.produce(hs300_js)
                result = time.strftime('%Y-%m-%d %H:%M:%S',time.localtime(time.time())) + "  " + str(hs300_index)
                return result
  except:
    error_log = open("/opt/springXD_python_error.txt",'a')
    traceback.print_exc(file = error_log)
    error_log.flush()
    error_log.close()


#===========================================
# Terminate a message using the default CRLF
#===========================================
def eod():
  send("\r\n")

#===========================
# Main - Echo the input
#===========================

while True:
  try:
    data = raw_input()
    if data:
      result = hs300(data)
      send(result)
      eod()
  except EOFError:
      eod()
      break

~~~

###计算结果

~~~
2016-01-13 11:08:37  2562.83
2016-01-13 11:08:37  2562.83
2016-01-13 11:08:37  2562.79
2016-01-13 11:08:37  2562.83
2016-01-13 11:08:38  2562.73
2016-01-13 11:08:38  2562.6
2016-01-13 11:08:38  2562.61
2016-01-13 11:08:39  2562.6
2016-01-13 11:08:39  2562.69
2016-01-13 11:08:39  2562.68
2016-01-13 11:08:40  2562.7
2016-01-13 11:08:40  2562.71
2016-01-13 11:08:40  2562.68
2016-01-13 11:08:40  2562.68
2016-01-13 11:08:41  2562.7
2016-01-13 11:08:41  2562.85
2016-01-13 11:08:41  2562.84
2016-01-13 11:08:42  2562.83
2016-01-13 11:08:42  2562.98
2016-01-13 11:08:42  2563.0
2016-01-13 11:08:43  2562.99
2016-01-13 11:08:43  2562.99
2016-01-13 11:08:43  2563.04
2016-01-13 11:08:43  2563.02
2016-01-13 11:08:44  2563.01
2016-01-13 11:08:44  2562.68
2016-01-13 11:08:44  2562.71
2016-01-13 11:08:45  2562.61
2016-01-13 11:08:45  2562.66
2016-01-13 11:08:45  2562.63
2016-01-13 11:08:46  2562.58
~~~

