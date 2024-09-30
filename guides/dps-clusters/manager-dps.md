# DPS 管理
MAXIR 提供了丰富的功能来帮助你管理集群以满足你的业务需求。例如，你可以根据特定规格创建多个 DPS 来处理不同的工作负载，实现工作负载间的隔离。


## 创建 Extreme DPS 
MAXIR 支持按需创建多个 Extreme DPS 。

操作流程
>1. 登录 [MAXIR 控制台](https://console.ucloud.cn/maxir/standard)。在默认展示的集群列表中，点击目标集群名称，进入详情页。
>2. 点击 DPS 列表左上方的“创建Extreme DPS”按钮。
>3. 在弹出的 DPS配置 页面，配置 DPS 集群。
>4. 在右侧「付费信息」部分，确认付费信息，并点击“立即购买”按钮，创建成功，返回详情页。

![](/images/guide/DPS-1.png)
![](/images/guide/DPS-2.png)


相关参数说明如下：

| 参数 | 是否必填 | 描述 |
| --- | --- | --- |
| DPS类型 | 是 | DPS 集群类型。此处固定为 Extreme。 |
| 计算规格 | 是 | 计算资源的大小 |
| AQS | 否 | 默认关闭，可开启并设置可处理路由大查询的额外计算资源上限，[功能详情](/maxir/guides/dps-clusters/aqs) |
| DPS名称 | 是 | DPS 集群的名称。DPS集群名称必须以字母开头，只能包含字母、数字和连字符（-），长度为 4～32 个字符。 |


## 扩缩容 DPS
随着业务发展，工作负载的大小发生改变，当前的 DPS 集群大小可能不再满足需求。MAXIR 支持实时在线调整 DPS 集群的大小，且不会影响运行中的业务。

操作流程
>1. 登录 [MAXIR 控制台](https://console.ucloud.cn/maxir/standard)，在默认展示的集群列表中，点击目标集群名称，进入详情页。
>2. 在 DPS 列表中，找到目标 DPS 集群，点击操作列中的“扩缩容”按钮。
>3. 在 修改规格 区域，将计算规格调整为目标规格。
>4. 在右侧 付费信息 部分，确认付费信息，并点击“立即购买”按钮。

配置完成后，新的配置立即生效，无需重启等其他操作。

![](/images/guide/DPS-3.png)

![](/images/guide/DPS-4.png)

## 修改 AQS 配置
如果您在业务发展过程中，想开启/关闭集群自适应查询扩展[ AQS 能力](/maxir/guides/dps-clusters/aqs)，或者调整 AQS 可使用的共享计算资源的上限，可按如下操作：
>1. 登录 [MAXIR 控制台](https://console.ucloud.cn/maxir/standard)，在默认展示的集群列表中，点击目标集群名称，进入详情页。
>2. 在 DPS 列表中，找到目标  Extreme DPS 集群，点击AQS列中的编辑 icon。
>3. 弹窗设置 AQS 开关状态及计算资源规格，点击“确认”按钮。

![](/images/guide/AQS-1.png)
![](/images/guide/AQS-2.png)



## 关闭 DPS
当某个 DPS 集群暂时无需使用时，可通过「关闭」功能手动暂停该 DPS 集群。DPS 集群暂停后，正在运行或排队中的所有查询都将被终止。请谨慎执行此操作。

操作流程

>1. 登录 MAXIR 控制台。在默认展示的集群列表中，点击目标集群名称，进入详情页。
>2. 在 DPS 列表中，找到目标 DPS 集群，点击 操作 列中的“关闭”按钮。
>3. 在弹出的确认框中，点击“确认”。

![](/images/guide/DPS-5.png)

## 删除 DPS 
如果不再需要 Extreme DPS 集群，可以手动删除。Hybrid DPS 集群不能被删除。

>1. 登录 MAXIR 控制台。在默认展示的集群列表中，点击目标集群名称，进入详情页。
>2. 在 DPS 列表中，找到目标 DPS 集群，在 操作 列中选择 ... > 删除。
>3. 在弹出的确认对话框中，点击 确认。

![](/images/guide/DPS-6.png)
