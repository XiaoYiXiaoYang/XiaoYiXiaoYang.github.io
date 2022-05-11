---
title: openflow protocol
date: 2022-05-11 20:58:39
tags: [Network]
categories: [Network]
---

## The OpenFlow Protocol

OpenFlow swicth规范的核心是用于OpenFlow协议消息的一组结构。

下面描述的结构、定义和枚举来自于include/openflow/openflow.h的文件，这是标准OpenFlow规范分布的一部分。所有结构都装有偏移和8字节对齐，由断言语句检查。所有的OpenFlow消息都以大端格式发送。


### OpenFlow Header

每个OpenFlow消息都以OpenFlow的标题开头：

```
/* Header on all OpenFlow packets. */
struct ofp_header {
uint8_t version; /* OFP_VERSION. */
uint8_t type; /* One of the OFPT_ constants. */
uint16_t length; /* Length including this ofp_header. */
uint32_t xid; /* Transaction id associated with this packet.
Replies use the same id as was in the request
to facilitate pairing. */
};
OFP_ASSERT(sizeof(struct ofp_header) == 8);
```

version指定了正在使用的OpenFlow协议版本。在OpenFlow协议的早期起草阶段，最重要的位被设置为表示一个实验版本。下面的位表示协议的修订号。当前版本为0x04。

length字段表示消息的总长度，因此不使用额外的框架来区分一帧和下一帧。该类型可以具有以下值：

```
enum ofp_type {
/* Immutable messages. */
OFPT_HELLO = 0, /* Symmetric message */
OFPT_ERROR = 1, /* Symmetric message */
OFPT_ECHO_REQUEST = 2, /* Symmetric message */
OFPT_ECHO_REPLY = 3, /* Symmetric message */
OFPT_EXPERIMENTER = 4, /* Symmetric message */
/* Switch configuration messages. */
OFPT_FEATURES_REQUEST = 5, /* Controller/switch message */
OFPT_FEATURES_REPLY = 6, /* Controller/switch message */
OFPT_GET_CONFIG_REQUEST = 7, /* Controller/switch message */
OFPT_GET_CONFIG_REPLY = 8, /* Controller/switch message */
OFPT_SET_CONFIG = 9, /* Controller/switch message */
/* Asynchronous messages. */
OFPT_PACKET_IN = 10, /* Async message */
OFPT_FLOW_REMOVED = 11, /* Async message */
OFPT_PORT_STATUS = 12, /* Async message */
/* Controller command messages. */
OFPT_PACKET_OUT = 13, /* Controller/switch message */
OFPT_FLOW_MOD = 14, /* Controller/switch message */
OFPT_GROUP_MOD = 15, /* Controller/switch message */
OFPT_PORT_MOD = 16, /* Controller/switch message */
OFPT_TABLE_MOD = 17, /* Controller/switch message */
/* Multipart messages. */
OFPT_MULTIPART_REQUEST = 18, /* Controller/switch message */
OFPT_MULTIPART_REPLY = 19, /* Controller/switch message */
/* Barrier messages. */
OFPT_BARRIER_REQUEST = 20, /* Controller/switch message */
OFPT_BARRIER_REPLY = 21, /* Controller/switch message */
/* Queue Configuration messages. */
OFPT_QUEUE_GET_CONFIG_REQUEST = 22, /* Controller/switch message */
OFPT_QUEUE_GET_CONFIG_REPLY = 23, /* Controller/switch message */
/* Controller role change request messages. */
OFPT_ROLE_REQUEST = 24, /* Controller/switch message */
OFPT_ROLE_REPLY = 25, /* Controller/switch message */
/* Asynchronous message configuration. */
OFPT_GET_ASYNC_REQUEST = 26, /* Controller/switch message */
OFPT_GET_ASYNC_REPLY = 27, /* Controller/switch message */
OFPT_SET_ASYNC = 28, /* Controller/switch message */
/* Meters and rate limiters configuration messages. */
OFPT_METER_MOD = 29, /* Controller/switch message */
};
```

### Common Structures

本节描述了多种消息类型所使用的结构。

#### Port Structures

OpenFlow pipeline通过端口接收和发送数据包。交换机可以定义物理端口和逻辑端口，而OpenFlow规范定义了一些保留端口（参见4.1）。

物理端口、交换机定义的逻辑端口和OFPP_LOCAL保留端口的结构描述如下:

```
/* Description of a port */
struct ofp_port {
uint32_t port_no;
uint8_t pad[4];
uint8_t hw_addr[OFP_ETH_ALEN];
uint8_t pad2[2]; /* Align to 64 bits. */
char name[OFP_MAX_PORT_NAME_LEN]; /* Null-terminated */
uint32_t config; /* Bitmap of OFPPC_* flags. */
uint32_t state; /* Bitmap of OFPPS_* flags. */
/* Bitmaps of OFPPF_* that describe features. All bits zeroed if
* unsupported or unavailable. */
uint32_t curr; /* Current features. */
uint32_t advertised; /* Features being advertised by the port. */
uint32_t supported; /* Features supported by the port. */
uint32_t peer; /* Features advertised by peer. */
uint32_t curr_speed; /* Current port bitrate in kbps. */
uint32_t max_speed; /* Max port bitrate in kbps */
};
OFP_ASSERT(sizeof(struct ofp_port) == 64);
```

port_no字段可唯一地标识交换机内的一个端口。hw_addr字段通常是该端口的MAC地址；OFP_MAX_ETH_ALEN为6。名称字段是一个以空结尾的字符串，其中包含接口的可读名称。OFP_MAX_PORT_NAME_LEN的值是16。

config字段描述了端口管理设置，其结构如下：

```
/* Flags to indicate behavior of the physical port. These flags are
* used in ofp_port to describe the current configuration. They are
* used in the ofp_port_mod message to configure the port’s behavior.
*/
enum ofp_port_config {
OFPPC_PORT_DOWN = 1 << 0, /* Port is administratively down. */
OFPPC_NO_RECV = 1 << 2, /* Drop all packets received by port. */
OFPPC_NO_FWD = 1 << 5, /* Drop packets forwarded to port. */
OFPPC_NO_PACKET_IN = 1 << 6 /* Do not send packet-in msgs for port. */
};
```

OFPPC_PORT_DOWN位表示端口已经被关闭，不应该被OpenFlow使用。OFPPC_NO_RECV位表示应该忽略在该端口上接收到的数据包。OFPPC_NO_FWD位表示OpenFlow不应该向该端口发送数据包。OFPPFL_NO_PACKET_IN位表示，该端口上生成table-miss的包永远不会触发发送给控制器的packet-in消息。

一般来说，端口配置位是由控制器设置的，而不是由交换机改变的。这些位可能对控制器实现诸如STP或BFD等协议很有用。如果交换机通过另一个管理接口更改了端口配置位，则交换机将发送一条OFPT_PORT_STATUS消息，通知控制器该更改。

state字段描述了端口的内部状态，其结构如下：

```
/* Current state of the physical port. These are not configurable from
* the controller.
*/
enum ofp_port_state {
OFPPS_LINK_DOWN = 1 << 0, /* No physical link present. */
OFPPS_BLOCKED = 1 << 1, /* Port is blocked */
OFPPS_LIVE = 1 << 2, /* Live for Fast Failover Group. */
};
```

端口状态位表示在OpenFlow之外的物理链路或交换机协议的状态

OFPPS_LINK_DOWN位表示物理link不存在。OFPPS_BLOCKED位表示是一个OpenFlow之外的交换机协议，如802.1D生成树，正在阻止使用OFPP_FLOOD使用该端口。

所有端口状态位都是只读的，控制器不能更改。当端口标志更改时，交换机发送OFPT_PORT_STATUS消息通知控制器更改。

端口号使用以下约定：

```
/* Port numbering. Ports are numbered starting from 1. */
enum ofp_port_no {
/* Maximum number of physical and logical switch ports. */
OFPP_MAX = 0xffffff00,
/* Reserved OpenFlow Port (fake output "ports"). */
OFPP_IN_PORT = 0xfffffff8, /* Send the packet out the input port. This
reserved port must be explicitly used
in order to send back out of the input
port. */
OFPP_TABLE = 0xfffffff9, /* Submit the packet to the first flow table
NB: This destination port can only be
used in packet-out messages. */
OFPP_NORMAL = 0xfffffffa, /* Process with normal L2/L3 switching. */
OFPP_FLOOD = 0xfffffffb, /* All physical ports in VLAN, except input
port and those blocked or link down. */
OFPP_ALL = 0xfffffffc, /* All physical ports except input port. */
OFPP_CONTROLLER = 0xfffffffd, /* Send to controller. */
OFPP_LOCAL = 0xfffffffe, /* Local openflow "port". */
OFPP_ANY = 0xffffffff /* Wildcard port used only for flow mod
(delete) and flow stats requests. Selects
all flows regardless of output port
(including flows with no output port). */
};
```

curr、advertised、supported和peer字段表示链路模式（速度和重复性）、链路类型（铜/光纤）和链路特性（自动协商和暂停）。端口特征由以下结构表示：

```
/* Features of ports available in a datapath. */
enum ofp_port_features {
OFPPF_10MB_HD = 1 << 0, /* 10 Mb half-duplex rate support. */
OFPPF_10MB_FD = 1 << 1, /* 10 Mb full-duplex rate support. */
OFPPF_100MB_HD = 1 << 2, /* 100 Mb half-duplex rate support. */
OFPPF_100MB_FD = 1 << 3, /* 100 Mb full-duplex rate support. */
OFPPF_1GB_HD = 1 << 4, /* 1 Gb half-duplex rate support. */
OFPPF_1GB_FD = 1 << 5, /* 1 Gb full-duplex rate support. */
OFPPF_10GB_FD = 1 << 6, /* 10 Gb full-duplex rate support. */
OFPPF_40GB_FD = 1 << 7, /* 40 Gb full-duplex rate support. */
OFPPF_100GB_FD = 1 << 8, /* 100 Gb full-duplex rate support. */
OFPPF_1TB_FD = 1 << 9, /* 1 Tb full-duplex rate support. */
OFPPF_OTHER = 1 << 10, /* Other rate, not in the list. */
OFPPF_COPPER = 1 << 11, /* Copper medium. */
OFPPF_FIBER = 1 << 12, /* Fiber medium. */
OFPPF_AUTONEG = 1 << 13, /* Auto-negotiation. */
OFPPF_PAUSE = 1 << 14, /* Pause. */
OFPPF_PAUSE_ASYM = 1 << 15 /* Asymmetric pause. */
};
```

可以同时设置多个这些标志。如果没有设置任何端口速度标志，则会使用max_speed或curr_speed。

curr_speed和max_speed字段表示链路的当前和最大比特率（原始传输速度），单位为kbps。数字应该舍入以匹配常用用法。例如，一个光学10Gb的以太网端口应该将此字段设置为10000000（而不是10312500），而一个OC-192端口应该将该字段设置为10000000（而不是9953280）。

max_speed字段表示链路的最大配置容量，而curr_speed则表示当前容量。如果端口有3个1Gb/s个容量的链接，LAG的一个端口关闭，一个端口以1Gb/s自动协商，1个端口以100Mb/s自动协商，max_speed为3Gb/s，curr_speed为1.1Gb/s。

#### Queue Structures

OpenFlow交换机通过一个简单的排队机制提供了有限的服务质量支持(QoS)。一个（或多个）队列可以附加到一个端口，并用于在该端口上映射流条目。映射到特定队列的流条目将根据该队列的配置（例如，分钟速率）进行处理。

一个队列由ofp_packet_queue结构来描述。

```
/* Full description for a queue. */
struct ofp_packet_queue {
uint32_t queue_id; /* id for the specific queue. */
uint32_t port; /* Port this queue is attached to. */
uint16_t len; /* Length in bytes of this queue desc. */
uint8_t pad[6]; /* 64-bit alignment. */
struct ofp_queue_prop_header properties[0]; /* List of properties. */
};
OFP_ASSERT(sizeof(struct ofp_packet_queue) == 16);
```

每个队列都由一组属性进一步描述，每个属性都具有特定的类型和配置

```
enum ofp_queue_properties {
OFPQT_MIN_RATE = 1, /* Minimum datarate guaranteed. */
OFPQT_MAX_RATE = 2, /* Maximum datarate. */
OFPQT_EXPERIMENTER = 0xffff /* Experimenter defined property. */
};
```

每个队列属性描述都以一个公共标头开头：

```
/* Common description for a queue. */
struct ofp_queue_prop_header {
uint16_t property; /* One of OFPQT_. */
uint16_t len; /* Length of property, including this header. */
uint8_t pad[4]; /* 64-bit alignemnt. */
};
OFP_ASSERT(sizeof(struct ofp_queue_prop_header) == 8);
```

最小速率队列属性使用以下结构和字段：

```
/* Min-Rate queue property description. */
struct ofp_queue_prop_min_rate {
struct ofp_queue_prop_header prop_header; /* prop: OFPQT_MIN, len: 16. */
uint16_t rate; /* In 1/10 of a percent; >1000 -> disabled. */
uint8_t pad[6]; /* 64-bit alignment */
};
OFP_ASSERT(sizeof(struct ofp_queue_prop_min_rate) == 16);
```

最大速率队列属性使用以下结构和字段：

```
/* Max-Rate queue property description. */
struct ofp_queue_prop_max_rate {
struct ofp_queue_prop_header prop_header; /* prop: OFPQT_MAX, len: 16. */
uint16_t rate; /* In 1/10 of a percent; >1000 -> disabled. */
uint8_t pad[6]; /* 64-bit alignment */
};
OFP_ASSERT(sizeof(struct ofp_queue_prop_max_rate) == 16);
```

实验者队列属性使用以下结构和字段：

```
/* Experimenter queue property description. */
struct ofp_queue_prop_experimenter {
struct ofp_queue_prop_header prop_header; /* prop: OFPQT_EXPERIMENTER, len: 16. */
uint32_t experimenter; /* Experimenter ID which takes the same
form as in struct
ofp_experimenter_header. */
uint8_t pad[4]; /* 64-bit alignment */
uint8_t data[0]; /* Experimenter defined data. */
};
OFP_ASSERT(sizeof(struct ofp_queue_prop_experimenter) == 16);
```

实验者队列属性体的其余部分不被标准的OpenFlow处理所解释，并由相应的实验者任意定义。


#### Flow Match Structures

OpenFlow匹配由一个流匹配头和一个零或多个流匹配字段序列组成。

##### Flow Match Header

流匹配标头由ofp_match结构描述：

```
/* Fields to match against flows */
struct ofp_match {
uint16_t type; /* One of OFPMT_* */
uint16_t length; /* Length of ofp_match (excluding padding) */
/* Followed by:
* - Exactly (length - 4) (possibly 0) bytes containing OXM TLVs, then
* - Exactly ((length + 7)/8*8 - length) (between 0 and 7) bytes of
* all-zero bytes
* In summary, ofp_match is padded as needed, to make its overall size
* a multiple of 8, to preserve alignement in structures using it.
*/
uint8_t oxm_fields[4]; /* OXMs start here - Make compiler happy */
};
OFP_ASSERT(sizeof(struct ofp_match) == 8);
```

type字段设置为OFPMT_OXM，length字段设置为包含所有匹配字段的ofp_match结构的实际长度。OpenFlow匹配的有效负载是一组OXMFlow匹配字段。

```
/* The match type indicates the match structure (set of fields that compose the
* match) in use. The match type is placed in the type field at the beginning
* of all match structures. The "OpenFlow Extensible Match" type corresponds
* to OXM TLV format described below and must be supported by all OpenFlow
* switches. Extensions that define other match types may be published on the
* ONF wiki. Support for extensions is optional.
*/
enum ofp_match_type {
OFPMT_STANDARD = 0, /* Deprecated. */
OFPMT_OXM = 1, /* OpenFlow Extensible Match */
};
```

此规范中唯一有效的匹配类型是OFPMT_OXM，它不反对使用OpenFlow1.1匹配类型OFPMT_STANDARD。如果使用替代匹配类型，匹配字段和有效负载可能设置不同，但这超出了本规范的范围。

##### Flow Match Field Structures

流匹配字段使用OpenFlow可扩展匹配(OXM)格式进行描述，这是一种紧凑的类型-长度-值(TLV)格式。每个OXMTLV长5到259（包括）字节。OXMtlv没有对齐到任何多字节边界上或没有填充。OXMTLV的前4个字节是它的头，后面是条目目的主体

OXMTLV的头按网络字节顺序解释为32位单词（见图4）。
![](openflow-2/2022-05-11-22-44-01.png)

OXMTLV的报头字段在表9中都有定义。
oxm_class是一个包含相关匹配类型的OXM匹配类，在a.2.3.3节中也有描述。oxm_field是一个特定于类的值，它标识匹配类中的匹配类型之一。oxm_class和oxm_field（报头中最重要的23位）的组合被统称为oxm_type。oxm_type通常指定一个协议报头字段，如以太网类型，但它也可以引用数据包元数据，如数据包到达的交换机端口。

![](openflow-2/2022-05-11-22-46-09.png)

oxm_hasmask定义了OXMTLV是否包含一个位掩码，更多的细节将在A.2.3.5节中解释。

oxm_length是一个正整数，以字节表示OXMTLV有效负载的长度。OXMTLV的长度，包括标头，正好是4+oxm_length字节

对于给定的oxm_class、oxm_field和oxm_hasmask值，oxm_length是一个常数。它只包括允许软件最低限度地解析未知类型的OXMtlv。(类似地，对于一个给定的oxm_class、oxm_field和oxm_length，oxm_hasmask是一个常数。)

##### OXM classes

#####  Flow Matching

##### Flow Match Field Masking

#####  Flow Match Field Prerequisite


#####  Flow Match Fields

##### Experimenter Flow Match Fields


#### Flow Instruction Structures