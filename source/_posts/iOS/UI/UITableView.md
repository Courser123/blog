---
title: UITableView
date: 2021-12-29 17:11:49
tags:
categories: UI
---

### UITableView的重用机制

{% asset_img 0398244C-928A-412F-BEFE-6BA54BA5E0ED.png This is an image %}

#### UITableView的重用机制可以理解为四个部分

- 即将滚出可视区域的cell
- 可视区域显示的cell
- 即将滚入可视区域的cell
- 重用池

#### 重用机制

- 即将滚入可视区域的cell, 先去重用池中根据identifier寻找是否有可重用的cell, 有就复用, 没有就创建一个cell
- 即将滚出可视区域的cell, 在滚出可视区域后, 根据identifier去复用池查找是否存在这个cell, 没有就加入缓冲池(重用池)
- 如此滚动循环根据identifier完成缓冲池(复用池)的更新和cell的重用


### 数据源同步

#### 方案一 (并发访问, 数据拷贝)

{% asset_img 78636893-F9AA-4FEA-BA71-8DD4EB997002.png This is an image %}

① 先将数据拷贝一份(一般在主线程)
② 拷贝的数据传给子线程, 在子线程对数据进行处理(新数据请求、数据解析、预排版等)
③ 子线程处理新数据的同时, 在主线程对数据进行了删除/新增操作, 需要对删除/新增操作进行记录, 然后reloadUI
④ 子线程处理完数据之后, 同步删除/新增操作
⑤ 回到主线程reloadUI

#### 方案二 (串行访问)

{% asset_img D1A906F5-B600-440D-B9AE-FE416040FB07.png This is an image %}

① 子线程进行网络请求, 数据解析, 然后交给子线程的串行队列进行预排版
② 此时在主线程进行了删除/新增操作, 需要同步的在串行队列中进行处理, 如果此时串行队列中有任务正在进行, 会block直至队列中的任务处理完
③ 在串行队列中同步主线程的删除/新增操作, 回到主线程reloadUI

#### 取舍

```
方案一(并发访问, 数据拷贝): 需要记录操作, 且数据拷贝也需要额外的内存, 所以可能会有内存开销的问题
方案二(串行访问): 如果在子线程处理的任务特别耗时, 那么主线程的操作可能会有一定延迟
```

两种方案各有利弊, 根据实际情况选择哪种方案


### UITableView 中 cell 的动态高度计算

- cell 使用 AutoLayout 布局, 设置tableview 的 estimatedRowHeight 为一个非零值(预估高度, 不用太准确), rowHeight 设置为 UITableViewAutomaticDimension
- 手动计算每个控件的高度并相加, 然后缓存高度


### 如何对 UITableView 的滚动加载进行优化，防止卡顿

#### 减少 cellForRowAtIndexPath 代理中的计算量(cell的内容计算)

- 提前计算每个 cell 中需要的一些基本数据, 代理调用是直接使用

- 图片优化
    - 图片异步加载
    - 子线程预解码 ([主流图片加载库所使用的预解码究竟干了什么](https://dreampiggy.com/2019/01/18/%E4%B8%BB%E6%B5%81%E5%9B%BE%E7%89%87%E5%8A%A0%E8%BD%BD%E5%BA%93%E6%89%80%E4%BD%BF%E7%94%A8%E7%9A%84%E9%A2%84%E8%A7%A3%E7%A0%81%E7%A9%B6%E7%AB%9F%E5%B9%B2%E4%BA%86%E4%BB%80%E4%B9%88/))
    - 优化图片大小, 尽量避免动态缩放(contentMode), 必要时准备一张缩略图和一张高清图, 需要时再加载高清图
    - 尽可能将多张图片合成一张进行展示
    - 图片"懒加载"(延迟加载), 快速滚动时不频繁的请求/处理数据

#### 减少 heightForRowAtIndexPath 代理中的计算量(cell的高度计算)

- 如果 cell 的高度是固定的, 去掉 heightForRowAtIndexPath 的代理, 直接设置 UITableView 的 rowHeight 为固定高度

- 预计算 cell 的高度并缓存, heightForRowAtIndexPath 调用时直接设置