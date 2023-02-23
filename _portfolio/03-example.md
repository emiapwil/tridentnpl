---
title: TridentNPL示例分析
subtitle: 用TridentNPL实现基于属性的网络控制示例
image: assets/img/trident/example.png
alt: Keep Exploring

caption:
  title: TridentNPL示例分析
  subtitle: 用TridentNPL实现基于属性的网络控制示例
  thumbnail: assets/img/trident/example-400x300.png
---

我们通过一个示例来展示如何使用TridentNPL在SDNFV网络中实现基于属性的网络控制。以上图为例，假设目标网络由三个核心路由器构成，其中连接到互联网的边界路由器上部署了一个验证服务器和一个旁路入侵检测系统，另外两个路由器连接了3个子网。
{:.text-left}

假设网络管理人员希望实现下面的网络策略：
{:.text-left}

{:.text-left}

- 内网中的主机之间可以无需验证互相访问
- 一个内网主机经过验证后可以访问互联网，否则流量将被重定向到验证服务器
- 普通内网主机在访问互联网时只能使用10 Gbps的链路，并且必须经过入侵检测系统
- 如果内网到互联网的流量属于一个大文件传输任务时，可以不经过入侵检测系统并且使用40 Gbps的链路


{:.list-inline}

- Date: January 2017
