---
title: MvvmToolkit conflicts with PropertyChanged.Fody
published: 2024-10-18
description: ''
image: ''
tags: ['Dotnet', 'Wpf']
category: 'Develop'
draft: false
---

### 问题描述：在同时使用 `MvvmToolkit` 与 `PropertyChanged.Fody` 时，如果使用 `MvvmToolkit` 的  `OnPropertyNameChanged(T newValue)` 时，在使用VS调试时不会有任何问题，但是在打包时会出现 `PropertyChanged.Fody` 异常。
### 原因：`PropertyChanged.Fody` 有默认的 `On_PropertyName_Changed` 方法注入，但是注入的方法不支持只有一个参数前面的方法。详情：https://github.com/Fody/PropertyChanged/wiki/On_PropertyName_Changed#passing-old-and-new-value-to-the-on_propertyname_changed-method
### 解决方案：
1. 使用两个参数签名的方法。
2. 更改Fody 配置项`FodyWeavers.xml`，`InjectOnPropertyNameChanged='false'` 禁止注入属性变更方法。
```xml
<Weavers xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="FodyWeavers.xsd">
    <PropertyChanged InjectOnPropertyNameChanged='false'/>
</Weavers>
```
