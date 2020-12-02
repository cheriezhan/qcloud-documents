# 如何使用TKE审计/事件服务快速排查问题？

## 使用场景
TKE的集群审计和事件存储为用户配置了丰富的可视化图表，以多个维度对审计日志和集群事件进行呈现，操作简单且足以涵盖绝大多数常见集群运维场景， 让无论是发现问题还是定位问题都事半功倍，提升运维效率，真正将审计和事件数据的价值最大化 。
本文结合几个具体的使用场景和示例，介绍如何利用审计/事件仪表盘快速定位集群问题。

## 前提
集群已在控制台【功能管理】>【设置】中开启集群审计和事件存储，详情请参考[集群审计](https://cloud.tencent.com/document/product/457/48346)、[事件存储](https://cloud.tencent.com/document/product/457/32091)。

## 使用示例：

### 示例1: 排查一个工作负载消失的问题
在"审计检索"页面中，单击【K8S对象操作概览】标签，在过滤项中指定想要排查的操作类型和资源对象，如下图所示：
![](https://main.qcloudimg.com/raw/4c4ba2c916cca86f0c4879670e48cd34.png)
 

查询结果如下图所示：
![](https://main.qcloudimg.com/raw/ce597ecb9acb845743bdbbc3d45c76ef.png)


由图可见，是 "10001****7138"这个帐号，对应用「nginx」进行了删除。可根据帐号ID在【访问管理】>【用户列表】中查找关于此账号的详细信息。

### 示例2: 排查一个节点被封锁的问题
在"审计检索"页面中，单击【节点操作概览】标签，在过滤项中指定被封锁的节点IP名，如下图所示：
![](https://main.qcloudimg.com/raw/7864ec7140664ced4a3c0189b862e64d.png)


查询结果如下图所示：
![](https://main.qcloudimg.com/raw/b748fb55179eca69f88c658eed2d7202.png)
  

由图可见，是“10001****7138”这个帐号在2020-1-30T06:22:18时对"172.16.18.13"这台节点进行了封锁操作。

### 示例3: 排查apiserver响应变慢的问题
在"审计检索"的【聚合检索】标签页中，提供了从用户、操作类型、返回状态码等多个维度对于apiserver访问的趋势图。 
![](https://main.qcloudimg.com/raw/1dd7b23886fbac9da2cd0a8b1f9ee65e.png)
![](https://main.qcloudimg.com/raw/cae4ff8e36bdb9edded07031b2845002.png)
![](https://main.qcloudimg.com/raw/6130c71389d87c99f3bd653fde115904.png)

由图可见，用户"tke-kube-state-metrics"的访问量远高于其他用户，并且在“操作类型分布趋势”图中可以看出大多数都是list操作，在“状态码分布趋势”图中可以看出，状态码大多数为403，结合业务日志可知，由于RBAC鉴权问题导致“tke-kube-state-metrics”组件不停的请求apiserver重试，导致apiserver访问剧增。日志如下所示：

```
E1130 06:19:37.368981       1 reflector.go:156] pkg/mod/k8s.io/client-go@v0.0.0-20191109102209-3c0d1af94be5/tools/cache/reflector.go:108: Failed to list *v1.VolumeAttachment: volumeattachments.storage.k8s.io is forbidden: User "system:serviceaccount:kube-system:tke-kube-state-metrics" cannot list resource "volumeattachments" in API group "storage.k8s.io" at the cluster scope
```


### 示例4: 排查节点异常的问题

一台node节点出现异常，在“事件检索”页面，点击【事件总览】，在过滤项中输入异常节点IP，如下图所示：
![](https://main.qcloudimg.com/raw/35986b6c944125b46e98444f2c065448.png)


查询结果显示，有一条“节点磁盘空间不足”的事件记录查询结果如下图：
![](https://main.qcloudimg.com/raw/3b1518605191a867f511529b8b3621c0.png)


进一步查看异常事件趋势
![](https://main.qcloudimg.com/raw/c534743d20c99c710e0950b0c9c4a5b5.png)
![](https://main.qcloudimg.com/raw/0f5e005c700a6dc29a65cdc90cbc28b2.png)

可以发现，2020-11-25号开始，节点“172.16.18.13”由于磁盘空间不足导致节点异常，此后kubelet开始尝试驱逐节点上的pod以回收节点磁盘空间。


### 示例5: 查找触发节点扩容的原因
开启了节点池「弹性伸缩」的集群，CA（cluster-autoscler）组件会根据负载状况自动对集群中节点数量进行增减。如果集群中的节点发生了自动扩（缩）容，用户可通过事件检索对整个扩（缩）容过程进行回溯。

在“事件检索”页面，点击【全局检索】，输入以下检索命令：

```
event.source.component : "cluster-autoscaler"
```

在左侧隐藏字段中选择“event.reason”、“event.message”、“event.involvedObject.name”、"event.involvedObject.name"进行显示，将查询结果按照”日志时间“倒序排列，结果如下图所示：
![](https://main.qcloudimg.com/raw/a7d4649a541b40db5d3190b0dc049124.png)


由上图可见，通过事件可以看到节点扩容操作在‘2020-11-25 20:35:45’左右，分别由三个nginx pod(nginx-5dbf784b68-tq8rd/nginx-5dbf784b68-fpvbx/nginx-5dbf784b68-v9jv5)进行触发，最终扩增了三个节点，后续的扩容由于达到节点池的最大节点数没有再次触发。
