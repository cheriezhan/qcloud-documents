﻿# 常见问题
## 日志通用问题
**1、TKE/EKS集群配置日志采集后，为什么在CLS控制台查看不到日志？**

发生日志查看不到或者缺失的情况可能是存在以下几点原因：
- **检查所选的日志topic是否开启了索引**，索引配置是使用日志服务进行检索分析的必要条件，若未开启，则无法查看日志，配置索引的操作请参照[日志服务配置索引](https://cloud.tencent.com/document/product/614/50922)。
![](https://main.qcloudimg.com/raw/0e2cb0b38733cf2099a1e269c51e04b0.png)

- **日志、审计、事件不可选用同一个topic**，会发生日志覆盖，造成缺失。
- **检查所选的topic是否选用了两种提取模式**，新的提取模式会覆盖掉原来的。
- **检查是否存在软链接**。如果选择采集类型为‘容器文件路径‘时，对应的‘容器文件路径’不能是一个软连接，这样会导致软连接的实际路径在采集器的容器内不存在，导致采集日志失败。
- 若采用环境变量的方式开启EKS日志采集，并选择角色授权，在**创建角色时需要选择CVM载体**，详细请参考[EKS日志采集](https://cloud.tencent.com/document/product/457/47200)。

若以上原因均坚决不了您的问题，请 [提交工单](https://console.cloud.tencent.com/workorder/category?level1_id=6&level2_id=350&source=0&data_title=%E5%AE%B9%E5%99%A8%E6%9C%8D%E5%8A%A1TKE&step=1)，选择【人工支持】或者【日志问题】>【立即创建】，进入创建工单信息填写页面。



**2、配置好日志规则后，日志在哪查看？**

点击进入[CLS控制台>检索页面](https://main.qcloudimg.com/raw/e7ae5dd20b35c615f225202d74918ec9.png)，选择想查看日志的日志集和日志主题，并开启全文索引，便可检索分析日志了。
![](https://main.qcloudimg.com/raw/e7ae5dd20b35c615f225202d74918ec9.png)

## 使用环境变量开启EKS日志采集时

**1、java类应用如何多行日志合并？**

当客户程序的日志数据跨占多行时（例如 Java 程序日志），在这种情况下，以换行符\n为日志的结束标识符就显得有些不合理，为了能让日志系统明确区分开每条日志，需要配置具有首行正则表达式的configmap去进行匹配，当某行日志匹配上预先设置的正则表达式，就认为是一条日志的开头，而下一个行首出现作为该条日志的结束标识符。具体请参考最佳实践


**2、如何调整日志采集配置，以适应不同的日志输出速率？**

当使用环境变量的方式开启EKS日志采集时，EKS会在pod sandbox里额外启动一个日志采集组件来进行日志收集并上报。由于EKS限制了日志采集组件的内存使用量，所以当日志的输出速率过高时，日志采集组件可能会发生OOM的情况，此时可以按需调整日志采集的配置。具体方法如下： 手动修改以下两个pod annotation，将日志采集组件使用的内存缓冲区调小，以减少内存占用。

```
internal.eks.tke.cloud.tencent.com/tail-buffer-chunk-size: "2M"
internal.eks.tke.cloud.tencent.com/tail-buffer-max-size: "2M"
```

两个annotation的含义如下，可参考详情[Fluent Bit](https://docs.fluentbit.io/manual/administration/configuring-fluent-bit/unit-sizes)

| 参数 | 含义 | 默认值 |
|---------|---------|---------|
| Buffer_Chunk_Size | 设置初始缓冲区大小以采集容器文件日志，也可用于增加缓冲区的大小 | 2M |
| Buffer_Max_Size | 设置每个受监视文件的缓冲区大小的限制。当需要增加缓冲区时（例如很长的日志），此值可以限制缓冲区可以增加多少。如果读取的文件超过此限制，将从监视列表中删除此文件 | 2M |


**3、EKS集群容器内应用输出日志的标准。**

在应用输出日志时需要注意，容器内应用尽可能的输出到stdout，如果您的程序日志是输出到容器的文件，就要有相应的手段来管理日志文件的滚动和清理，或者挂在volume持久化存储，否则20g临时空间会被占满（详细请参考[临时存储](https://cloud.tencent.com/document/product/457/39815)）。

若无管理日志文件的滚动和清理手段，推荐您进行一下操作，
- 挂载volume持久化存储，详细请参考[存储管理](https://cloud.tencent.com/document/product/457/46962)。
- 开启EKS日志采集，详细请参考[EKS日志采集](https://cloud.tencent.com/document/product/457/47200)。
