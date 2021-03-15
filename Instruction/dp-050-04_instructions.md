# DP 050 – 将 SQL 工作负载迁移到 Azure

## 实验室 4 - 将 SQL 工作负载迁移到 SQL 数据库

**预计用时：** 60 分钟

**先决条件：** 没有可在此实验室中执行的先决条件步骤。

**实验室文件：** 这个实验室没有实验室文件

## 实验室概述

在本实验中，你将执行到 Azure SQL 数据库的迁移。首先，你将执行架构迁移并创建作为先决条件的目标数据库，然后从在本地 SQL 实例中运行的数据库迁移到 Azure SQL 数据库。你将使用数据库迁移服务 (DMS) 执行联机迁移，以在直接转换到新数据库前保持源数据库和目标数据库之间的数据同步。

## 实验室目标

本实验室结束时，你将能够：

- 使用 Azure Cloud Shell 在 Azure SQL 数据库中创建数据库
- 配置 Azure 数据迁移服务。
- 将数据库架构迁移到 Azure SQL 数据库。
- 使用数据迁移服务执行联机迁移。

## 应用场景

你是负责 AdventureWorks 的高级数据库管理主管，正在为数据现代化项目运行做准备。你将准备必要的环境以将一组数据库迁移到 Azure 虚拟机中的 SQL Server，并使用数据迁移助手执行测试迁移。

## 练习 1：使用 Azure Cloud Shell 在 Azure SQL 数据库中创建数据库

在本练习中，你将使用 Azure Cloud Shell：

- 创建新资源组。
- 创建新的 Azure SQL 数据库服务器实例。
- 配置 Azure SQL 数据库服务器防火墙。
- 创建新的通用数据库。

**预计用时：** 20 分钟

Azure 中有许多选项可以自动执行服务的安装、管理和部署。其中一个选项可能是安装 Azure CLI 命令行工具，这是一个轻量级的跨平台命令行工具。另一种选项是使用 Azure Cloud Shell 自动执行和编写脚本。

Azure Cloud Shell 是一个托管在 Azure 中并通过浏览器管理的交互式 shell 环境。借助 Cloud Shell，你可以使用 bash 或 PowerShell 来使用 Azure 服务。你可以使用 Cloud Shell 预安装命令来运行本文中的代码，而无需在本地环境中安装任何内容。

### 任务 1：登录并配置 Azure Cloud Shell

> [注意！]
> 此任务可以完全在 Azure 门户中完成。

1. 转到 [Azure Shell](https://shell.azure.com)
1. 使用用于此培训的凭据登录 Azure 订阅。
1. 选择 **“Bash”** 作为脚本环境。

### 任务 2：为 Azure Cloud Shell 创建新的存储帐户和共享

1. 当系统提示你没有为 Azure Cloud Shell 创建存储帐户时，请选择 **“显示高级设置”**，输入以下值，然后选择 **“创建存储”**：

    | 属性 | 值 |
    | --- | --- |
    | 订阅 | 选择用于本实验室的 Azure 订阅 |
    | Cloud Shell 区域 | 选择接近你所在地理位置的可用 Cloud Shell 区域 |
    | 资源组 | 创建一个名为 **“dp050lab4rg”** 的新资源组 |
    | 存储帐户 | 创建一个名为 **“dp050sa&lt;youridentifier&gt;”** 的新存储帐户，其中，**“&lt;youridentifier&gt;”** 是一个唯一字符串 |
    | 文件共享 | 创建一个名为 **“dp050share&lt;youridentifier&gt;”** 的新文件共享，其中， **“&lt;youridentifier&gt;”** 是一个唯一名称 |

1. 创建存储帐户后，Azure Cloud Shell 便会打开。

### 任务 3：创建 Azure SQL 数据库服务器示例和数据库

1. 单击 **“{}”** 图标打开代码编辑器。
1. 将此脚本粘贴到代码编辑器中：

    ```bash
    #用于创建一个新 sql db server 实例和 sql db 的 bash 脚本  
    
    #免责：此脚本是一个示例脚本，介绍如何创建 Azure 数据库，但为了执行实验室，使用限制最少的防火墙设置。请不要使用此脚本  
    
    #定义资源组的名称。
    resourcegroup=dp-050-labresourcegroup
    
    #编辑提供唯一服务器名的如下变量
    servername=dp-050-servername
    
    #编辑提供位置的如下变量 – 可以通过在 shell 命令界面中键入“az account list-locations -o table”来列出 azure 位置
    location=eastus
    
    adminuser=sqladmin
    password=Pa55w.rdPa55w.rd
    firewallrule=dp-050-access
    
    #编辑提供唯一数据库名的脚本
    labdatabase=dp-050-Adventureworksxxx
    
    #创建一个资源组
    az group create --name $resourcegroup --location $location
    
    #创建 sql db 服务器名
    az sql server create --name $servername --resource-group $resourcegroup --admin-user $adminuser --admin-password $password --location $location
    
    #显示当前防火墙列表
    az sql server firewall-rule list --resource-group $resourcegroup --server $servername
    
    #创建基于服务器的防火墙 - 注意 - 你应该根据你的环境限制开始/结束 IP 范围
    az sql server firewall-rule create --resource-group $resourcegroup --server $servername --name $firewallrule --start-ip-address 0.0.0.0 --end-ip-address 255.255.255.255
    
    #创建一个通用 SQL 数据库
    az sql db create --name $labdatabase --resource-group $resourcegroup --server $servername -e GeneralPurpose
    ```

1. 在脚本中，将值 `dp-050-servername` 更改为唯一的服务器名，例如 `dp-050-serverxxx`。请不要在此名称中使用大写字母。
1. 在脚本中，将值 `eastus` 更改为你附近的 Azure 位置。
1. 在脚本中，将值 `dp-050-Adventureworksxxx` 更改为唯一的服务器名。
1. 若要保存脚本，请按 <kbd>CTRL + S</kbd>，键入 **“CreateSQLDBandServer.sh”**，然后选择 **“保存”**。
1. 若要退出代码编辑器，请按 <kbd>CTRL + Q</kbd>。
1. 若要执行脚本，请键入 **“sh CreateSQLDBandServer.sh”**，然后按 <kbd>Enter</kbd>。

    脚本成功完成后，你可以验证是否已在 Azure 帐户订阅中创建资源组、服务器和数据库。

## 练习 2：将数据库迁移到 Azure SQL 数据库

在本练习中，你将迁移在本地运行的 SQL Server 实例中的某一数据库的数据库架构，并将架构加载到上一练习中创建的 SQL 数据库中。数据库架构的源 SQL Server实例在 VM LON-DEV-01 上运行。

本实验室结束时，你将能够：

- 使用数据迁移助手创建新迁移项目
- 迁移数据库架构。

**预计用时：** 20 分钟

> [注意！]
> 本练习使用在 LON-DEV-01 VM 上运行的数据迁移助手来完成。

### 任务 1：使用数据迁移助手创建迁移项目

1. 登录到在教室环境中运行的 **LON-DEV-01** 虚拟机。用户名是 **“administrator”**，密码为 **“Pa55w.rd”**。
1. 若要创建新数据迁移项目，请打开 **Microsoft 数据迁移助手**，然后单击 **“+”**。
1. 在 **“新建”** 页中，输入以下值，然后选择 **“创建”**：

    | 属性 | 值 |
    | --- | --- |
    | 项目类型 | 迁移 |
    | 项目名称 | SQLSchemaMigration |
    | 源服务器类型 | SQL Server |
    | 目标服务器类型 | Azure SQL 数据库 |
    | 迁移范围 | 仅限架构 |

### 任务 2：执行架构迁移

1. 在 **“选择源”** 页上的 **“连接到源服务器”** 下，输入以下值，然后选择 **“连接”**：

    | 属性 | 值 |
    | --- | --- |
    | 服务器名称 | localhost |
    | 身份验证类型 | Windows 身份验证 |
    | 加密连接 | 否 |
    | 信任服务器证书 | 是 |

1. 在数据库列表中，选择 **“AdventureworksLT2008R2”** 并取消选择 **“迁移前访问数据库”** 复选框。
1. 选择 **“下一步”**。
1. 在 **“选择目标”** 页上的 **“连接到目标服务器”** 下，输入以下值，然后选择 **“连接”**：

    | 属性 | 值 |
    | --- | --- |
    | 服务器名称 | 键入你在上一个练习中创建的服务器的 **servername**。按其 **“完全限定名”** 列出服务器，例如：dp-050-servername.database.windows.net |
    | 身份验证类型 | SQL Server 身份验证 |
    | 用户名 | sqladmin |
    | 密码 | Pas55w.rdPa55w.rd |
    | 加密连接 | 否 |
    | 信任服务器证书 | 是 |

1. 在数据库列表中，选择你在上一个练习中创建的目标数据库，然后选择 **“下一步”**
1. 查看所有架构对象，然后选择 **“生成 SQL 脚本”**

    “数据迁移助手”窗口外观类似于：

    ![“数据迁移助手”窗口](/images/dbmigrate.png)

1. 选择 **“部署架构”**。
1. 部署完成后，关闭数据迁移助手。

## 练习 3：将本地数据库迁移到 Azure SQL 数据库

在本练习中，你将使用 Azure 数据迁移服务执行数据库的联机迁移。

本练习结束时，你将能够：

- 配置要复制的源实例，并确保满足迁移的先决条件。
- 使用 Azure 数据迁移服务执行实时迁移。

**预计用时：** 20 分钟

### 任务 1：将 SQL Server 配置为复制分发服务器

你将使用 SQL 2008 R2 虚拟机上的 SQL Management Studio 完成此任务，但此虚拟机已连接到 **SQL Server 2017 实例**。

> [注意！]
> 在连接到 Azure 中的 SQL Server 2017 实例时在 LON-DEV-01 上执行这些任务，以免必须在托管的 SQL 2008 R2 VM 和 Azure 之间配置 VPN 访问。在非实验室环境中，你通常会配置 VPN 或 ExpressRoute。

1. 在 SQL Management Studio 对象资源管理器中，连接到你在实验室 3 中创建的 SQL Server 2017 实例。

    如果未注册 SQL Server 2017 实例，请选择 **“连接/数据库引擎”**，然后使用以下值：

    | 属性 | 值 |
    | --- | --- |
    | 服务器名称 | SQL 2017 VM 的 IP 地址或完全限定域名，例如 sql2017vmxxxx.centralus.cloudapp.azure.com |
    | 身份验证 | SQL Server 身份验证 |
    | 登录名 | sqladmin |
    | 密码 | Pa55w.rdPa55w.rd |

1. 在 **“对象资源管理器”** 中，右键单击 **“复制”**，然后选择 **“配置分发”**。
1. 在 **“配置分发”** 向导中，接受每个“配置分发”页上的默认设置。如果向导提示你**自动启动 SQL Server 代理**，请选择 **“是”**。
1. 在 **“完成向导”** 页中，选择 **“完成”**
1. 如果 **SQL Server 代理无法启动**，右键单击 **“SQL Management Studio 中的 SQL Server 代理” 手动启动代理 | 对象资源管理器 | SQL Server 代理** 和启动服务
1. 若要将 **AdventureworksLT2008R2** 数据库的数据库恢复模式设置为“完整”，请在新查询窗口中执行此查询：

    ```sql
    ALTER DATABASE AdventureworksLT2008R2 SET RECOVERY FULL WITH NO_WAIT
    ```

1. 执行以下查询，执行数据库的完整备份  

    ```sql
    BACKUP DATABASE AdventureworksLT2008R2 TO DISK = 'd:\awlt2008r2backup.bak'
    ```

1. 备份完成后，关闭 **SQL Management Studio**。

### 任务 2：使用 Azure 数据库迁移服务执行联机迁移

在此任务中，你将配置 Azure 数据库迁移服务，以便在 VM 和 Azure SQL 数据库中运行的数据库之间实现实时迁移。

> [注意！]
> 请在 Azure 门户中完成所有这些任务。

1. 在 Azure 门户中，选择 **“创建资源”**。
1. 在 **“在市场中搜索”** 文本框中，键入 **“Azure 数据库迁移服务”**，然后按 <kbd>Enter</kbd>。
1. 选择 **“创建”**。
1. 在 **“创建迁移服务”** 向导中的 **“基本信息”** 页上，输入以下值，然后选择 **“下一步: 网络 \>”**：

    | 属性 | 值 |
    | --- | --- |
    | 订阅 | 选择你的订阅。 |
    | 资源组 | dp050lab4rg |
    | 迁移服务名称 | **dp050dmsxxx**，其中 **“xxx”** 是一个唯一值。 |
    | 位置 | 选择你附近的位置。 |
    | 服务模式 | Azure |
    | 定价层 | 高级 |

1. 在 **“网络”** 页上的 **“虚拟网络名称”** 文本框中，键入 **“OnPremGateway”**，然后选择 **“查看 + 创建”**。
1. 在 **“查看 + 创建”** 页上，选择 **“创建”**。

    > [注意！]
    > 此部署最多可能需花费 10 分钟。

1. 部署完成后，选择 **“转到资源”**。
1. 选择 **“+ 新增迁移项目”**。
1. 在 **“新建迁移项目”** 页上，输入以下值，然后选择 **“创建并运行活动”**：

    | 属性 | 值 |
    | --- | --- |
    | 项目名称 | DP050OnlineMigration |
    | 源服务器类型 | SQL Server |
    | 目标服务器类型 | Azure SQL 数据库 |
    | 选择活动类型 | 联机数据迁移 |

1. 在 **“从 SQL Server 到 Azure SQL 数据库的联机迁移”** 向导中的 **“选择源”** 页上，输入以下值，然后选择 **“下一步: 选择目标 \>\>”**：

    | 属性 | 值 |
    | --- | --- |
    | 源 SQL Server 实例名称 | Azure 中 SQL 7 VM 的完全限定的域名 |
    | 身份验证类型 | SQL 身份验证 |
    | 用户名 | sqladmin |
    | 密码 | Pa55w.rdPa55w.rd |
    | 加密连接 | 已取消选中 |
    | 信任服务器证书 | 已选中 |

1. 在 **“选择目标”** 页上，输入以下值，然后选择 **“映射到目标数据库 \>\>”**：

    | 属性 | 值 |
    | --- | --- |
    | 目标服务器名 | Azure SQL 数据库服务器的完全限定的域 |
    | 身份验证类型 | SQL 身份验证 |
    | 用户名 | sqladmin |
    | 密码 | Pa55w.rdPa55w.rd |
    | 加密连接 | 已取消选中 |
    | 信任服务器证书 | 已选中 |

1. 在 **“映射到目标数据库”** 页上，选择要为其进行架构迁移的数据库，选择目标数据库，然后选择 **“配置迁移设置 \>\>”**。
1. 在 **“配置迁移设置”** 页上，检查每个表的注释，然后选择 **“摘要 \>\>”**。
1. 为 **“活动名称”** 文本框命名，键入 **“dp050lab4activity”**，然后选择 **“运行迁移”**。

    > [注意！]
    在生产环境中，你将增加 SQL 数据库的数据库层，以执行更快的数据加载。

1. 在迁移过程中，请偶尔选择 **“刷新”** 并监视 **“状态”** 列，直到其显示 **“准备进行直接转换”**。

    > [注意！]
    > 在迁移期间，对源数据库中的数据所进行的任何更改都将同步到目标数据库。

    如果有时间，你可以使用此 SQL 命令在源数据库的 **ProductCategory** 表中插入一些记录。确保在源 VM 上执行以下命令：

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

1. 数据迁移完成后，选择数据库，然后选择 **“开始直接转换”**。
1. 选择 **“确认”**，然后选择 **“应用”**。

结果：你现在已成功完成到 Azure SQL 数据库的联机迁移。