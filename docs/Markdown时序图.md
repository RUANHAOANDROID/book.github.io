---
title: Markdown 序列图应用
categories: -技术
tags: 
- Markdown 
- 画图 
date: 2022-01-04
---

# Markdown 序列图应用


## 句法

| 类型 | 描述                           |
| ---- | ------------------------------ |
| ->   | 没有箭头的实线                 |
| -->  | 没有箭头的虚线                 |
| ->>  | 带箭头的实线                   |
| -->> | 带箭头的虚线                   |
| -X   | 实线末尾有一个十字             |
| - X  | 末端带有十字的虚线。           |
| -)   | 末尾带有开放箭头的实线（异步） |
| --)  | 末尾带有开放箭头的虚线（异步） |


## 字符含义
> | **Opt**      | 选项   | 包含一个可能发生或可能不发生的序列。 可以在临界中指定序列发生的条件。 |
> | ------------ | ------ | ------------------------------------------------------------ |
> | **Alt**      | 抉择   | 包含一个片段列表，这些片段包含备选消息序列。 在任何场合下只发生一个序列。可以在每个片段中设置一个临界来指示该片段可以运行的条件。 **else** 的临界指示其他任何临界都不为 True 时应运行的片段。 如果所有临界都为 False 并且没有 **else**，则不执行任何片段。 |
> | **Loop**     | 循环   | 片段重复一定次数。 可以在临界中指示片段重复的条件。Loop 组合片段具有**“Min”**和**“Max”**属性，它们指示片段可以重复的最小和最大次数。 默认值是无限制。 |
> | **Break**    | 中断   | 如果执行此片段，则放弃序列的其余部分。 可以使用临界来指示发生中断的条件。 |
> | **Par**      | 并行   | 并行处理。 片段中的事件可以交错。                            |
> | **Critical** | 关键   | 用在 Par 或 Seq 片段中。 指示此片段中的消息不得与其他消息交错。 |
> | **Seq**      | 弱顺序 | 有两个或更多操作数片段。 涉及同一生命线的消息必须以片段的顺序发生。 如果消息涉及的生命线不同，来自不同片段的消息可能会并行交错。 |
> | **Strict**   | 强顺序 | 有两个或更多操作数片段。 这些片段必须按给定顺序发生。        |


## 示例
### 注册
```mermaid
Title:注册流程图
Note over 客户端: 填写用户信息
Note right of 客户端: 密码加密
客户端->服务端:  提交用户信息
Note over 服务端: 密码再次加密
服务端 -> 数据库 :用户入库
Note over 数据库: 校验该用户是否存在
数据库-->服务端:已存在
服务端 --> 客户端 :提示用户已经存在
数据库-->服务端:注册成功
服务端 --> 客户端 :返回用户信息（不返回密码信息）
```


### 登录

```mermaid
sequenceDiagram
    客户端-->服务端: 提交用户名密码
    Note over 服务端:校验用户信息合法性
    alt 验证失败
    	服务端-->客户端:提示用户登录失败
    else 验证成功
    	服务端-->客户端:返回TOKEN
    end
```


```mermaid
sequenceDiagram
    Alice->>Bob: Hello Bob, how are you?
    alt is sick
        Bob->>Alice: Not so good :(
    else is well
        Bob->>Alice: Feeling fresh like a daisy
    end
    opt Extra response
        Bob->>Alice: Thanks for asking
    end
```


## 参考
[sequence diagrams](https://mermaid-js.github.io/mermaid/#/sequenceDiagram)
[UML序列图总结Loop,Opt,Par和Alt](https://www.cnblogs.com/cy568searchx/p/6227238.html)