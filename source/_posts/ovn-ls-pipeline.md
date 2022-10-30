---
title: 浅析ovn LS流表设计
tags:
  - OVN
  - LS
categories:
  - Network
top: false
cover: false
toc: true
mathjax: true
date: 2022-09-27 22:41:49
img: /medias/files/ovn.jpg
summary: 基本的二层流表分析
password:
---

OVN通过引入逻辑交换机(Logical Switch)、逻辑交换机端口(Logical Switch Port)来组成虚拟网络拓扑。

二层包含的拓扑主要是同主机和跨主机两种。用户在页面通过lsp将虚拟机连接到同一个ls；其对应数据面，则是虚拟机的网口通过ovs port连接到ovs，每个主机上的ovs为其提供了虚拟网络功能。
```
(host1)vm1 - lsp1 - ls1
(host1)vm2 - lsp2 -
(host2)vm3 - lsp3 -
```




**参考：**
https://www.ovn.org/support/dist-docs/ovn-architecture.7.pdf
https://www.cnblogs.com/laolieren/p/ovn-architecture.html