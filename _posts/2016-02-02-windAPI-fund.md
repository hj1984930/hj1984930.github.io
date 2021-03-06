---
layout: post
title: 通过万得量化接口获取基金数据
description: ""
modified: 2016-02-02
tags: [wind, python]
image:
  feature: abstract-12.jpg
  
---

###通过万得量化接口获取基金数据

- - -

###环境说明

* wind量化客户端
* mysql5.5
* windows2008
* python2.7


###准备工作
mysql数据库中创建基金基本信息表

~~~sql
SET NAMES utf8;
SET FOREIGN_KEY_CHECKS = 0;

-- ----------------------------
--  Table structure for `t_fund_basic_info`
-- ----------------------------
DROP TABLE IF EXISTS `t_fund_basic_info`;
CREATE TABLE `t_fund_basic_info` (
  `fund_id` int(11) NOT NULL AUTO_INCREMENT,
  `FUND_CODE` varchar(20) DEFAULT NULL COMMENT '基金代码',
  `ASSETS_SCALE_STA_DATE` varchar(10) DEFAULT NULL COMMENT '资产规模统计日期',
  `FUND_TYPE_CODE` varchar(20) DEFAULT NULL COMMENT '基金类型代码',
  `FUND_INVERSTMENT_TYPE_1` varchar(20) DEFAULT NULL,
  `FUND_INVERSTMENT_TYPE_2` varchar(20) DEFAULT NULL,
  `FUND_NAME` varchar(100) DEFAULT NULL COMMENT '基金名称',
  `FUND_ABBR` varchar(40) DEFAULT NULL COMMENT '基金简称',
  `PUBLISH_DATE` varchar(50) DEFAULT NULL COMMENT '发行日期',
  `ESTABLE_DATE` varchar(50) DEFAULT NULL COMMENT '成立日期',
  `ESTABLE_SCALE` varchar(40) DEFAULT NULL COMMENT '成立规模',
  `ASSET_SCALE` varchar(40) DEFAULT NULL COMMENT '资产规模',
  `FUND_STATUS` varchar(20) DEFAULT NULL COMMENT '基金状态',
  `START_DATE` varchar(10) DEFAULT NULL COMMENT '开始日期',
  `END_DATE` varchar(10) DEFAULT NULL COMMENT '结束日期',
  `MAXIMUM_PURCHASE_RATE` decimal(38,18) DEFAULT NULL COMMENT '最高申购费率',
  `MAXIMUM_REDEMPTION_RATE` decimal(38,18) DEFAULT NULL COMMENT '最高赎回费率',
  `FUND_MINPURCHASEDDISCOUNT` decimal(38,18) DEFAULT NULL COMMENT '最低申购折扣',
  PRIMARY KEY (`fund_id`)
) ENGINE=InnoDB  DEFAULT CHARSET=utf8 COMMENT='基金基本信息表';

SET FOREIGN_KEY_CHECKS = 1;
~~~

###调用WIND PythonAPI

* 调用wind提供的pythonAPI，读取基金基本信息数据，并写入到mysql表
* 这里将股票型基金、混合型中的偏股型基金以及货币基金从成立之日开始至今的时间序列数据导入到数据库

~~~python
# -*- coding:utf-8 -*-
#  __author__ = "huangjie"

from WindPy import *
import MySQLdb
from datetime import datetime
import pandas as pd
from itertools import *
import sys
reload(sys)
sys.setdefaultencoding('utf-8')

host = "10.2.28.225"
db = "fund_index"
user = "root"
passwd = "suntimexxb"
param = []

def create_table():
    conn = MySQLdb.connect(host,user,passwd,db,charset="utf8")
    if conn:
        try:
            cursor = conn.cursor()
            cursor.execute("DROP TABLE IF EXISTS T_HUANGJIE")
            sql = """CREATE TABLE T_HUANGJIE (
            FUND_CODE  VARCHAR(20) NOT NULL,
            FUND_DATE  VARCHAR(50),
            FUND_NAME VARCHAR(50))"""
            cursor.execute(sql)
            conn.commit()
        finally:
            conn.close()

def insert_table(param):
    conn = MySQLdb.connect(host,user,passwd,db,charset="utf8")
    if conn:
        try:
            cursor = conn.cursor()
            sql = 'insert into t_huangjie(FUND_CODE,FUND_DATE) values(%s, %s, %s)'
            cursor.executemany(sql,param)
            conn.commit()
        except Exception as e:
            print e
            conn.rollback()
        finally:
            conn.close()


w.start()
dt=datetime.now()
try:
    conn = MySQLdb.connect(host,user,passwd,db,charset="utf8")
except MySQLdb.Error, e:
    try:
        sqlError =  "Error %d:%s" % (e.args[0], e.args[1])
    except IndexError:
        print "MySQL Error:%s" % str(e)

cursor = conn.cursor()
sql = "INSERT INTO t_fund_basic_info(FUND_CODE,ASSETS_SCALE_STA_DATE,FUND_TYPE_CODE,FUND_INVERSTMENT_TYPE_1,FUND_INVERSTMENT_TYPE_2,FUND_NAME,FUND_ABBR,PUBLISH_DATE,ESTABLE_DATE,ESTABLE_SCALE,ASSET_SCALE,MAXIMUM_PURCHASE_RATE,MAXIMUM_REDEMPTION_RATE,FUND_MINPURCHASEDDISCOUNT) VALUES (%s, %s, %s, %s, %s,%s, %s, %s, %s, %s, %s, %s, %s, %s)"



# 通过wset来取数据集数据
fund_codes = ['000008.OF','000042.OF','000051.OF','000059.OF','000082.OF','000176.OF','000248.OF','000309.OF','000311.OF','000312.OF','000313.OF','000368.OF','000373.OF','000376.OF','000409.OF','000411.OF','000418.OF','000457.OF','000471.OF','000478.OF','000513.OF','000524.OF','000549.OF','000577.OF','000586.OF','000592.OF','000594.OF','000596.OF','000613.OF','000628.OF','000656.OF','000688.OF','000696.OF','000697.OF','000711.OF','000729.OF','000746.OF','000751.OF','000756.OF','000761.OF','000778.OF','000780.OF','000793.OF','000803.OF','000820.OF','000821.OF','000826.OF','000827.OF','000828.OF','000831.OF','000835.OF','000854.OF','000866.OF','000867.OF','000884.OF','000893.OF','000913.OF','000916.OF','000925.OF','000942.OF','000950.OF','000955.OF','000960.OF','000961.OF','000962.OF','000968.OF','000971.OF','000974.OF','000975.OF','000978.OF','000979.OF','000985.OF','000991.OF','000996.OF','001008.OF','001009.OF','001015.OF','001016.OF','001027.OF','001028.OF','001036.OF','001039.OF','001040.OF','001042.OF','001043.OF','001044.OF','001047.OF','001048.OF','001050.OF','001051.OF','001052.OF','001054.OF','001064.OF','001070.OF','001072.OF','001097.OF','001104.OF','001105.OF','001113.OF','001126.OF','001133.OF','001149.OF','001158.OF','001162.OF','001163.OF','001166.OF','001167.OF','001171.OF','001178.OF','001180.OF','001186.OF','001188.OF','001193.OF','001195.OF','001208.OF','001214.OF','001223.OF','001230.OF','001236.OF','001237.OF','001241.OF','001242.OF','001243.OF','001245.OF','001277.OF','001291.OF','001313.OF','001319.OF','001351.OF','001361.OF','001396.OF','001397.OF','001404.OF','001409.OF','001410.OF','001416.OF','001420.OF','001421.OF','001426.OF','001455.OF','001458.OF','001459.OF','001460.OF','001469.OF','001473.OF','001476.OF','001482.OF','001490.OF','001496.OF','001521.OF','001528.OF','001539.OF','001541.OF','001542.OF','001548.OF','001549.OF','001550.OF','001551.OF','001552.OF','001553.OF','001554.OF','001555.OF','001556.OF','001557.OF','001558.OF','001559.OF','001560.OF','001561.OF','001577.OF','001583.OF','001586.OF','001587.OF','001588.OF','001589.OF','001590.OF','001591.OF','001592.OF','001593.OF','001594.OF','001595.OF','001599.OF','001600.OF','001605.OF','001611.OF','001612.OF','001616.OF','001617.OF','001618.OF','001626.OF','001628.OF','001629.OF','001630.OF','001631.OF','001632.OF','001637.OF','001638.OF','001643.OF','001644.OF','001645.OF','001651.OF','001663.OF','001672.OF','001677.OF','001692.OF','001705.OF','001713.OF','001714.OF','001717.OF','001718.OF','001719.OF','001726.OF','001736.OF','001766.OF','001781.OF','001849.OF','001877.OF','001884.OF','001899.OF','001915.OF','001938.OF','001956.OF','001975.OF','002168.OF','002199.OF','002210.OF','002236.OF','002300.OF','002310.OF','002311.OF','002315.OF','002316.OF','002334.OF','002335.OF','002385.OF','020011.OF','020021.OF','040002.OF','040180.OF','040190.OF','050002.OF','050013.OF','050021.OF','050024.OF','070023.OF','070030.OF','090010.OF','090012.OF','100032.OF','100038.OF','100053.OF','110003.OF','110019.OF','110020.OF','110021.OF','110022.OF','110026.OF','110030.OF','159901.OF','159902.OF','159903.OF','159905.OF','159906.OF','159907.OF','159908.OF','159909.OF','159910.OF','159911.OF','159912.OF','159913.OF','159915.OF','159916.OF','159918.OF','159919.OF','159921.OF','159922.OF','159923.OF','159924.OF','159925.OF','159927.OF','159928.OF','159929.OF','159930.OF','159931.OF','159932.OF','159933.OF','159935.OF','159936.OF','159938.OF','159939.OF','159940.OF','159942.OF','159943.OF','159944.OF','159945.OF','159946.OF','160119.OF','160127.OF','160133.OF','160135.OF','160136.OF','160137.OF','160218.OF','160219.OF','160221.OF','160222.OF','160224.OF','160225.OF','160415.OF','160417.OF','160418.OF','160419.OF','160420.OF','160516.OF','160517.OF','160615.OF','160616.OF','160620.OF','160625.OF','160626.OF','160628.OF','160629.OF','160630.OF','160631.OF','160632.OF','160633.OF','160634.OF','160635.OF','160636.OF','160637.OF','160638.OF','160639.OF','160640.OF','160706.OF','160716.OF','160806.OF','160807.OF','160808.OF','160809.OF','160814.OF','160919.OF','161017.OF','161022.OF','161024.OF','161025.OF','161026.OF','161027.OF','161028.OF','161029.OF','161030.OF','161031.OF','161032.OF','161033.OF','161118.OF','161121.OF','161122.OF','161123.OF','161207.OF','161211.OF','161213.OF','161217.OF','161223.OF','161227.OF','161507.OF','161604.OF','161607.OF','161612.OF','161613.OF','161628.OF','161629.OF','161630.OF','161715.OF','161718.OF','161720.OF','161721.OF','161723.OF','161724.OF','161725.OF','161726.OF','161811.OF','161812.OF','161816.OF','161819.OF','161825.OF','161832.OF','161833.OF','161907.OF','161910.OF','162010.OF','162107.OF','162208.OF','162213.OF','162216.OF','162307.OF','162412.OF','162413.OF','162509.OF','162510.OF','162711.OF','162714.OF','162907.OF','163109.OF','163110.OF','163111.OF','163113.OF','163114.OF','163115.OF','163116.OF','163117.OF','163118.OF','163209.OF','163407.OF','163808.OF','163821.OF','164304.OF','164401.OF','164402.OF','164403.OF','164508.OF','164809.OF','164811.OF','164818.OF','164819.OF','164820.OF','164821.OF','164905.OF','164907.OF','164908.OF','165309.OF','165310.OF','165312.OF','165315.OF','165316.OF','165511.OF','165515.OF','165519.OF','165520.OF','165521.OF','165522.OF','165523.OF','165524.OF','165525.OF','165707.OF','165806.OF','166007.OF','166802.OF','167301.OF','167503.OF','167601.OF','167901.OF','168001.OF','168201.OF','168202.OF','168203.OF','168204.OF','168205.OF','180003.OF','180033.OF','200002.OF','202015.OF','202017.OF','202021.OF','202025.OF','206005.OF','206010.OF','206012.OF','213010.OF','217016.OF','217017.OF','217019.OF','217027.OF','233010.OF','240014.OF','240016.OF','240019.OF','257060.OF','263001.OF','270010.OF','270026.OF','290010.OF','310318.OF','310398.OF','320010.OF','320014.OF','320022.OF','340006.OF','360001.OF','370023.OF','376510.OF','399001.OF','399011.OF','410008.OF','410010.OF','450008.OF','450009.OF','460220.OF','460300.OF','470007.OF','470068.OF','481009.OF','481012.OF','501002.OF','501005.OF','501006.OF','502000.OF','502003.OF','502006.OF','502010.OF','502013.OF','502016.OF','502020.OF','502023.OF','502026.OF','502030.OF','502036.OF','502040.OF','502048.OF','502053.OF','502056.OF','510010.OF','510020.OF','510030.OF','510050.OF','510060.OF','510070.OF','510090.OF','510110.OF','510120.OF','510130.OF','510150.OF','510160.OF','510170.OF','510180.OF','510190.OF','510210.OF','510220.OF','510230.OF','510260.OF','510270.OF','510280.OF','510290.OF','510300.OF','510310.OF','510330.OF','510360.OF','510410.OF','510420.OF','510430.OF','510440.OF','510450.OF','510500.OF','510510.OF','510520.OF','510560.OF','510580.OF','510610.OF','510620.OF','510630.OF','510650.OF','510660.OF','510680.OF','510710.OF','510880.OF','512010.OF','512070.OF','512110.OF','512120.OF','512210.OF','512220.OF','512230.OF','512300.OF','512310.OF','512330.OF','512340.OF','512500.OF','512510.OF','512600.OF','512610.OF','512640.OF','512990.OF','519027.OF','519032.OF','519034.OF','519100.OF','519116.OF','519117.OF','519180.OF','519195.OF','519300.OF','519606.OF','519671.OF','519673.OF','519677.OF','519686.OF','519706.OF','519714.OF','519965.OF','519975.OF','530010.OF','530015.OF','530018.OF','540006.OF','540007.OF','540008.OF','540009.OF','540010.OF','540012.OF','585001.OF','590007.OF','660008.OF','660011.OF','660014.OF','690008.OF','700002.OF','740101.OF','960000.OF','F050002.OF','000001.OF','000011.OF','000017.OF','000020.OF','000021.OF','000031.OF','000039.OF','000057.OF','000061.OF','000083.OF','000117.OF','000120.OF','000124.OF','000127.OF','000172.OF','000173.OF','000209.OF','000220.OF','000251.OF','000263.OF','000294.OF','000308.OF','000339.OF','000354.OF','000432.OF','000522.OF','000532.OF','000547.OF','000550.OF','000551.OF','000589.OF','000591.OF','000601.OF','000612.OF','000646.OF','000800.OF','000940.OF','000945.OF','000946.OF','000965.OF','001000.OF','001076.OF','001088.OF','001118.OF','001144.OF','001256.OF','001267.OF','001268.OF','001306.OF','001307.OF','001349.OF','001403.OF','001463.OF','001475.OF','001811.OF','001881.OF','001882.OF','001883.OF','001886.OF','001888.OF','001985.OF','002011.OF','002173.OF','002305.OF','020001.OF','020003.OF','020005.OF','020009.OF','020010.OF','020015.OF','020023.OF','020026.OF','040005.OF','040007.OF','040008.OF','040011.OF','040016.OF','040020.OF','040025.OF','040035.OF','050004.OF','050008.OF','050009.OF','050010.OF','050014.OF','050018.OF','050026.OF','070001.OF','070002.OF','070003.OF','070006.OF','070010.OF','070011.OF','070013.OF','070017.OF','070019.OF','070021.OF','070022.OF','070027.OF','070032.OF','070099.OF','080001.OF','080005.OF','080007.OF','080012.OF','080015.OF','090003.OF','090004.OF','090007.OF','090009.OF','090011.OF','090013.OF','090015.OF','090016.OF','090018.OF','090020.OF','100020.OF','100022.OF','100026.OF','100039.OF','100056.OF','100060.OF','110002.OF','110005.OF','110009.OF','110011.OF','110013.OF','110015.OF','110023.OF','110025.OF','110029.OF','112002.OF','121003.OF','121005.OF','121008.OF','151001.OF','160105.OF','160106.OF','160211.OF','160212.OF','160215.OF','160311.OF','160314.OF','160505.OF','160512.OF','160603.OF','160605.OF','160607.OF','160610.OF','160611.OF','160613.OF','160805.OF','160910.OF','160916.OF','160918.OF','161005.OF','161601.OF','161606.OF','161609.OF','161610.OF','161611.OF','161616.OF','161706.OF','161810.OF','161818.OF','161903.OF','162006.OF','162102.OF','162201.OF','162202.OF','162203.OF','162204.OF','162207.OF','162209.OF','162212.OF','162214.OF','162605.OF','162607.OF','162703.OF','163302.OF','163406.OF','163409.OF','163412.OF','163415.OF','163503.OF','163801.OF','163803.OF','163804.OF','163805.OF','163818.OF','163822.OF','165313.OF','165508.OF','165512.OF','165516.OF','166001.OF','166005.OF','166006.OF','166009.OF','166011.OF','168102.OF','180010.OF','180012.OF','180013.OF','180031.OF','200006.OF','200008.OF','200010.OF','200012.OF','200015.OF','202001.OF','202003.OF','202005.OF','202007.OF','202009.OF','202011.OF','202019.OF','206002.OF','206007.OF','206009.OF','210003.OF','210004.OF','210005.OF','210008.OF','210009.OF','213002.OF','213003.OF','213008.OF','217001.OF','217005.OF','217009.OF','217010.OF','217012.OF','217013.OF','229002.OF','233001.OF','233006.OF','233007.OF','233009.OF','233011.OF','233015.OF','240001.OF','240004.OF','240005.OF','240008.OF','240009.OF','240010.OF','240011.OF','240017.OF','240020.OF','240022.OF','257010.OF','257020.OF','257030.OF','257040.OF','257050.OF','257070.OF','260101.OF','260104.OF','260108.OF','260109.OF','260110.OF','260111.OF','260112.OF','260115.OF','260116.OF','260117.OF','270002.OF','270005.OF','270006.OF','270007.OF','270008.OF','270021.OF','270025.OF','270028.OF','270041.OF','270050.OF','288001.OF','288002.OF','290002.OF','290004.OF','290006.OF','290008.OF','290011.OF','290014.OF','310308.OF','310328.OF','310358.OF','310368.OF','310388.OF','320001.OF','320003.OF','320005.OF','320007.OF','320011.OF','320012.OF','320016.OF','340007.OF','350002.OF','350005.OF','350008.OF','360005.OF','360006.OF','360007.OF','360010.OF','360012.OF','360016.OF','370024.OF','370027.OF','373020.OF','375010.OF','377010.OF','377020.OF','377150.OF','377240.OF','377530.OF','378010.OF','379010.OF','398001.OF','398011.OF','398021.OF','398041.OF','398061.OF','400003.OF','400007.OF','400011.OF','400032.OF','410001.OF','410003.OF','410009.OF','420003.OF','420005.OF','450002.OF','450003.OF','450004.OF','450007.OF','450011.OF','460001.OF','460002.OF','460005.OF','460007.OF','460009.OF','470006.OF','470008.OF','470009.OF','470028.OF','470098.OF','481001.OF','481004.OF','481006.OF','481008.OF','481010.OF','481013.OF','481015.OF','481017.OF','483003.OF','510081.OF','519001.OF','519002.OF','519005.OF','519008.OF','519011.OF','519013.OF','519015.OF','519017.OF','519018.OF','519019.OF','519021.OF','519025.OF','519026.OF','519033.OF','519035.OF','519039.OF','519056.OF','519068.OF','519069.OF','519087.OF','519089.OF','519093.OF','519095.OF','519097.OF','519099.OF','519110.OF','519115.OF','519150.OF','519158.OF','519181.OF','519185.OF','519664.OF','519665.OF','519668.OF','519670.OF','519672.OF','519674.OF','519678.OF','519679.OF','519688.OF','519692.OF','519694.OF','519698.OF','519702.OF','519704.OF','519712.OF','519727.OF','519736.OF','519915.OF','519979.OF','519983.OF','519987.OF','519993.OF','519994.OF','519996.OF','530001.OF','530003.OF','530006.OF','530011.OF','530019.OF','540002.OF','540004.OF','550001.OF','550002.OF','550003.OF','550008.OF','550009.OF','560002.OF','560003.OF','570001.OF','570005.OF','570006.OF','570007.OF','570008.OF','580001.OF','580002.OF','580003.OF','580006.OF','580007.OF','580008.OF','582003.OF','590001.OF','590002.OF','590005.OF','590008.OF','610001.OF','610004.OF','610005.OF','610006.OF','610007.OF','620001.OF','620004.OF','620005.OF','620006.OF','620008.OF','630001.OF','630002.OF','630006.OF','630010.OF','630011.OF','660001.OF','660003.OF','660004.OF','660005.OF','660006.OF','660010.OF','660012.OF','660015.OF','671010.OF','688888.OF','690003.OF','690004.OF','690005.OF','690007.OF','700001.OF','700003.OF','710001.OF','730001.OF','730002.OF','740001.OF','960001.OF','960006.OF','960007.OF','960008.OF','960010.OF','960011.OF','960012.OF','960013.OF','960014.OF','960015.OF','960016.OF','960018.OF','960019.OF','960020.OF','F000172.OF','F050004.OF','F050010.OF','F050026.OF','F070001.OF','F070013.OF','J460002.OF','J481004.OF','J519110.OF','000009.OF','000010.OF','000013.OF','000037.OF','000038.OF','000089.OF','000090.OF','000132.OF','000133.OF','000198.OF','000203.OF','000204.OF','000210.OF','000211.OF','000300.OF','000301.OF','000322.OF','000323.OF','000324.OF','000325.OF','000330.OF','000331.OF','000332.OF','000343.OF','000353.OF','000359.OF','000366.OF','000371.OF','000379.OF','000380.OF','000381.OF','000389.OF','000397.OF','000424.OF','000425.OF','000434.OF','000439.OF','000464.OF','000475.OF','000476.OF','000483.OF','000484.OF','000485.OF','000486.OF','000487.OF','000488.OF','000493.OF','000494.OF','000495.OF','000505.OF','000506.OF','000509.OF','000528.OF','000533.OF','000539.OF','000540.OF','000542.OF','000543.OF','000559.OF','000560.OF','000569.OF','000575.OF','000576.OF','000580.OF','000581.OF','000588.OF','000599.OF','000600.OF','000602.OF','000604.OF','000605.OF','000607.OF','000615.OF','000618.OF','000620.OF','000621.OF','000625.OF','000626.OF','000627.OF','000638.OF','000640.OF','000641.OF','000642.OF','000644.OF','000645.OF','000647.OF','000648.OF','000650.OF','000651.OF','000657.OF','000658.OF','000659.OF','000660.OF','000661.OF','000662.OF','000665.OF','000677.OF','000678.OF','000681.OF','000682.OF','000683.OF','000685.OF','000686.OF','000687.OF','000693.OF','000699.OF','000700.OF','000701.OF','000704.OF','000705.OF','000707.OF','000709.OF','000710.OF','000712.OF','000713.OF','000715.OF','000716.OF','000719.OF','000721.OF','000722.OF','000724.OF','000725.OF','000726.OF','000730.OF','000734.OF','000735.OF','000738.OF','000740.OF','000741.OF','000748.OF','000750.OF','000758.OF','000759.OF','000760.OF','000764.OF','000771.OF','000773.OF','000779.OF','000783.OF','000784.OF','000785.OF','000786.OF','000787.OF','000789.OF','000790.OF','000791.OF','000797.OF','000808.OF','000809.OF','000816.OF','000818.OF','000819.OF','000832.OF','000836.OF','000837.OF','000846.OF','000847.OF','000848.OF','000855.OF','000856.OF','000857.OF','000860.OF','000861.OF','000862.OF','000863.OF','000868.OF','000869.OF','000871.OF','000872.OF','000882.OF','000883.OF','000891.OF','000895.OF','000901.OF','000902.OF','000903.OF','000905.OF','000907.OF','000908.OF','000912.OF','000917.OF','000918.OF','000919.OF','000920.OF','000921.OF','000922.OF','000923.OF','000980.OF','000981.OF','000982.OF','001006.OF','001010.OF','001026.OF','001038.OF','001041.OF','001057.OF','001058.OF','001077.OF','001078.OF','001094.OF','001095.OF','001096.OF','001134.OF','001175.OF','001176.OF','001177.OF','001211.OF','001232.OF','001233.OF','001234.OF','001240.OF','001251.OF','001308.OF','001374.OF','001386.OF','001391.OF','001401.OF','001477.OF','001478.OF','001497.OF','001516.OF','001526.OF','001527.OF','001529.OF','001621.OF','001624.OF','001625.OF','001666.OF','001669.OF','001693.OF','001697.OF','001698.OF','001699.OF','001787.OF','001788.OF','001812.OF','001820.OF','001821.OF','001826.OF','001842.OF','001843.OF','001867.OF','001870.OF','001871.OF','001893.OF','001894.OF','001895.OF','001909.OF','001916.OF','001925.OF','001926.OF','001929.OF','001930.OF','001931.OF','001937.OF','001973.OF','001981.OF','001982.OF','001987.OF','001991.OF','001992.OF','002077.OF','002078.OF','002183.OF','002184.OF','002185.OF','002195.OF','002200.OF','002201.OF','002202.OF','002240.OF','002241.OF','002247.OF','002248.OF','002260.OF','002318.OF','002324.OF','003003.OF','020007.OF','020031.OF','020032.OF','040003.OF','040028.OF','040029.OF','040030.OF','040031.OF','040033.OF','040034.OF','040038.OF','040039.OF','041003.OF','050003.OF','070008.OF','070028.OF','070029.OF','070035.OF','070036.OF','070088.OF','080011.OF','090005.OF','090021.OF','090022.OF','090023.OF','091005.OF','091021.OF','091022.OF','091023.OF','100025.OF','100028.OF','110006.OF','110016.OF','110050.OF','110051.OF','110052.OF','110053.OF','121011.OF','128011.OF','150005.OF','150015.OF','159001.OF','159002.OF','159003.OF','159004.OF','159005.OF','159006.OF','160606.OF','160609.OF','161608.OF','161615.OF','161622.OF','161623.OF','162206.OF','163802.OF','163820.OF','166014.OF','166015.OF','180008.OF','180009.OF','200003.OF','200103.OF','202301.OF','202302.OF','202303.OF','202304.OF','202305.OF','202306.OF','202307.OF','202308.OF','210012.OF','210013.OF','213009.OF','213909.OF','217004.OF','217014.OF','217025.OF','217026.OF','240006.OF','240007.OF','253050.OF','253051.OF','260102.OF','260202.OF','270004.OF','270014.OF','270046.OF','270047.OF','288101.OF','288201.OF','290001.OF','310338.OF','310339.OF','320002.OF','320019.OF','340005.OF','350004.OF','360003.OF','360019.OF','360020.OF','360021.OF','360022.OF','370010.OF','37001B.OF','380001.OF','380002.OF','380003.OF','380004.OF','380007.OF','380008.OF','380010.OF','380011.OF','392001.OF','392002.OF','400005.OF','400006.OF','410002.OF','420006.OF','420106.OF','460006.OF','460106.OF','470014.OF','470030.OF','470060.OF','471007.OF','471014.OF','471030.OF','471060.OF','472007.OF','482002.OF','485018.OF','485020.OF','485022.OF','485118.OF','485120.OF','485122.OF','511800.OF','511810.OF','511823.OF','511830.OF','511860.OF','511880.OF','511890.OF','511900.OF','511930.OF','511960.OF','511980.OF','511990.OF','519131.OF','519501.OF','519505.OF','519506.OF','519507.OF','519508.OF','519509.OF','519510.OF','519511.OF','519512.OF','519513.OF','519516.OF','519517.OF','519518.OF','519566.OF','519567.OF','519568.OF','519588.OF','519589.OF','519716.OF','519717.OF','519721.OF','519722.OF','519800.OF','519801.OF','519808.OF','519809.OF','519858.OF','519859.OF','519878.OF','519879.OF','519888.OF','519889.OF','519898.OF','519899.OF','519998.OF','519999.OF','530002.OF','530014.OF','530028.OF','530029.OF','530030.OF','531014.OF','531028.OF','531029.OF','531030.OF','540011.OF','541011.OF','550010.OF','550011.OF','550012.OF','550013.OF','560001.OF','583001.OF','583101.OF','620010.OF','620011.OF','630012.OF','630112.OF','660007.OF','660016.OF','660107.OF','660116.OF','690010.OF','690012.OF','690210.OF','690212.OF','710501.OF','710502.OF','730003.OF','730103.OF','740601.OF','740602.OF','750006.OF','750007.OF']

#create_table()
for j in range(0,len(fund_codes)):

    # 通过wsd来提取时间序列数据
    print "\n\n-----第 %i 次通过wsd来提取 %s 基金数据-----\n" %(j,str(fund_codes[j]))
    wssdata=w.wss(str(fund_codes[j]),'fund_setupdate')
    print(wssdata.Data[0])
    print(type(wssdata.Data[0][0]))
    if wssdata.Data[0][0] < datetime(1900,1,1,00,00,00,5000):
        wssdata.Data[0][0] = datetime(1991,1,1,00,00,00,5000)
    #wsddata1=w.wsd(str(fund_codes[j]), "trade_code,fund_type,fund_firstinvesttype,fund_investtype,fund_fullname,sec_name,issue_date,fund_setupdate,issueamount,unit_total,fund_pchredm_pchmaxfee,fund_pchredm_maxredmfee,fund_minpurchasediscounts", wssdata.Data[0][0], dt, "Fill=Previous")
    wsddata1=w.wsd(str(fund_codes[j]), "fund_type,fund_firstinvesttype,fund_investtype,fund_fullname,sec_name,issue_date,fund_setupdate,issueamount,unit_total,fund_pchredm_pchmaxfee,fund_pchredm_maxredmfee,fund_minpurchasediscounts", wssdata.Data[0][0], dt, "Fill=Previous")

    if wsddata1.ErrorCode!=0:
        continue
    print wsddata1
    for i in range(0,len(wsddata1.Data[0])):
        sqllist=[]
        sqltuple=()
        sqllist.append(str(fund_codes[j]))
        if len(wsddata1.Times)>1:
            sqllist.append(wsddata1.Times[i].strftime('%Y%m%d'))
        for k in range(0, len(wsddata1.Fields)):
            sqllist.append(wsddata1.Data[k][i])
        sqltuple=tuple(sqllist)
        print(sqltuple)
        cursor.execute(sql,sqltuple)
        conn.commit()
conn.close()
~~~


