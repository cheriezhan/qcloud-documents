# 使用环境变量的方式开启EKS日志采集如何实现多行日志合并
# 使用场景
当使用环境变量去配置EKS日志采集时，默认使用单行提取模式。但当客户程序的日志数据跨占多行时（例如 Java 程序日志），在这种情况下，以换行符\n为日志的结束标识符就显得有些不合理，为了能让日志系统明确区分开每条日志，需要配置具有首行正则表达式的configmap去进行匹配，当某行日志匹配上预先设置的正则表达式，就认为是一条日志的开头，而下一个行首出现作为该条日志的结束标识符。

# 实现效果
假设您的原始日志为，

```
2020-09-24 16:09:07 ERROR System.out(4844) java.lang.NullPointerException
at com.temp.ttscancel.MainActivity.onCreate(MainActivity.java:43)
at android.app.Activity.performCreate(Activity.java:5248)
at android.app.Instrumentation.callActivityOnCreate(Instrumentation.java:1110) at android.app.ActivityThread.performLaunchActivity(ActivityThread.java:2162) at android.app.ActivityThread.handleLaunchActivity(ActivityThread.java:2257)
at android.app.ActivityThread.access$800(ActivityThread.java:139)
at android.app.ActivityThread$H.handleMessage(ActivityThread.java:1210)
```
该条日志最终会被结构化处理为，

```
2020-09-24 16:09:07 ERROR System.out(4844) java.lang.NullPointerException \at com.temp.ttscancel.MainActivity.onCreate(MainActivity.java:43) \at android.app.Activity.performCreate(Activity.java:5248) \at android.app.Instrumentation.callActivityOnCreate(Instrumentation.java:1110) \at android.app.ActivityThread.performLaunchActivity(ActivityThread.java:2162) \at android.app.ActivityThread.handleLaunchActivity(ActivityThread.java:2257) \at android.app.ActivityThread.access$800(ActivityThread.java:139) \at android.app.ActivityThread$H.handleMessage(ActivityThread.java:1210)
```
# 操作步骤
## 创建Configmap
针对以上所展示的日志文件，parser.conf文件内容为，

```
apiVersion: v1 
data:
  parser.conf: |- 
    [PARSER]
    Name parser_name
    Format regex
    Regex ^(?<timestamp>[0-9]{2,4}\-[0-9]{1,2}\-[0-9]{1,2} [0-9]{1,2}\:[0-9]{1,2}\:[0-9]{1,2}) (?<message>.*) 
kind: ConfigMap
metadata:
  name: cm 
  namespace: default
```

其中的 parser_name 需要在创建工作负载的时候设置在annotation中：
spec.template.metadata.annotations 
eks.tke.cloud.tencent.com/parser-name: parser_name
关于更多parser.conf情况可参考[Regular Expression](https://docs.fluentbit.io/manual/pipeline/parsers/regular-expression)

## 创建Pod时挂载Configmap
在创建pod时需要做一下操作，
- 以Volume形式挂载上面创建的Configmap
- 通过环境变量的方式开启日志采集
- 指定两个annotation，eks.tke.cloud.tencent.com/parser-name 是指上面所创建的Configmap的name，eks.tke.cloud.tencent.com/volume-name-for-parser 是指pod中挂载的volume名称，可自己定义。

```
eks.tke.cloud.tencent.com/parser-name: "parser_name"
eks.tke.cloud.tencent.com/volume-name-for-parser: "volume-name"
```
具体Pod yaml模版如下：

```
apiVersion: apps/v1 
kind: Deployment 
metadata:
  labels:
    k8s-app: multiline 
    qcloud-app: multiline
      name: multiline
      namespace: default 
spec:
  replicas: 1 
  selector:
    matchLabels:
    k8s-app: multiline
    qcloud-app: multiline 
template:
  metadata: 
      annotations:
        eks.tke.cloud.tencent.com/parser-name: parser_name
        eks.tke.cloud.tencent.com/volume-name-for-parser: volume-name 
      labels:
        k8s-app: multiline
        qcloud-app: multiline 
  spec:
		containers: 
		- env:
			- name: EKS_LOGS_OUTPUT_TYPE 
			value: cls
			- name: EKS_LOGS_LOG_PATHS 
			value: stdout
			- name: EKS_LOGS_TOPIC_ID 
			value: topic-id
			- name: EKS_LOGS_LOGSET_NAME 
			value: eks
			- name: EKS_LOGS_SECRET_ID 
			valueFrom:
				secretKeyRef: 
					key: SecretId 
					name: cls 
					optional: false
			- name: EKS_LOGS_SECRET_KEY 
			valueFrom:
				secretKeyRef: 
						key: SecretKey 
						name: cls 
						optional: false
image: nginx 
imagePullPolicy: Always 
name: ng
resources:
limits:
	cpu: 500m 
	memory: 1Gi
requests:
	cpu: 250m 
	memory: 256Mi
volumeMounts:
	- mountPath: /mnt
	name: volume-name 
imagePullSecrets:
- name: qcloudregistrykey 
restartPolicy: Always 
volumes:
- configMap:
		defaultMode: 420
		name: cm
name: volume-name
```
