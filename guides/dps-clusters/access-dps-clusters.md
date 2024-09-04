# 访问 DPS 集群
MAXIR 提供了多种访问集群的方式，以满足不同的用户需求。默认情况下通过内网链接，同时也提供外网访问的解决方案。


## 内网访问
**方式一：使用 postgres client psql 访问**

在与MAXIR处于同一VPC下的云主机上，安装postgres client psql 版本。建议最佳为12版本。用户名操作请参考 [用户管理](/maxir/guides/dw-users/manage-dwusers) 。

```plain
psql -h {maxir地址} -U {maxir控制台创建的用户名} -d ${maxir_database}
```

**方式二：使用云平台提供的“DWSU控制台”访问**

通过云平台的“DWSU控制台”访问集群时，默认使用内网进行连接。如需外网访问，具体操作请参考。

操作流程：
>1. 登录控制台集群[列表页](https://console.ucloud.cn/maxir/standard)
>2. 找到目标集群，点击操作列的“DWSU控制台”按钮
>3. 在新的登录页，输入用户名、密码登录

![](/images/guides/optimization/1.jpg)

![](/images/guides/optimization/2.jpg)

## 外网访问
MAXIR 的外网访问解决方案依托于 UCloud 的负载均衡产品 ALB，ALB 提供高效且安全的外网负载均衡服务，帮助您高效管理和优化外部流量。

1.创建负载均衡：登录 [ALB 控制台](https://console.ucloud.cn/ulb/alb)，点击“创建负载均衡”按钮，创建实例，计费参考 [ALB 产品定价](https://docs.ucloud.cn/ulb/alb/buy/charge)。

![](/images/guides/optimization/3.jpg)

![](/images/guides/optimization/4.jpg)

2.进入监听管理页面：在[控制台列表页](https://console.ucloud.cn/ulb/alb)找到目标实例，点击“监听器管理”按钮，进行监听管理页面。

![](/images/guides/optimization/5.jpg)

3.创建监听器：点击“添加”按钮，创建监听器，设置协议及对应端口号，点击“确认”按钮创建，创建成功并返回监听管理页面。

![](/images/guides/optimization/6.jpg)

![](/images/guides/optimization/7.jpg)

4.创建节点：点击“添加节点”按钮，添加资源节点关联 MAXIR 实例。此处选择资源类型为“内⽹IP”， 监听端⼝、资源分别填写MAXIR 集群基础信息中的“DWSU控制台”内网地址和端⼝号。

![](/images/guides/optimization/8.jpg)

![](/images/guides/optimization/9.jpg)

5.健康检查：添加节点后，等待健康检查通过后，出现如下所示。

![](/images/guides/optimization/10.jpg)

6.创建防⽕墙：若在创建 ALB 时没有创建了防⽕墙，可以在安全规则中创建防⽕墙规则，其中源地址设置为本机地址，即访问ALB的客户端IP。防火墙不计费。

![](/images/guides/optimization/11.jpg)

7.测试连通性：本地访问控制台地址，访问地址：协议://外网地址:端口号。

![](/images/guides/optimization/12.jpg)

![](/images/guides/optimization/13.jpg)
