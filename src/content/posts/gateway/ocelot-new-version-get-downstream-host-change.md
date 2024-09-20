---
title: ocelot 新版本获取下游服务地址变更
published: 2024-09-20
description: ''
image: ''
tags: ['Gateway']
category: 'Develop'
draft: false
---

错误原因：`Ocelot` 新版本在 `Consul` 获取服务地址时默认返回的是节点名称 `node != null ? node.Name : entry.Service.Address`

解决方案：继承默认的 `DefaultConsulServiceBuilder`，重写 `GetDownstreamHost`方法。

> [Service Discovery — Ocelot 23.3 documentation](https://ocelot.readthedocs.io/en/latest/features/servicediscovery.html#consul-service-builder)
>
> [After Upgrade to 23.3.3 from 23.2.2 cluster name is being used in place of service address · Issue #2109 · ThreeMammals/Ocelot (github.com)](https://github.com/ThreeMammals/Ocelot/issues/2109)

```c#
public class ConsulServiceBuilder : DefaultConsulServiceBuilder
{
    public ConsulServiceBuilder(Func<ConsulRegistryConfiguration> configurationFactory, IConsulClientFactory clientFactory, IOcelotLoggerFactory loggerFactory) : base(configurationFactory, clientFactory, loggerFactory)
    {
    }

    protected override string GetDownstreamHost(ServiceEntry entry, Node node)
    {
        return entry.Service.Address;
    }
}

```



