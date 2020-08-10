## 操作场景
集群健康检查功能是容器服务为集群提供的各个资源的状态、运行情况检查服务，检查被告详细展示了组件、节点、工作负载的状态和配置的检查内容，若出现异常项，可进行异常详情描述，并自动分析异常级别、异常原因、异常影响和修复建议等。

## 主要检查项目
<table>

<thead>
<tr>
<th width="10%">检查类别</th>
<th width="30%">检查项</th>
<th width="40%">检查内容</th>
<th width="15%">仅独立集群</th>
</tr>
</thead>

<tbody>

<tr>
<td rowspan=10>资源状态</td>
<td> kube-apiserver的状态</td>
<td rowspan=7> 检测组件是否正在运行，如果组件以Pod形式运行，则检测其24小时内是否重启过。</td>
<td> 是</td>
</tr>

<tr>
<td> kube-scheduler的状态</td>
<td> 是</td>
</tr>

<tr>
<td> kube-controller-manager的状态</td>
<td> 是</td>
</tr>

<tr>
<td> etcd的状态</td>
<td> 是</td>
</tr>

<tr>
<td> kubelet的状态</td>
<td> 否</td>
</tr>

<tr>
<td> kube-proxy的状态</td>
<td> 否</td>
</tr>

<tr>
<td> dockerd的状态</td>
<td> 否</td>
</tr>

<tr>
<td> master节点的状态</td>
<td> 检测节点状态是否Ready且无其他异常情况，如内存不足，磁盘不足等。</td>
<td> 是</td>
</tr>

<tr>
<td> worker节点的状态</td>
<td> 检测节点状态是否Ready且无其他异常情况，如内存不足，磁盘不足等。</td>
<td> 否</td>
</tr>

<tr>
<td> 各个工作负载的状态</td>
<td> 检测工作负载当前可用Pod数是否符合其期望目标Pod数。</td>
<td> 否</td>
</tr>

<tr>
<td rowspan=14>运行情况</td>
<td> kube-apiserver的参数配置</td>
<td> 根据master节点配置检测以下参数<ul></ul>
1、max-requests-inflight: 给定时间内运行的变更类请求的最大值<ul></ul>
2、max-mutating-requests-inflight: 给定时间内运行的非变更类请求的最大值</td>
<td> 是</td>
</tr>

<tr>
<td> kube-scheduler的参数配置</td>
<td> 根据master节点配置检测以下参数<ul></ul>
1、kube-api-qps: 请求kube-apiserver使用的QPS<ul></ul>
2、kube-api-burst: 和kube-apiserver通信的时候最大burst值</td>
<td> 是</td>
</tr>

<tr>
<td> kube-controller-manager的参数配置</td>
<td> 根据master节点配置检测以下参数<ul></ul>
1、kube-api-qps: 请求kube-apiserver使用的QPS<ul></ul>
2、kube-api-burst: 和kube-apiserver通信的时候最大burst值</td>
<td> 是</td>
</tr>

<tr>
<td> etcd的参数配置</td>
<td> 根据master节点配置检测以下参数<ul></ul>
1、quota-backend-bytes: 存储大小</td>
<td> 是</td>
</tr>


<tr>
<td> master节点的配置合理性</td>
<td> 检测当前master节点配置是否足以支撑当前的集群规模。</td>
<td> 是</td>
</tr>

<tr>
<td> node高可用</td>
<td> 检测目前集群是否是单节点集群；<ul></ul>
检测当前集群节点是否支持多可用区容灾；<ul></ul>
即当一个可用区不可用后，其他可用区的资源总和是否足以支撑当前集群业务规模。</td>
<td> 否</td>
</tr>

<tr>
<td> 工作负载的Request 和 Limit 配置</td>
<td> 检测工作负载是否有未设置资源限制的容器，配置资源限制有益于完善资源规划、pod 调度、集群可用性等。</td>
<td> 否</td>
</tr>

<tr>
<td>工作负载的反亲和性配置</td>
<td> 检测工作负载是否配置了亲和性或者反亲和性，配置反亲和性有助于提高业务的高可用性。</td>
<td> 否</td>
</tr>

<tr>
<td> 工作负载的PDB 配置</td>
<td> 检测工作负载是否配置了 PDB，配置 PDB 可避免您的业务因驱逐操作而不可用。</td>
<td> 否</td>
</tr>

<tr>
<td> 工作负载的健康检查配置</td>
<td> 检测工作负载是否配置了健康检查，配置健康检查有助于发现业务异常。</td>
<td> 否</td>
</tr>

<tr>
<td> HPA-ip配置</td>
<td> 当前集群剩余的 Pod ip数目是否满足 HPA 扩容的最大数。</td>
<td> 否</td>
</tr>



</tr>
</tr>
</tbody></table>

## 操作步骤
1. 登录 [容器服务控制台](https://console.cloud.tencent.com/tke2)，选择左侧导航栏中的【健康检查】。
2. 有三种健康检查方式。
**批量检查**，适用于同时检查多个集群。
选择需要健康检查的集群，点击【批量检查】。如下图所示：
![](https://main.qcloudimg.com/raw/e782264955c22b49b0b615c925e902d1.png)
**立即检查**，适用于只检查一个集群。找到需要健康检查的集群，点击【立即检查】。如下图所示，
![](https://main.qcloudimg.com/raw/7e23d624112de08c13aa59b767933668.png)
**自动检查**，适用于需要周期性检查的集群。找到需要周期检查的集群，点击【自动检查】，其中可根据您的需求设置开启状态、检查周期和时间。如下图所示：
![](https://main.qcloudimg.com/raw/665b7b467a603d12a76b125c03ad00c4.png)
![](https://main.qcloudimg.com/raw/3a45748c37134b198634067c82843a8c.png)
3. 选择好检查方式之后，等待检查完成，检查进度可查看。
![](https://main.qcloudimg.com/raw/05c6cc587d7a323463af9fe0452a816a.png)
4. 检查完成后，可点击【查看结果】查看检查报告。
![](https://main.qcloudimg.com/raw/6b496f3e7106293cbda0bd6c7184dda2.png)
在检查报告页面，选择【资源状态】和【运行情况】分别查看资源状态和异常情况，点击【检查内容】可展示具体的检查内容，点击【异常】可查看异常级别、异常描述、异常原因、异常影响和修复建议。
![](https://main.qcloudimg.com/raw/a39ada0b3a653282d5de7162a0920225.png)


