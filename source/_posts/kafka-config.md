---
title: Kafka 配置
date: 2020-10-15 22:50:15
tags: [Kafka]
categories: [学习笔记]
---

http://kafka.apache.org/documentation/#brokerconfigs

官方文档 http://kafka.apache.org/documentation/#brokerconfigs

<!--more-->

### Broker 配置

- `broker.id`

- `log.dirs`

- `zookeeper.connect`

  #### zookeeper.connect

  以如下形式指定ZooKeeper连接字符串，`hostname:port`其中host和port是ZooKeeper服务器的主机和端口。要允许在ZooKeeper机器关闭时通过其他ZooKeeper节点进行连接，您还可以在表单中指定多个主机`hostname1:port1,hostname2:port2,hostname3:port3`。
  服务器还可以在其ZooKeeper连接字符串中包含一个ZooKeeper chroot路径，该路径将其数据放在全局ZooKeeper命名空间中的某个路径下。例如，要给您一个chroot路径，`/chroot/path`请将连接字符串指定为`hostname1:port1,hostname2:port2,hostname3:port3/chroot/path`。

  |     类型： | 串   |
  | ---------: | ---- |
  |     默认： |      |
  |   有效值： |      |
  |   重要性： | 高   |
  | 更新模式： | 只读 |



#### advertised.host.name

已弃用：仅在未设置`advertised.listeners`或时使用`listeners`。使用`advertised.listeners`代替。
要发布给ZooKeeper的主机名，供客户端使用。在IaaS环境中，这可能需要与代理绑定的接口不同。如果未设置，它将使用`host.name`配置的值。否则，它将使用从java.net.InetAddress.getCanonicalHostName（）返回的值。

|     类型： | 串   |
| ---------: | ---- |
|     默认： | 空值 |
|   有效值： |      |
|   重要性： | 高   |
| 更新模式： | 只读 |



#### advertised.listeners


如果listeners与`listeners`config属性不同，则发布到ZooKeeper以便客户端使用的listeners。在IaaS环境中，这可能需要与代理绑定的接口不同。如果未设置，将使用`listeners`设置的值。`listeners`与之不同的是， advertise 0.0.0.0 地址无效。	

|     类型： | 串     |
| ---------: | ------ |
|     默认： | 空值   |
|   有效值： |        |
|   重要性： | 高     |
| 更新模式： | broker |



