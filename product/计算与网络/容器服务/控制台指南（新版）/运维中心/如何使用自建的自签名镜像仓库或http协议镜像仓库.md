## 如何使用自建的自签名镜像仓库或http协议镜像仓库
在弹性集群使用自建镜像仓库的镜像创建工作负载，可能会见到 ”**ErrImagePull**“ 报错，拉取镜像失败。

![](https://main.qcloudimg.com/raw/08d85768db0f556ab05a231a2e9205b6.png)

通常在保证网络连通性的前提下，有两个原因可能导致该问题：

1. 自建镜像仓库采用 https 协议，但是 https 协议证书是自签名的；
2. 自建镜像仓库采用的是 http 协议；

这两个问题都可以通过在工作负载 Yaml 配置里的 PodTemplate 中添加 Annotation 来解决。

## https自签名镜像仓库
如果自建的镜像仓库是https协议的，且证书是自签名的。需要在 PodTemplate 中，添加如下 Annotation，跳过证书检验：

**eks.tke.cloud.tencent.com/registry-insecure-skip-verify: 镜像仓库地址(多个用“,”隔开，或者填写all)**

![](https://main.qcloudimg.com/raw/7291dff6d8be0271c133530b4be49eb2.png)

>**说明：**
>
>如果 Pod 内多个容器的镜像从不同仓库拉取，可填写多个镜像仓库地址，中间用“,”隔开。或填写“all”表示，Pod 内所有容器镜像的相关仓库均跳过证书检验。

## http协议镜像仓库
因为默认弹性集群的运行时会使用 https 协议拉取镜像，所以如果镜像仓库支持的协议为 http ，则也需要通过 Annotation 进行说明。

通过命令 “$kubectl describe pod $podname”，发现输出 “**http: server gave HTTP response to HTTPS client**” 报错，说明所访问的镜像仓库使用了 http 协议。

![](https://main.qcloudimg.com/raw/903d96070e1db400a382342940faa227.png)

需要在 PodTemplate 中，添加如下 Annotation，使用 http 协议访问镜像仓库：

**eks.tke.cloud.tencent.com/registry-http-endpoint: 镜像仓库地址(多个用“,”隔开，或者填写all)**

![](https://main.qcloudimg.com/raw/25b198f03ae26ad1ac8d3be86361655c.png)

>**说明：**
>
>如果 Pod 内多个容器的镜像从不同仓库拉取，可填写多个镜像仓库地址，中间用“,”隔开。或填写“all”表示，Pod 内所有容器镜像均采用 http 协议拉取。

## 镜像仓库地址填写说明
以上两个 Annotation 均涉及镜像仓库地址的填写，多个仓库地址可用“,”隔开。

请注意，如果镜像仓库有端口号，需要带上端口号。

举例，如果镜像地址为 10.16.100.174:5000/busybox:latest，那么 Annotation 的 value 应该填为 “10.16.100.174:5000”，即 “eks.tke.cloud.tencent.com/registry-insecure-skip-verify: 10.16.100.174:5000”。
