---
title: TridentNPL系统架构
subtitle: 介绍TridentNPL的系统架构和组件
image: assets/img/trident/architecture/architecture.png
alt: Shirts on a hanger

caption:
  title: TridentNPL系统架构
  subtitle: 介绍TridentNPL的系统架构和组件
  thumbnail: assets/img/trident/architecture/architecture-400-300.png
---

如上图所示，Trident 高级网络控制编程的核心系统由四个组件组成: 编译器，前端，分布式运行时（工作结点），流规则更新器。
{:.text-left}

### Trident编译器
{:.text-left}

Trident编译器采用Trident程序和相关的数据范式，并生成反应性表的规范TableSpec。TableSpec定义了反应表及其依赖关系，该反应表进一步分布到前端服务器和工作服务器 (分别在上图中的6和7)。编译过程如下图所示，源代码首先以静态单分配 (Static Single Assignment，SSA) 形式转换为中间格式，用于构造执行图。然后根据执行图轻松构建TableSpec。
{:.text-left}
<img src='assets/img/trident/architecture/compilation.png' width='100%'/>

### Trident前端
{:.text-left}
Trident系统有一到多个前端服务器。每个前端服务器可以提供两种服务: 
{:.text-left}
1. 键值API服务，它允许外部代理更新流属性或网络注释; 
2. 路由请求API服务，它允许构建在Trident之上的控制平台为给定流发出路由请求.
{:.text-left}

### Trident工作结点
{:.text-left}
Trident工作结点构成了系统的分布式反应式核心，它们使用Trident响应式更新协议管理响应表表并在集群中保持数据一致性。
{:.text-left}

### Trident流规则更新器
{:.text-left}
Trident流规则更新器负责根据路由决策生成相应的OpenFlow规则，并将其安装到交换机。它内部缓存数据平面中的当前流规则。在由路由请求或更新触发的新路由上，它将执行：
{:.text-left}
1. 检索受影响的路由。
2.  将它们映射到OpenFlow规则。
3. 将新规则与缓存的规则进行比较。
4. 使用 “先建后拆” 策略来安装流规则。在正确的流规则更新之后，安装在数据平面上的流规则应该是所有活动路由请求的最终路由的直接转换。
{:.text-left}

- Date: January 2022
