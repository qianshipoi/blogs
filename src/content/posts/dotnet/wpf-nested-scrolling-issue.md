---
title: WPF Nested scrolling issue
published: 2022-06-13
description: ''
image: ''
tags: ['Dotnet', 'WPF']
category: 'Develop'
draft: false
---

#### 当ListView嵌套ListView时，在内部ListView控件上使用鼠标滚轮会阻止事件传递到外层滚动事件。

#### 解决办法：

> ###### 在内部ListView 添加鼠标滚轮事件，将事件手动提交给父级元素。
>
> ```c#
> private void ListBox_PreviewMouseWheel(object sender, System.Windows.Input.MouseWheelEventArgs e)
>   {
>          if (!e.Handled)
>          {
>              e.Handled = true;
>
>              var eventArg = new MouseWheelEventArgs(e.MouseDevice, e.Timestamp, e.Delta);
>              eventArg.RoutedEvent = UIElement.MouseWheelEvent;
>              eventArg.Source = sender;
>              var parent = ((Control)sender).Parent as UIElement;
>              parent.RaiseEvent(eventArg);
>          }
>      }
> ```
