---
title: TridentNPL工作流
subtitle: 如何使用TridentNPL进行SDNFV网络管理
image: assets/img/trident/workflow.png
alt: Shirts on a hanger

caption:
  title: TridentNPL工作流
  subtitle: 如何使用TridentNPL进行SDNFV网络管理
  thumbnail: assets/img/trident/workflow-400x300.png
---

Trident高级网络控制编程的工作流程如图中所示可以分为9个步骤。其中1-5是Trident系统与用户、SDN控制器、NFV实例等外部组件的交互，6-9是Trident系统内部组件的交互过程。
{:.text-left}

### 定义流属性
{:.text-left}

Trident允许用户引入自定义的流属性对系统进行扩展。用户需要按照流属性的定义规范指定以下内容：
{:.text-left}

{:.text-left}

- 属性名称：一个字符串类型的值，表示属性的名称
- 属性类型：一个字符串类型的值，表示属性的类型。有效的值包括：
  - int：属性的值是一个整数类型的数字
  - float: 属性的值是一个浮点数类型的数字
  - string: 属性的值是一个字符串
  - boolean: 属性的值是一个布尔变量
- 属性流粒度：流粒度用字符串表示，表示属性对应的数据流聚合的粒度。有效的值包括
  - tcp-tuple: 该值表示属性的对象是由一个TCP五元组所定义的流
  - ipv4-pair: 该值表示属性的对象是由一对IP地址所定义的流
  - src-ipv4: 该值表示属性的对象是表示源主机的IP地址所定义的流
  - dst-ipv4: 该值表示属性的对象是表示目的主机的IP地址所定义的流

下面给出了Trident中定义一个流属性的例子：
{:.text-left}

~~~{.python}
attr1 = StreamAttribute("http-hostname", "string", "tcp-tuple")
~~~

这个实例定义了一个名称叫做“http-hostname”的属性，该属性的类型是字符串类型，其对应的流对象是由TCP五元组所定义的。
{:.text-left}

{:.list-inline}

- Date: January 2022
