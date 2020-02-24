# [时序数据的缓存设计 <br/> (Series data cache)](https://aiportal.github.io/series-data-cache/)

<br/>

**[Series data cache](https://aiportal.github.io/series-data-cache/) 是在 [ETag cache service](https://aiportal.github.io/etag-cache-service/) 的基础上，专为时序数据或海量数据所设计的缓存策略。**

**[Series data cache](https://aiportal.github.io/series-data-cache/) 将时序数据集按时间段分割成多个独立的资源单元，独立更新版本号，可以避免大面积缓存更新给系统带来的运算负载。**

<br/>

时序数据集因为数据量巨大，通常在查询条件中增加一个时间范围限定，以免遍历整个索引或数据集，造成超时响应或长时间等待。

在默认的 [ETag cache service](https://aiportal.github.io/etag-cache-service/) 缓存设计中，时序数据集的作为一个资源整体，每次发生资源更新操作 (INSERT, UPDATE, DELETE) 时都会造成所有相关缓存的整体更新，当数据集整体数量巨大时，缓存更新操作会给系统带来沉重的计算负荷。

要解决这个问题，可以将时序数据集按时间分段，把一个时序数据集当作多项资源，生成多项缓存，发生数据更新操作 (INSERT, UPDATE, DELETE) 时，也只更新与此时间段相关的缓存内容。

<br/>

## 举个栗子

* Order 表中存储了最近三年的订单数据，时序字段定义为 createdAt，分段跨度定义为一天。

* [Redis](https://redis.io) 中使用 [HSET](https://redis.io/commands/hset) 存储 Order 表所有时间段的资源版本信息。  
 (例如：`HSET order 20200220 6b7149833d3676e1fc38ce2ad733f640` )

* 服务端收到资源更新请求 (INSERT, UPDATE, DELETE) 时，根据被更新资源的 createdAt 字段，更新 [Redis](https://redis.io) 中相应的资源版本信息。

* 如果 2020年2月20日 (`20200220`) 这一天的数据有变化，就重新部署 2020年2月20日的缓存数据。

* 浏览器端请求 2020年2月20日 至 2020年2月22日 的数据时，服务端提取 `20200220`, `20200221`, `20200222` 这三天的缓存数据，合并后返回给浏览器端。

<br/>

## 时序数据的设计建议

好的时序数据设计应该有明确的“冷却”条件，“冷却”后的数据不可更改，这样可以避免缓存数据的“颠簸”。

![](./series-cool.png)

<br/>

## 非时序数据的缓存设计

非时序数据集可以选择一个能对数据集进行均匀或近似均匀划分的字段，仿照时序数据集设计缓存策略。

<br/>
<br/>
