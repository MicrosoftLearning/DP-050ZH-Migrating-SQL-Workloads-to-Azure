# DP 050 – 将 SQL 工作负载迁移到 Azure

## 实验室 3 - 将 SQL 工作负载迁移到 Azure 虚拟机中的 SQL Server

**预计用时：** 60 分钟

**先决条件：** 没有可在此实验室中执行的先决条件步骤。

**实验室文件：** 这个实验室没有实验室文件。

## 实验室概述

学生将首先评估他们将用于从本地 SQL Server 2008 R2 实例迁移到在虚拟机中运行的 SQL Server 2017 的迁移过程。之后，他们将使用数据迁移助手执行迁移，以迁移数据库。最后，他们将评估成功的迁移。

## 实验室目标

完成本实验室后，你将能够：

- 在 Azure 中创建运行 SQL Server 2017 的新虚拟机。
- 创建 Azure 存储帐户和文件共享。
- 将 SQL Server 2008 R2 数据库迁移到 Azure VM 中的 SQL Server。

## 应用场景

你是负责 AdventureWorks 的高级数据库管理主管，正在为数据现代化项目运行做准备。你会准备必要的环境以将一组数据库迁移到 Azure 虚拟机中的 SQL Server，并使用数据迁移助手执行测试迁移。

## 练习 1：在 Azure 中创建运行 SQL Server 2017 的新虚拟机

在本练习中，你将使用 Azure 门户在 Azure 上创建一个新的虚拟机。

**预计用时：** 20 分钟

本练习的任务是：

1. 在 Azure 门户中创建新虚拟机

### 预配 SQL Server 2017 虚拟机

> [！注意]
> 如果你在托管实验室环境中运行此实验室，请在该环境中执行这些步骤。

1. 在 [Azure 门户](https://portal.azure.com)中，选择 **“创建资源”**。
1. 在“市场”搜索框中，键入 **“Windows Server 2019 上的 SQL Server 2017”**，然后按 Enter。在 **“显示所有结果”** 下，选择 **“Windows Server 2019 上的 SQL Server 2017”**。
1. 在 **“选择计划”** 下拉列表中，选择 **“免费 SQL Server 许可证: Windows Server 2019 上的 SQL Server 2017 Developer”**，然后选择 **“创建”**。
1. 在 **“创建虚拟机”** 向导中的 **“基本信息”** 页上，输入以下值，然后选择 **“下一步: 磁盘 \>”**：

    | 属性 | 值 |
    | --- | --- |
    | 订阅 | 选择你的订阅 |
    | 资源组 | 创建一个名为 **“DP-050-Training”** 的新资源组 |
    | 虚拟机名称 | sql2017vm |
    | 区域 | 选择离你较近的区域 |
    | 可用性选项 | 无需基础结构冗余 |
    | 映像 | 免费 SQL Server 许可证：Windows Server 2019 上的 SQL Server 2017 Developer - Gen1 |
    | Azure Spot 实例 | 否 |
    | 大小 | Standard_D2_v2 |
    | 用户名 | sqladmin |
    | 密码 | Pa55w.rdPa55w.rd |
    | 确认密码 | Pa55w.rdPa55w.rd |
    | 公共入站端口 | 允许选定的端口 |
    | 选择入站端口 | RDP (3389) |
    | 你是否要使用现有 Windows 许可证？ | 否 |
    
1. 在 **“磁盘”** 页上，接受默认设置，然后选择 **“下一步: 网络 \>”**。
1. 在 **“网络”** 页上，接受默认设置，然后选择 **“下一步: 管理 \>”**。
1. 在 **“管理”** 页的 **“启动诊断”** 列表中，选择 **“禁用”**，然后选择 **“下一步: 高级 \>”**。
1. 在 **“高级”** 页上，接受默认设置，然后选择 **“下一步: SQL Server 设置 \>”**。
1. 在 **“SQL Server 设置”** 页上，输入以下值，然后选择 **“查看 + 创建”**：

    | 属性 | 值 |
    | --- | --- |
    | SQL 连接 | 公共 (Internet) |
    | 端口 | 1433 |
    | SQL 身份验证 | 启用 |
    | 登录名 | sqladmin |
    | 密码 | Pa55w.rdPa55w.rd |
    | Azure 密钥保管库集成 | 禁用 |

1. 在 **“查看 + 创建”** 页上，选择 **“创建”**。

    > [注意！]
    > 这一步大约需要 10 分钟才能完成。

1. 部署完成后，选择 **“转到资源”**。
1. 找到并记录你的 VM 的**公用 IP 地址**。稍后需要此地址。
1. 在 **“DNS 名称”** 的旁边，选择 **“配置”**。
1. 在 **“DNS 名称标签(可选)”** 文本框中，键入一个唯一的 DNS 名称并进行记录。

    例如：sql2017vmxxxx.centralus.cloudapp.azure.com

1. 选择 **“保存”**。

结果：完成此练习后，你便拥有在 Azure 虚拟机中运行的 SQL Server 2017 实例。

## 练习 2：创建 Azure 存储帐户和文件共享

在本练习中，你将使用 Azure 门户在 Azure 上创建新存储帐户。

**预计用时：** 15 分钟

本练习的主要任务是：

1. 创建 Azure 存储帐户。
1. 在 Azure 存储帐户中创建一个文件共享。

### 创建 Azure 存储帐户

1. 在 [Azure 门户](https://portal.azure.com)中，选择 **“创建资源”**。
1. 在 **“在市场中搜索”** 文本框中，键入 **“存储帐户”**，然后按 **Enter**。
1. 在 **“显示所有结果”** 下，选择 **“存储帐户”**，然后选择 **“创建”**。
1. 在 **“创建存储帐户”** 向导的 **“基本信息”** 页上，输入以下值：

    | 属性 | 值 |
    | --- | --- |
    | 订阅 | 选择你的订阅 |
    | 资源组 | DP-050-Training |
    | 存储帐户名称 | **dp050storagexxxx** （xxxx 是随机字符序列） |
    | 位置 | 选择与虚拟机相同的位置 |
    | 性能 | 标准 |
    | 帐户类型 | StorageV2（常规用途 v2） |
    | 复制 | 本地冗余存储 (LRS) |

    > [注意！]
    > 请仔细记下你使用的存储帐户名称。该实验室稍后将需要此名称。

1. 选择 **“查看 + 创建”**。
1. 在 **“查看 + 创建”** 页上，选择 **“创建”**。

    > [注意！]
    > 此部署将花费几分钟时间

1. 部署完成后，选择 **“转到资源”**。
1. 在 **“设置”** 下，选择 **“访问密钥”**。
1. 在 **“访问密钥”** 页上，选择 **“显示密钥”**，然后在 **“key1”** 下，记录 **“密钥”** 文本框的内容。

### 创建文件共享

1. 在存储帐户页的左侧菜单的 **“文件服务”** 下，选择 **“文件共享”**。
1. 在 **“文件共享”** 页上，选择 **“+ 文件共享”**。
1. 在 **“新建文件共享”** 页上，输入以下值：

    | 属性 | 值 |
    | --- | --- |
    | 名称 | backupshare |
    | 配额 | 200 GiB |

1. 选择 **“创建”**。

结果：你现已成功创建 Azure 文件共享，它将用作 SQL Server 数据库备份文件的共享访问位置。在下一个练习中，你将配置 SQL 实例来访问共享位置。

## 练习 3：为 SQL Server 实例创建连接以连接到 Azure 文件共享

在本练习中，你将配置 SQL Server 环境，以访问本地服务器和新 Azure VM 上的 Azure 文件共享。

**预计用时：** 10 分钟

本练习的主要任务是：

1. 通过映射网络驱动器，通过 SQL Management Studio 注册文件共享
1. 连接到文件共享。

### 在 SQL Management Studio 中注册服务器实例

1. 登录到在教室环境中运行的 **LON-DEV-01** 虚拟机。用户名是 **“administrator”**，密码为 **“Pa55w.rd”**。
1. 启动 **SQL Management Studio**，并连接到本地实例 (LONDON)。
1. 在 SQL Management Studio 对象资源管理器中，选择 **“连接”**，然后选择 **“数据库引擎”**。
1. 在 **“连接到服务器”** 对话框中，输入以下值，然后选择 **“连接”**：

    | 属性 | 值 |
    | --- | --- |
    | 服务器名称 | 输入 Azure 中 SQL 2017 VM 的完全限定的域名或 IP 地址例如： **sql2017vmxxx.centralus.cloudapp.azure.com** |
    | 身份验证 | SQL Server |
    | 登录名 | sqladmin |
    | 密码 | Pa55w.rdPa55w.rd |

### 将本地 SQL 实例连接到文件共享

> [注意！]
> 为了使 SQL Server 能够连接到文件共享上的驱动器号，必须通过在 SQL Server Management Studio 中运行 `xp_cmdshell` 来映射网络驱动器，以便 SQL 服务帐户能够访问共享。数据迁移助手使用 SQL 服务帐号备份数据库。由于安全原因，命令行访问权限应限制为 SQL Server 服务帐户。默认情况下，SQL 命令行被禁用。

1. 若要在本地 SQL Server 上配置连接，请在 SQL Management Studio 中的 **“对象资源管理器”** 中，右键单击 **LONDON** 服务器，然后选择 **“新建查询”**。
1. 输入以下 Transact-SQL 代码：

    ```sql
    EXECUTE sp_configure 'show advanced options', 1;
    RECONFIGURE;
    EXECUTE sp_configure 'xp_cmdshell', 1;
    RECONFIGURE;
    GO
    EXECUTE xp_cmdshell 'net use U: \\<storageaccountname>.file.core.windows.net\backupshare /persistent:Yes /u:Azure\<storageaccountname> <storageaccountkey>';
    EXECUTE xp_cmdshell 'dir U:';
    ```

1. 在查询文本中，将 `<storageaccountname>` 替换为你先前创建的存储帐户的名称。名称必须在两个地方输入。
1. 将 `<storageaccountkey>` 替换为你为该存储帐户记录的主访问密钥。
1. 执行查询并检查结果中是否存在错误消息。

    > [注意！]
    > SQL 代码将网络驱动器 U: 映射到 Azure 中的存储帐户。但是，由于这是在 SQL 服务帐户的上下文中完成的，因此文件资源管理器中或使用 `net use` 命令时将看不到 U: 驱动器。

1. 将查询保存在“Labfiles”文件夹中，命名为 **“MapNetworkdrive.sql”**
1. 启动一个新的查询窗口，并通过运行以下查询在 LONDON SQL Server 上禁用 `xp_cmdshell`：

    ```sql
    EXECUTE sp_configure 'xp_cmdshell', 0;
    RECONFIGURE;
    ```

1. 关闭所有查询窗口，并且不保存任何文件。

### 将 Azure VM SQL 实例连接到文件共享

1. 若要在 Azure VM SQL Server 上配置连接，请在 **“对象资源管理器”** 中，右键单击 Azure 中的 SQL Server，然后选择 **“连接”**。
1. 在 **“连接到 SQL Server”** 对话框的 **“密码”** 文本框中键入 **“Pa55w.rdPa55w.rd”**，然后选择 **“连接”**。
1. 在 **“文件”** 菜单上，选择 **“打开/文件”**，然后打开上面保存的 **MapNetworkDrive.sql** 文件。
1. 在查询窗口底部的状态栏中，检查是否已连接到 Azure VM。
1. 若要映射 Azure VM 上的 U: 驱动器，请执行查询并检查结果中是否存在错误消息。
1. 启动新的查询窗口并通过运行以下查询禁用 `xp_cmdshell`：

    ```sql
    EXECUTE sp_configure 'xp_cmdshell', 0;
    RECONFIGURE;
    ```

1. 关闭 SQL Server Management Studio。

## 练习 4：使用 SQL Server 数据迁移助手执行数据库迁移

在本练习中，你将数据从本地 SQL Server 迁移到 Azure 中的 VM。

**预计用时：** 10 分钟

本练习的主要任务是：

1. 使用数据迁移助手迁移数据库。
1. 验证数据库是否成功迁移。

### 使用数据迁移助手迁移 SQL 数据库

1. 在 **LON-DEV-01** 虚拟机中，打开 **Microsoft 数据迁移助手**，然后选择 **“+”**。
1. 在 **“新建”** 页上，输入以下值：

    | 属性 | 值 |
    | --- | --- |
    | 项目类型 | 迁移 |
    | 项目名称 | 迁移到 Azure VM |
    | 源服务器类型 | SQL Server |
    | 目标服务器类型 | Azure 虚拟机上的 SQL Server |

1. 选择 **“创建”**。
1. 在 **“指定源和目标”** 页上的 **“源服务器详细信息”** 下，输入以下值：

    | 属性 | 值 |
    | --- | --- |
    | 服务器名称 | localhost |
    | 身份验证类型 | Windows 身份验证 |
    | 加密连接 | 否 |
    | 信任服务器证书 | 是 |

1. 在 **“目标服务器详细信息”** 下，输入以下值：

    | 属性 | 值 |
    | --- | --- |
    | 服务器名称 | 输入 Azure 中虚拟机的 IP 地址或 DNS 名称 |
    | 身份验证类型 | SQL Server 身份验证 |
    | 用户名 | sqladmin |
    | 密码 | Pa55w.rdPa55w.rd |
    | 加密连接 | 是 |
    | 信任服务器证书 | 是 |

1. 选择 **“下一步”**。
1. 在 **“添加数据库”** 页上，取消选择所有数据库，但 **AdventureWorks** 和 **AdventureWorksLT2008TR2** 除外。
1. 在 **“共享位置”** 文本框中，键入 **“U:\\”**，然后选择 **“下一步”**。
1. 查看 **“选择登录”** 窗口。没有要迁移的登录名。选择 **“StartMigration”**

    > [注意！]
    > 所有数据库都将备份到 Azure 存储文件共享中的共享网络驱动器。

1. 监视迁移过程。
1. 迁移完成后，关闭数据迁移助手。

### 验证是否成功迁移

1. 在 **LON-DEV-01** 虚拟机中，打开 **SQL Management Studio**。
1. 在 **“连接到服务器”** 对话框的 **“服务器名称”** 列表中，选择 Azure VM 的 IP 地址或 DNS 名称。
1. 在 **“密码”** 文本框中，键入 **“Pa55w.rdPa55w.rd”**，然后选择 **“连接”**。
1. 在对象资源管理器中，展开 **“数据库”** 列表。
1. 验证 **AdventureWorks** 数据库是否已成功迁移。
1. 从 **“文件”** 菜单中，选择 **“新建/使用当前连接查询”**。
1. 键入并执行此查询，以验证每个数据库的数据库兼容性级别。

    ```sql
    SELECT name, compatibility_level FROM sys.databases
    ```

1. 使用以下查询更改 **Adventureworks** 数据库的数据库兼容性级别：

    ```sql
    ALTER DATABASE AdventureWorks
    SET COMPATIBILITY_LEVEL = 110;
    GO
    ```

1. 使用以下查询备份 Adventureworks 数据库：

    ```sql
    BACKUP DATABASE Adventureworks  
    TO DISK = 'U:\Adventureworks'  
       WITH FORMAT,  
          MEDIANAME = 'Adventureworks',  
          NAME = 'Full Backup of Adventureworks';  
    ```

1. 成功完成备份后关闭 **SQL Management Studio**。

> [!重要事项]
> 本练习结束时，请勿删除在 Azure 中运行的 SQL Server VM。你在实验室 4 中会将其用作迁移到 Azure 数据库的源。

结果：你现在已将 SQL Server 数据库成功迁移到在 Azure VM 中运行的 SQL Server 2017。