---
layout: post
title: Kafka集群的安装和部署
description: ""
modified: 2015-09-15
tags: [kafka, 大数据, 安装, 配置]
image:
  feature: abstract-7.jpg

---

###Kafka集群的安装和部署

####1. 安装部署zookeeper
因为已经在上一篇文章中部署了Pivtoal HD，所以zookeeper已经部署过了，这里可以直接使用；如果没有现成的zookeeper，也可以使用kafka自带的zookeeper。

####2. 下载kafka
从官方网站下载[kafka二进制安装包](https://www.apache.org/dyn/closer.cgi?path=/kafka/0.8.2.1/kafka_2.9.1-0.8.2.1.tgz)，解压缩kafka_2.9.1-0.8.2.1.tgz 并修改名称为kafka,存放于`/opt/kafka`

####3. 配置kafka的环境变量KAFKA_HOME、PATH

####4. 修改`conf/server.properties`配置

~~~shell
zookeeper.connect=phd3-m1.xxb.cn:2181,phd3-m1.xxb.cn:2181,phd3-m1.xxb.cn:2181
broker.id=1(其他两个机器是2，3)
host.name=kafka1
log.dirs=/opt/kafka/kafka-logs(文件夹权限为755)
~~~

####5. 然后复制kafka文件夹到集群内的其他机器上
注意修改host.name

####6. 启动kafka

~~~shell
nohup /opt/kafka/bin/kafka-server-start.sh /opt/kafka/config/server.properties &
~~~

####7. 创建topic
~~~shell
/opt/kafka/bin/kafka-topics.sh --create --zookeeper phd3-m1:2181 --replication-factor 1 --partitions 1 --topic price
/opt/kafka/bin/kafka-topics.sh --create --zookeeper phd3-m1:2181 --replication-factor 1 --partitions 1 --topic order
/opt/kafka/bin/kafka-topics.sh --create --zookeeper phd3-m1:2181 --replication-factor 1 --partitions 1 --topic orderqueue
/opt/kafka/bin/kafka-topics.sh --create --zookeeper phd3-m1:2181 --replication-factor 1 --partitions 1 --topic transaction
/opt/kafka/bin/kafka-topics.sh --create --zookeeper phd3-m1:2181 --replication-factor 1 --partitions 1 --topic product
~~~

####8. 监控kafka
这里使用的yahoo开源的kafka监控工具，先准备sbt环境（sbt是scala的打包构建工具）

~~~shell
curl https://bintray.com/sbt/rpm/rpm | sudo tee /etc/yum.repos.d/bintray-sbt-rpm.repo
yum install sbt
~~~

然后将源码clone下来

~~~shell
git clone https://github.com/yahoo/kafka-manager.git
~~~

编辑`conf/application.conf`配置zookeeper,`kafka-manager.zkhosts="phd3-m1:2181"`

使用sbt编译安装，编译后生成部署包，解压缩后，是一个play的应用，启动即可

~~~shell
cd /root/kafka-manager
sbt clean dist
cp /root/kafka-manager/target/universal/kafka-manager-1.2.5.zip /opt/
unzip kafka-manager-1.2.5.zip
cd /opt/kafka-manager-1.2.5/bin
./kafka-manager &
~~~
然后通过9000端口访问：
![kafka-manager界面](/images/kafka-manager.png)





