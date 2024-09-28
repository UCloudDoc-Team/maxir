
# 集群管理

MAXIR 的基本部署单元是数仓服务单元，每个服务单元包括一个不可或缺的 Hybrid DPS 集群和多个可按需调整的 Extreme DPS 集群。因此，使用 MAXIR 产品的首要步骤是创建集群。


## 创建集群

MAXIR 标准版在创建集群时，默认使用 Hybrid DPS 计算引擎。创建 Hybrid DPS 集群的同时，系统会自动生成相应的数据仓库服务单元。如需创建 Extreme DPS 集群，可在 Hybrid DPS 集群创建完成后，于对应的数据仓库服务单元中添加多个Extreme DPS集群，操作详情请操作[DPS 管理](/maxir/guides/dw-service-units/manage-dw-service-units#创建数仓服务单元) 中的“创建 Extreme DPS ”。

操作流程
>1. 登录 [MAXIR 控制台](https://console.ucloud.cn/maxir/standard) ，选择“创建集群”
>2. 根据需求，选择对应配置，并“立即购买”
>3. 跳转列表，点击“详情”查看配置

![](https://cdn.nlark.com/yuque/0/2024/png/29571810/1727528756951-5ed87cec-43cd-480c-a722-757b91b02e27.png)

![](https://cdn.nlark.com/yuque/0/2024/png/29571810/1727528756931-a892dae3-0026-4c1f-b29b-e6e0ed29fe76.png)

相关参数说明如下：

| 参数 | 是否必填 | 描述 |
| --- | --- | --- |
| 地域选择 | 是 | 集群所在地域。 |
| 默认DPS配置 | 是 | 默认创建的 Hybrid DPS 集群配置，包括计算规格和名称。 |
| 网络设置 | 是 | 包括 所属 VPC 和 所属子网。 |
| 集群名称 | 是 | 集群的名称。 |


## 查看集群
你可以查看每个集群信息，包括集群名称、资源 ID、网络信息、DPS 集群等。

操作流程
>1. 登录控制台集群[列表页](https://console.ucloud.cn/maxir/standard)，
>2. 找到目标集群，点击操作列“详情”按钮，跳转查看

![](https://cdn.nlark.com/yuque/0/2024/png/29571810/1727528756931-ad046c7e-61bd-4daa-88b9-6cb4dbfb84e4.png)






