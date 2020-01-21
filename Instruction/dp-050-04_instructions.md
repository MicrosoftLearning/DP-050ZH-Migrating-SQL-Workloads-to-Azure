---
lab:
    title: '实验室 4 – 将 SQL 工作负荷迁移至 Azure'
    module: '模块 4：将 SQL 工作负荷迁移到 Azure SQL 数据库'
---

# DP-050 – 将 SQL 工作负荷迁移到 Azure

## 实验室 4 – 将 SQL 工作负荷迁移至 Azure  

**预计用时：** 60 分钟

**先决条件：** 没有可在此实验室中执行的先决条件步骤。

**实验室文件：** 这个实验室没有实验室文件

## 实验室概述

在本实验中，你将执行到 Azure SQL 数据库的迁移。首先，你将执行架构迁移并创建作为先决条件的目标数据库，然后从在 SQL 实例中运行的数据库迁移到 Azure 上的 SQL 数据库。你将使用数据库迁移服务 (DMS) 执行联机迁移，以在直接转换数据迁移前保持源数据库和目标数据库之间的数据同步。

## 实验室目标

在本实验室结束时，你将可以：

1. 使用 Azure Cloud Shell 创建 SQL 数据库
1. 配置 Azure 数据迁移服务
1. 将数据库架构迁移到 Azure SQL 数据库
1. 使用数据迁移服务执行联机迁移

## 方案

你是负责 AdventureWorks 的高级数据库工程师，正在为数据现代化项目做准备。你将准备必要的环境以将一组数据库迁移到 Azure 虚拟机中的 SQL Server，并使用数据迁移助手执行测试迁移。

## 练习 1：使用 Azure Cloud Shell 创建 SQL 数据库

在本练习中，你将使用 Azure Cloud Shell：

- 创建新资源组

- 创建新的 Azure SQL Database 服务器实例

- 配置 Azure SQL 数据库服务器防火墙

- 创建新的通用 Azure SQL 数据库

**预计用时：** 20 分钟

Azure 中有许多选项可以自动执行服务的安装、管理和部署。其中一个选项可能是安装 Azure CLI 命令行工具，这是一个轻量级的跨平台命令行工具。另一种选项是使用 Azure Cloud Shell 自动执行和编写脚本。

Azure Cloud Shell 是一个托管在 Azure 中并通过浏览器管理的交互式 shell 环境。Cloud Shell 允许你使用 bash 或 PowerShell 来使用 Azure 服务。你可以使用 Cloud Shell 预安装命令来运行本文中的代码，而无需在本地环境中安装任何内容。

### 任务 1：登录并配置 Azure Cloud Shell

注意：此任务可以全部在 Azure 门户中完成

1. 转到 [Azure Shell](https://shell.azure.com)
1. 使用用于此培训的凭据登录 Azure 订阅

 ![欢迎使用 Azure Shell 登录窗口](/images/azureshell.png)

3. 选择 Bash 作为脚本环境

![用于选择 Bach shell 的 Azure Shell 屏幕](/images/bash.png)

### 任务 2：为 Azure Cloud Shell 创建新的存储帐户和共享

1. 当系统提示你没有为 Azure Cloud Shell 创建存储帐户时，请单击“**显示高级设置**”
1. 填写下列详细信息：
    1. 订阅：**选择用于本实验室的 Azure 订阅**
    1. Cloud Shell 区域：**选择最接近你所在地理位置的可用 Cloud Shell 区域**
    1. 资源组：  
        1. 选择“**新建**”
        1. 键入“**dp050lab4rg**”
    1. 存储帐户：
        1. 选择“**新建**”
        1. 键入“**dp050sa&lt;youridentifier&gt;**”，其中“**youridentifier**”是唯一名称
    1. 文件共享：
        1. 选择“**新建**”
        1. 键入“**dp050share&lt;youridentifier&gt;**”，其中“**youridentifier**”是唯一名称
    1. 选择“创建存储”

完成后，Azure Cloud Shell 命令行将打开 

### 任务 3：通过修改并执行以下脚本来创建 SQL 数据库服务器实例和 SQL 数据库

1. 单击“**{}**”图标打开编辑器
1. 粘贴下面的脚本并将其另存为文件：**CreateSQLDBandServer**

<脚本文件也将在托管实验室环境“Labfiles” 文件夹中以及 github 上的“lab starter”文件夹中提供>

```shell
#用于创建一个新 sql db server 实例和 sql db 的 bash 脚本  

#免责：此脚本是一个示例脚本，介绍如何创建 Azure 数据库，但为了执行实验室，使用限制最少的防火墙设置。不要使用此脚本  

#定义要传递给 Azure CloudShell Bash 的变量

resourcegroup=dp-050-labresourcegroup

#编辑提供唯一服务器名的如下变量

servername=dp-050-servername

#编辑提供位置的如下变量 – 可以通过在 shell 命令界面中键入“az account list-locations -o table”来列出 azure 位置

location=eastus

adminuser=sqladmin

password=Pa55w.rd.123456789

firewallrule=dp-050-access

#编辑提供唯一数据库名的脚本

labdatabase=dp-050-AdventureworksLT2008R2

#创建一个资源组

az group create --name $resourcegroup --location $location

#创建 sql db 服务器名

az sql server create --name $servername --resource-group $resourcegroup --admin-user $adminuser --admin-password $password --location $location

#显示当前防火墙列表

az sql server firewall-rule list --resource-group $resourcegroup --server $servername

#创建基于服务器的防火墙 - 注意 - 你应该根据你的环境限制开始/结束 IP 范围

az sql server firewall-rule create --resource-group $resourcegroup --server $servername --name $firewallrule --start-ip-address 0.0.0.0 --end-ip-address 255.255.255.255

#创建一个通用 SQL 数据库

az sql db create --name $labdatabase --resource-group $resourcegroup --server $servername -e GeneralPurpose -f Gen4 -c 1 

```

3. 通过键入“**sh CreateSQLDBandServer**”来执行脚本

完成并成功执行脚本后，你可以验证是否已在 Azure 帐户订阅中创建资源组、服务器和数据库。

bash 脚本输出应指示与下面突出显示的文本类似的服务器名：

```shell
{ "administratorLogin": "sqladmin",

  "administratorLoginPassword": null,

  "fullyQualifiedDomainName": "dp-050-servername.database.windows.net",

  "id": "/subscriptions/xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx/resourceGroups/dp-050-labresourcegroup/providers/Microsoft.Sql/servers/dp-050-servername",

  "identity": null,

  "kind": "v12.0",

  "location": "eastus",

  "name": "dp-050-servername",

  "resourceGroup": "dp-050-labresourcegroup",

  "state": "Ready",

  "tags": null,

  "type": "Microsoft.Sql/servers",

  "version": "12.0"

}
```

## 练习 2：将数据库迁移到 Azure SQL 数据库（脱机）

在本练习中，你将迁移在 Azure 上运行的 SQL Server 实例中的某一数据库的数据库架构（在以前的实验中创建的 SQL 2017 VM），并将架构加载到上一练习中创建的 SQL 数据库中。

你将：

- 使用数据迁移助手创建新的迁移项目
- 仅执行架构迁移

**预计用时：**20 分钟

**注意：**本练习将使用 SQL Server 2008 R2 虚拟机上运行的数据迁移助手完成

### 任务 1：使用数据迁移助手创建迁移项目

1. 打开“**Microsoft 数据迁移助手**”
1. 单击“**+**”启动一个新项目
1. 选择“**迁移**”作为项目类型
1. 键入“**SQLDBMigration**”作为项目的名称
1. 验证以下设置：
1. 源服务器类型：**SQL Server**
1. 目标服务器类型：**Azure SQL 数据库**
1. 迁移范围：**仅架构**
1. 选择“**创建**”

### 任务 2：执行架构迁移

1. 在“连接到源服务器”中，键入“**伦敦**”
1. 取消选中“**加密连接**”选项
1. 选择“**连接**”
1. 在显示数据库列表时，选择“**AdventureworksLT2008R2**”
1. 在你验证先前的评估中没有迁移问题后，取消选中“**迁移前评估数据库**”复选框
1. 单击“**下一步**”
1. 在“连接到目标服务器”对话框中，键入你在上一个练习中创建的服务器的“**服务器名**”。按其“**完全限定名**”列出服务器（例如：dp-050-servername.database.windows.net）
1. 将身份验证类型更改为“**SQL Server 身份验证**”
1. 提供用户名：**sqladmin**
1. 提供密码：**Pa55w.rd.123456789**
1. 单击“**连接**”
1. 选择在上一个练习中创建的目标数据库作为 SQL 数据库服务器上的目标数据库
1. 单击“**下一步**”
1. 查看所有架构对象，如果未发现任何问题，单击“生成 SQL 脚本”

“数据迁移助手”窗口外观类似于：
![“数据迁移助手”窗口](/images/dbmigrate.png)

15. 单击“部署架构”
1. 关闭“数据迁移助手”

**注意：** 现在，所有数据库对象都将迁移到 Azure SQL 数据库，此迁移最多可能需要 10 分钟才能完成 

## 练习 3：将本地数据库迁移到 Azure SQL 数据库（联机）

在本练习中，你将使用 Azure 数据迁移服务执行数据库的联机迁移。

你将：

- 配置源实例以进行复制，并确保满足迁移的先决条件。
- 使用 Azure 数据迁移服务执行实时迁移

**预计用时：** 20 分钟

### 任务 1：将 SQL Server 配置为复制分发服务器

此任务将使用 SQL 2008 R2 虚拟机上的 SQL Management Studio 完成，但此虚拟机已连接到 **SQL Server 2017 实例。**

**注意：** 在连接到 SQL Server 2017 实例时执行这些任务，以避免必须在托管的 SQL 2008 R2 VM 和 Azure 之间配置 VPN 访问。在非实验室环境中，你通常会配置 VPN 或 Expressroute。

1. 在 SQL Management Studio 对象资源管理器中，连接到你在实验室 3 中创建的 SQL Server 2017 实例。
    如果未注册 SQL Server 2017 实例，请执行以下步骤：
        选择“**连接 | 数据库引擎**”并在“**连接到服务器**”对话框中填写以下信息：

            服务器名：**<SQL 2017 VM 完全限定的域名>**

            例如 sql2017vmxxxx.centralus.cloudapp.azure.com 

2. **右键单击** “**复制**”
1. 选择“**配置分发**”
1. 运行“**配置分发向导**”
1. 接受每个“配置分发”页面上的默认设置。
    1. 如果向导提示你“**自动启动 SQL Server 代理**”，单击“**是**”。
1. 单击“**下一步**”
1. 单击“**快照**”屏幕上的“**下一步**”
1. 单击“**分发**”数据库名上的“**下一步**”
1. 单击“**发布者**”对话窗口上的“**下一步**”
1. 单击“**向导操作**”对话窗口上的“**下一步**”
1. 单击“**完成**”
1. 如果 **SQL Server 代理无法启动**，右键单击 **SQL Management Studio 中的 SQL Server 代理手动启动代理 | 对象资源管理器 | SQL Server 代理** 和启动服务
1. 通过在新查询窗口中执行以下查询设置将 **AdventureworksLT2008R2** 数据库的数据库恢复模式设置为“完整”：

```sql 
ALTER DATABASE AdventureworksLT2008R2 SET RECOVERY FULL WITH NO_WAIT
```

14. 执行以下查询，执行数据库的完整备份  

```sql
BACKUP DATABASE AdventureworksLT2008R2 TO DISK = ‘d:\awlt2008r2backup.bak’
```

15. 备份完成后，关闭“**SQL Management Studio**”。

### 任务 2：使用 Azure 数据库迁移服务执行联机迁移

在此任务中，你将配置 Azure 数据库迁移服务，以便在 VM 和 SQL 数据库中运行的数据库之间实现实时迁移。

**注意：** 所有这些任务都在 Azure 门户中完成

1. 在 Azure 门户网站上，单击“**创建资源**”
1. 键入“**Azure 数据库迁移服务**”
1. 单击“**创建**”
1. 在“服务名”中 - 提供服务名称“**dp050dmsxxxxxx**”，其中 xxxxx 是唯一值
1. 选择“**Azure 订阅**”
1. 选择你以前创建的“**资源组**”
1. 选择“**位置**”
1. 创建一个“**新虚拟网络**”，将其命名为“**OnPremGateway**”
1. 将定价层更改为“**高级**”
1. 单击“**确定**”。
1. 单击“**创建**”

**注意：** 此部署可能需要长达 10 分钟

12. 完成部署后，单击“转到资源”

    或者，你可以将 Azure 数据迁移服务添加到 Azure 门户中的左侧边栏选项卡中

    a) 在 Azure 门户上的“所有服务”中找到“Azure 数据迁移服务” 
    b) 勾选星形图标，将 Azure 数据迁移服务添加到 Azure 门户中的左侧边栏选项卡中
    c) 将左侧边栏选项卡展开到 Azure 数据迁移服务
    d) 选择“数据迁移服务” 

13. 单击“新迁移项目”
1. 按如下方式填写“新建迁移项目”对话框：

    项目名：**DP050OnlineMigration**

    源服务器类型：**SQL Server**

    目标服务器类型：**Azure SQL 数据库**

    选择活动类型：**更改为联机数据迁移**

15. 单击“保存”
1. 单击“创建并运行活动”
1. 在“迁移源详细信息”中提供以下信息：

    源 SQL Server 实例名称：&lt;SQL 2017 VM 完全限定的域名&gt;

    验证类型：**SQL 身份验证**

    用户名：**sqladmin**

    密码：**Pa55w.rd.123456789**

18. 取消选中“加密连接”
1. 单击“保存”
1. 选择 sql 迁移的目标目的地，并在“迁移目标详细信息”中提供以下详细信息：

    目标服务器名：&lt;full qualified domain of the azure sql 数据库服务器完全限定的域&gt;

    验证类型：**SQL 身份验证**

    用户名：**sqladmin**

    密码：**Pa55w.rd.123456789**

21. 单击“**保存**”
1. 在“**映射到目标数据库**”窗口选择你为其执行架构迁移的数据库，同时选择目标数据库。
1. 单击“**保存**”
1. 在里面“**配置迁移**”设置窗口，单击下拉列表，注意不会复制某个表，因为未对 dbo 表启用“更改数据捕获”。
1. 单击“**保存**”
1. 为“**迁移项目**”活动命名，键入“**dp050lab4activity**”

**注意：**在生产环境中，你将增加 SQL 数据库的数据库层，以执行更快的数据加载。

27. 单击“**运行查询**”
1. 在迁移过程中，“**刷新**”该“**活动**”窗口。
1. 查看状态，直到状态显示为“**准备直接转换**”

    **注意：** 如果要进行数据库更改，则可以查看增量数据加载

    或者，你可以使用以下命令在源数据库中的“**Productcategory**”表中插入一些记录（在设置为源数据库的 SQL 实例上）

```sql
USE [AdventureWorksLT2008R2]
GO

INSERT INTO [Sales LT].[ProductCategory]
 ([ParentProductCategoryID]
 ,[Name]
 ,[ModifiedDate])
VALUES
 (1, ' Myproduct', getdate())

```

    然后查看“增量数据同步”选项卡并浏览插入内容。

30. 选择数据库，然后单击**启动直接转换**
1. 勾选**确认**
1. 勾选**验证我的数据库**
1. 勾选所有验证选项
1. 单击**应用**

恭喜！你现在已成功完成到 Azure SQL 数据库的联机迁移。
