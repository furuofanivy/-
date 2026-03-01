---
title: "《每日网络知识》 —— 互联网控制报文协议（ICMP）：Ping和Traceroute的工作原理"
source: "https://mp.weixin.qq.com/s/KWVwzrXe2qrOcETOvMwC6g"
author:
  - "[[运维搬砖]]"
published:
created: 2026-03-01
description: "《每日网络知识》Week 4, Day 3 — 互联网控制报文协议（ICMP）：Ping和Traceroute的工作原理"
tags:
  - "clippings"
---
原创 运维搬砖 *2025年11月5日 12:01*

### Week 4, Day 3 — 互联网控制报文协议（ICMP）：Ping和Traceroute的工作原理

#### 一、技术原理

**1\. ICMP协议概述**

ICMP是IP协议的重要补充，用于在IP网络中传递 **控制消息** 和 **错误报告** 。它位于IP层，但通常被认为是网络层的一部分。

**ICMP的主要功能** ：

- • **网络可达性测试** （Ping）
- • **路径追踪** （Traceroute）
- • **错误报告** （目的地不可达、超时等）
- • **网络拥塞控制** （源站抑制）
- • **路由重定向**

**2\. ICMP报文格式**

```
0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|     类型     |     代码     |           校验和               |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                           数据部分                            |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

**关键字段** ：

- • **类型** ：ICMP消息的主要类别
- • **代码** ：同一类型中的不同子类别
- • **校验和** ：用于错误检测
- • **数据部分** ：包含具体信息，如引发错误的原始IP包头部

**常见ICMP类型** ：

| 类型 | 代码 | 描述 | 用途 |
| --- | --- | --- | --- |
| 0 | 0 | Echo Reply | Ping响应 |
| 3 | 0-15 | Destination Unreachable | 目的不可达 |
| 8 | 0 | Echo Request | Ping请求 |
| 11 | 0 | Time Exceeded | TTL超时 |
| 5 | 0-3 | Redirect | 路由重定向 |

**3\. Ping的工作原理**

Ping使用ICMP Echo Request和Echo Reply消息测试网络连通性：

**工作流程** ：

1. 1\. 源主机发送 **ICMP Echo Request** （类型8）到目标IP
2. 2\. 目标主机收到后回复 **ICMP Echo Reply** （类型0）
3. 3\. 源主机计算往返时间，统计丢包率

**Ping的输出示例** ：

```
$ ping google.com
PING google.com (142.251.42.206): 56 data bytes
64 bytes from 142.251.42.206: icmp_seq=0 ttl=116 time=12.345 ms
64 bytes from 142.251.42.206: icmp_seq=1 ttl=116 time=11.234 ms
64 bytes from 142.251.42.206: icmp_seq=2 ttl=116 time=13.456 ms

--- google.com ping statistics ---
3 packets transmitted, 3 packets received, 0.0% packet loss
round-trip min/avg/max/stddev = 11.234/12.345/13.456/0.987 ms
```

**4\. Traceroute的工作原理**

Traceroute通过巧妙利用IP包的 **TTL字段** 来发现路径上的每一跳路由器：

**工作流程** ：

1. 1\. 发送第一个数据包，TTL=1
- • 第一跳路由器将TTL减至0，丢弃包并回复 **ICMP Time Exceeded** 消息
	- • 源主机记录第一跳路由器的地址和响应时间
3. 2\. 发送第二个数据包，TTL=2
- • 第二跳路由器回复ICMP Time Exceeded
5. 3\. 重复此过程，逐步增加TTL，直到到达目标主机
6. 4\. 目标主机回复 **ICMP Echo Reply** （或端口不可达消息）

**Traceroute的输出示例** ：

```
$ traceroute google.com
traceroute to google.com (142.251.42.206), 64 hops max
 1  192.168.1.1  1.234 ms  1.123 ms  1.345 ms
 2  10.1.1.1     5.678 ms  6.789 ms  7.890 ms
 3  203.0.113.1  10.111 ms  11.222 ms  12.333 ms
 4  142.251.42.206  15.444 ms  16.555 ms  17.666 ms
```

#### 二、应用场景

**1\. 网络连通性测试**

- • **基础连通性** ：使用Ping测试设备是否在线
- • **网络质量** ：通过Ping的延迟和丢包率评估网络状况
- • **DNS解析** ：验证域名解析是否正确

**2\. 网络路径诊断**

- • **路由追踪** ：使用Traceroute定位网络故障点
- • **路径比较** ：比较不同时间或不同目标的路径变化
- • **运营商问题** ：识别跨运营商的路由问题

**3\. 网络监控**

- • **持续监控** ：定期Ping关键设备监控可用性
- • **性能基准** ：建立网络性能基准线
- • **故障预警** ：及时发现网络中断或性能下降

**4\. 安全检测**

- • **网络扫描** ：发现网络中的活跃主机
- • **防火墙测试** ：验证防火墙规则是否阻止ICMP
- • **拓扑发现** ：了解网络结构

#### 三、配置示例

**示例1：基本Ping命令使用**

```
# 基本ping（持续直到Ctrl+C）
ping 192.168.1.1

# 指定ping的次数
ping -c 5 google.com

# 指定数据包大小
ping -s 1000 192.168.1.1

# 指定时间间隔（秒）
ping -i 2 192.168.1.1

# Windows系统中的ping
ping -n 10 192.168.1.1
ping -l 1000 192.168.1.1
```

**示例2：高级Ping选项**

```
# 洪水ping（快速发送，需要root权限）
sudo ping -f 192.168.1.1

# 设置TTL值
ping -t 10 192.168.1.1

# 记录路由（在IP头中记录路径）
ping -R google.com

# 指定源接口或源IP
ping -I eth0 google.com
ping -S 192.168.1.100 google.com
```

**示例3：Traceroute命令使用**

```
# 基本traceroute
traceroute google.com

# 指定最大跳数
traceroute -m 30 google.com

# 不进行DNS反向解析（加快速度）
traceroute -n google.com

# 指定起始TTL
traceroute -f 5 google.com

# 使用ICMP而不是UDP（Windows风格）
traceroute -I google.com

# Windows系统中的tracert
tracert google.com
```

**示例4：在网络设备上配置ICMP相关功能**

**Cisco路由器 - 限制ICMP响应** ：

```
! 创建ACL限制对特定网络的ping
Router(config)# access-list 101 deny icmp any any echo
Router(config)# access-list 101 permit ip any any

! 应用ACL到接口
Router(config)# interface gigabitethernet 0/0
Router(config-if)# ip access-group 101 in
```

**Linux系统 - ICMP配置** ：

```
# 临时禁用ping响应
echo 1 > /proc/sys/net/ipv4/icmp_echo_ignore_all

# 启用ping响应
echo 0 > /proc/sys/net/ipv4/icmp_echo_ignore_all

# 永久配置（在/etc/sysctl.conf中）
net.ipv4.icmp_echo_ignore_all = 1
```

**示例5：使用Wireshark分析ICMP流量**

1. 1\. 打开Wireshark开始抓包
2. 2\. 在过滤器中输入： `icmp`
3. 3\. 在终端执行： `ping google.com`
4. 4\. 观察捕获的ICMP Echo Request和Echo Reply包
5. 5\. 分析每个包的Type、Code、Sequence等字段

#### 四、常见问题

**Q1：为什么有些Ping请求会超时？**  
A：可能的原因：

- • **目标主机离线** 或关机
- • **防火墙阻止** ICMP Echo Request
- • **中间路由器丢弃** 数据包
- • **网络拥塞** 导致丢包
- • **TTL过小** ，在到达目标前已超时

**Q2：Traceroute中的星号（\*）表示什么？**  
A：星号表示该跳路由器：

- • **未响应** ICMP Time Exceeded消息
- • **防火墙阻止** 了ICMP响应
- • **响应超时** 未在规定时间内收到回复
- • **负载均衡** 路径变化导致响应不匹配

**Q3：为什么Ping能通但网络应用无法使用？**  
A：这种情况表明：

- • **网络层连通** 正常（ICMP可通过）
- • **传输层或应用层** 有问题：
- • 目标端口被防火墙阻止
	- • 服务未运行或崩溃
	- • 应用层协议不匹配

**Q4：如何判断网络延迟是由哪一跳引起的？**  
A：使用Traceroute：

1. 1\. 比较连续跳数之间的延迟增加
2. 2\. 注意延迟显著增加的跳点
3. 3\. 结合多次测试结果，排除临时波动
4. 4\. 使用MTR（Matt's Traceroute）等工具进行持续监控

**Q5：ICMP有哪些安全风险？**  
A：ICMP可能被滥用于：

- • **网络侦察** ：发现活跃主机和网络拓扑
- • **拒绝服务** ：ICMP洪水攻击
- • **数据渗漏** ：通过ICMP隧道传输数据
- • **重定向攻击** ：恶意路由重定向

**防护措施** ：

- • 在边界防火墙限制不必要的ICMP类型
- • 在内网监控异常的ICMP流量模式
- • 对ICMP速率进行限制

**Q6：IPv6中的ICMP有什么不同？**  
A：IPv6使用 **ICMPv6** ，功能更强大：

- • 整合了IPv4中ICMP和IGMP的功能
- • 邻居发现替代ARP
- • 路径MTU发现更加重要
- • 多播监听发现功能

---

**今日总结：**  
ICMP是网络诊断不可或缺的工具，关键要点包括：

1. 1\. **ICMP角色** ：IP网络的"信使"，负责控制消息和错误报告
2. 2\. **Ping原理** ：基于ICMP Echo Request/Reply测试连通性
3. 3\. **Traceroute原理** ：利用TTL递减和ICMP Time Exceeded发现路径
4. 4\. **实用价值** ：网络排错、性能监控、拓扑发现
5. 5\. **安全考虑** ：合理配置ICMP过滤，平衡可用性与安全性

掌握ICMP协议和工具的使用，是网络工程师日常排错的基础技能。明天我们将探讨网关和代理的区别，理解网络流量转发的不同方式。
