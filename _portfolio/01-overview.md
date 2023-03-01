---
title: TridentNPL工作流
subtitle: 如何使用TridentNPL进行SDNFV网络管理
image: assets/img/trident/workflow.png
alt: TridentNPL工作流

caption:
  title: TridentNPL工作流
  subtitle: 如何使用TridentNPL进行SDNFV网络管理
  thumbnail: assets/img/trident/workflow-400x300.png
---

Trident 高级网络控制编程的工作流程如图中所示可以分为 9 个步骤。其中 1-5 是 Trident 系统与用户、SDN 控制器、NFV 实例等外部组件的交互，6-9 是 Trident 系统内部组件的交互过程。
{:.text-left}

### 定义流属性
{:.text-left}

Trident 允许用户引入自定义的流属性对系统进行扩展。用户需要按照流属性的定义规范指定以下内容：
{:.text-left}

{:.text-left}

- 属性名称：一个字符串类型的值，表示属性的名称
- 属性类型：一个字符串类型的值，表示属性的类型。有效的值包括：
  - int：属性的值是一个整数类型的数字
  - float: 属性的值是一个浮点数类型的数字
  - string: 属性的值是一个字符串
  - boolean: 属性的值是一个布尔变量
- 属性流粒度：流粒度用字符串表示，表示属性对应的数据流聚合的粒度。有效的值包括
  - tcp-tuple: 该值表示属性的对象是由一个 TCP 五元组所定义的流
  - ipv4-pair: 该值表示属性的对象是由一对 IP 地址所定义的流
  - src-ipv4: 该值表示属性的对象是表示源主机的 IP 地址所定义的流
  - dst-ipv4: 该值表示属性的对象是表示目的主机的 IP 地址所定义的流

Trident基于属性名称来唯一标识流属性，如果两个流属性具有相同的名称，则它们指向相同的流属性。此外，应该使用`完全限定域名（Fully Qual-ified Domain Name，FDQN）`以避免名称冲突。
{:.text-left}

下面给出了 Trident 中定义一个流属性的例子：
{:.text-left}

```{.python}
attr1 = StreamAttribute("http-hostname", "string", "tcp-tuple")
```

这个实例定义了一个名称叫做`http-hostname`的属性，该属性的类型是`字符串类型`，其对应的流对象是由 TCP 五元组所定义的。
{:.text-left}

本例中的`http-hostname`流属性可以通过一个独立的JSON数据模式文件来实现相同的定义效果，在`example-http-hostname.json`文件中，对`http-hostname`属性进行定义：
{:.text-left}
```{.json}
{
  "http-hostname":{
    "type":"string","stream-type":"tcp-tuple"}
}
```
{:.text-left}
在`example.tr`文件中读取`example-http-hostname.json`文件中的`http-hostname`属性：
{:.text-left}
```{.python}
attr2 = StreamAttribute("example-http-hostname.json","http-hostname")
```

在实践中，我们更推荐使用方式二定义流属性，通过将流属性的定义统一放在独立的数据文件中，可以更好地维护数据格式的统一性。
{:.text-left}

### 准备注解拓扑图
{:.text-left}
Trident将网络建模为带注释的拓扑图。网络运营商可以使用自定义属性标注设备、连接和端口，并根据这些自定义属性指定路由策略。同样，在使用前必须定义这些属性，下面的例子定义了名为`capacity-mb`的注解：
{:.text-left}
```{.python}
cap1 = NetworkAnnotation("capacity-mb","int","link")
```

注释通常由拓扑管理器组件或通过相关管理工具进行操作。在Trident中，只有`路由匹配(routeable matches)`的注解是必不可少的，此注释与所有外围端口 (即连接到主机或外部网络的端口) 相关联，并包含OpenFlow匹配的列表，表示可以通过给定端口传递匹配的数据包。



### 编写Trident程序
{:.text-left}
一旦正确指定了流属性和网络注释，就可以编写一个Trident程序来描述网络策略。我们通过Web鉴权的案例来演示Trident如何实现简单且细粒度的响应式网络策略。结合下图中的网络和如下网络策略：
- 内部主机无需身份验证即可相互连接。
- 内部主机必须被授权后才能访问互联网，如果鉴权失败则转发到鉴权服务器。
{:.text-left}
<img src='assets/img/trident/workflow/three-annotations-graph.png' width='100%'/>

采用Trident编写该网络策略：
{:.text-left}
```{.python}
auth = StreamAttribute("authenication","string","scr-ip")
H = Waypoint(role="host")
AAA = Waypoint(role="AAA")
Internet = Waypoint(role="Internet")

program(
  r1 = H :-: H
  iff(pkt.auth == "SUCCESS"){
    r2 =  H :-: Internet
  }else{
    r2 = H :-: AAA
  }
  bind(pkt,r1 > r2)
)
```
{:.text-left}

Trident程序由两个部分组成: 声明 (第2-5行) 和程序主体 (第7-15行)。在示例程序中，声明包含称为 “已认证” 的流属性的定义，该属性表示给定ip地址的认证状态。另外还有三种路由定义，是构造路由代数表达式的基本元素。坐标点是由不同注释 (例如 “角色”) 分类的一组外围端口。
{:.text-left}
策略首先定义了内部主机之间的路由 (第8行)，路由代数表达式表示所有以主机端口开头，以主机端口结尾的路径。内部主机到Internet (第9-13行) 之间的路由。使用条件定义语句，可以为经过身份验证的主机授予对互联网的访问权限 (第10行)，否则将流量重定向到身份验证服务器 (第12行)。在第14行中，该策略将两个路由意图与 “首选项” 运算符(>)结合在一起，以便Trident将在互联网路由之前尝试内部路由以获取新的路由请求。
{:.text-left}
### 发起路由请求
{:.text-left}

一旦程序正确编译并部署在Trident中，就可以开始发出路由请求。路由请求类似于OpenFlow PacketIn消息 ，其中包含用于确定数据包路由的基本字段。根据网络策略和可路由匹配的粒度，基本字段可能会有所不同。例如，在web身份验证示例中，路由请求仅需要源IP和目标ip地址，而在批量传输示例中，路由请求必须包括TCP 五元组。
{:.text-left}
一旦发出路由请求，要么通过PacketIn消息主动发出，要么通过代理主动发出，Trident会根据网络策略和当前属性值自动生成相应的路由。路由更新由Trident OpenFlow规则更新程序处理，该程序生成OpenFlow规则并将其发送到交换机。
{:.text-left}
### 更新流属性或网络注解
{:.text-left}
Trident不能单独工作: 它需要代理的帮助，以在数据包流经网络时报告参数更新，这通常需要额外的开发工作。幸运的是，由于已经开发了许多软件产品 (Bro，Snort等)来获取有关流量的有用信息，因此程序员通常只需要开发一个简单的扩展来使软件Trident兼容。
{:.text-left}
Trident允许网络运营商以很少的开销集成这些现有系统: 它仅要求代理通过RESTful 键-值API更新相应的数据。API如下图所示，采用三个参数: 表Id，键和值（第一行）。Trident检查键和值是否与表Id标识的流属性或网络注释数据架构匹配。例如，假设定义了称为is-bulk-transfer的流属性，以指示TCP连接是否属于批量传输作业（第三行），并且定义了称为丢失率的网络注释，以指示为链路测量的丢失率（第五行）。
{:.text-left}
<img src='assets/img/trident/workflow/restful-requests.png' width='100%'/>

- Date: January 2022
