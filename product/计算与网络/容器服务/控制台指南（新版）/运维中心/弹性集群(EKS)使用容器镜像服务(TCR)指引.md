## 弹性集群(EKS)使用容器镜像服务(TCR)指引
### 前提1. 确保已选择对应的镜像访问凭证
容器镜像默认都是私有的，故请在创建工作负载时，选择实例对应的镜像访问凭证。

![](https://main.qcloudimg.com/raw/bb53c701afb9f86351b7d00c7176e081.png)

若不清楚如何创建镜像访问凭证，可以在【弹性集群】-->【命名空间】界面上，【新建】“命名空间”，然后选择自动下发访问凭证。这样在新建的命名空间上，就可以选到镜像的访问凭证了。

![](https://main.qcloudimg.com/raw/435feca4a391af10781ae22046fba887.png)

### 前提2. 确保弹性集群到镜像服务网络打通
弹性集群到容器镜像服务间的网络，默认是不通的，所以拉取镜像时还是会报 “dial tcp x.x.x.x:443: i/o timeout”类似网络不通的错误。

处理方式有两种（推荐使用第一种）：

1. 内网访问方式：容器镜像服务新建内网访问链路，并配置内网域名解析，弹性集群通过新建的内网链路访问容器镜像服务；
2. 外网访问方式：弹性集群开启外网访问，通过公网访问容器镜像服务，同时容器镜像服务也需允许公网访问；

#### 2.1. 内网访问方式（推荐）
步骤如下：
##### 2.1.1 新建内网访问链路
新建内网访问链路，打通弹性集群所在子网至容器镜像服务实例。

![](https://main.qcloudimg.com/raw/34ad65fe337b47f18a1cb97b000f0257.png)

##### 2.1.2 开启域名内网解析
容器镜像服务使用的域名为“<tcr-name>.tencentcloudcr.com”，此域名在vpc内的解析需额外开通。若不开通，“<tcr-name>.tencentcloudcr.com”的域名依旧解析成公网IP，无法解析成内网IP，内网访问会失败。

>**注意**：新建链路后，需要等待后台生成内网解析IP（大约一分钟），IP生成后，下面的按钮才能开启。

![](https://main.qcloudimg.com/raw/f20fb1ef5ff40b3e1e717ddc027afeef.png)

#### 2.2 外网访问方式
步骤如下：
##### 2.2.1 确保容器镜像服务开启公网访问
确保对应的容器镜像服务实例已开启公网访问。

![](https://main.qcloudimg.com/raw/b54946da58af5db83b7087b8b83905e1.png)

测试体验阶段，可将公网IP地址段设置为：0.0.0.0/0。后续运行阶段，可将步骤2.2.2里涉及的NAT 网关出口的弹性IP添加到访问白名单。
##### 2.2.2 为弹性集群开启公网访问
弹性集群默认不开通外网访问，需通过 NAT 网关才能外网访问，步骤可参见[通过 NAT 网关访问外网](https://cloud.tencent.com/document/product/457/48710)。

配置好 NAT 网关，将弹性集群所在子网关联至NAT 网关的路由表后， 并确保 NAT 网关出口的弹性IP在访问容器镜像服务的白名单里（**请参见2.2.1**），弹性集群就可正常访问容器镜像服务，从公网拉取镜像了。

### 3. 常见报错指引
#### 3.1. Error: failed to do request:Head "xxxx/manifests/late-st": dial tcp xxx:443: i/o timeout

含有“**443: i/o timeout**”报错：大部分情况，是因为EKS到TCR网络不通。
**请参见上述前提2**，选择其中一种访问方式，打通EKS到TCR的网络。注意：“<tcr-name>.tencentcloudcr.com”的域名默认会解析成公网IP的，报错时请注意识别 “dial tcp xxx” 里的ip地址是公网还是内网，然后按情况处理。

#### 3.2. Error: code = Unknown, pull access denied, repository does not exist or may require authorization: server message: insufficient_scope: authorization failed

含有“**insufficient_scope: authorization failed**”报错：说明EKS到TCR网络已经开通，权限不足。可能原因有：命名空间不存在、密钥不正确、密钥不适用正在拉取的镜像。

#### 3.3. Error:  code = NotFound, failed to resolve reference "xxx:xxx":  not found

含有“**not found**”报错：说明镜像不存在。

#### 3.4. 其他
请参考[TCR 常见问题指引](https://cloud.tencent.com/document/product/1141/39292)。


