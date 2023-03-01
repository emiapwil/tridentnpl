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

使用Trident实现上述网络策略的示例代码如下：
{:.text-left}
```{.python}
if (pkt.sip in "10.0.0.0/16") and (pkt.dip in "10.0.0.0/16"){
  r = H :-:  H // case1
}else{
  iff(pkt.authentication == "success"){
    iff (pkt.is_bulk_transfer){
      r = H :-: Internet where (cap_gb == 40) > H :-: Internet
      // case 4
    }else{
       r = H :-: IDS :-: Internet where (cap_gb == 10)
       // case 3
    }
  }else{
    r = H :-: AAA where (cap_gb == 10) // case 2
  }
}
```
{:.text-left}

{:.list-inline}

- Date: January 2022
