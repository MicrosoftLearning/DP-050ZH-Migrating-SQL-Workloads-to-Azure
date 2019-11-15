---
lab:
    title: '实验室 3 - 在 Azure 虚拟机中将 SQL 工作负荷迁移到 SQL Server'
    module: '模块 3：将 SQL 工作负荷迁移到 Azure 虚拟机'
---

# DP-050 – 将 SQL 工作负荷迁移到 Azure

## 实验室 3 - 在 Azure 虚拟机中将 SQL 工作负荷迁移到 SQL Server

**预计用时：** 60 分钟
**先决条件：** 没有可在此实验室中执行的先决条件步骤。
**实验室文件：** 这个实验室没有实验室文件

## 实验室概述

学生将首先评估他们将用于从本地 SQL Server 2008 R2 实例迁移到在虚拟机中运行的 SQL Server 2017 的迁移过程。然后，他们将使用数据迁移助手执行迁移，以使用数据迁移助手移动数据库。  最后，他们将评估成功的迁移。

## 实验室目标

在本实验室结束时，你将可以：

1. 在 Azure 上创建一个新的 SQL Server 2017 虚拟机
2. 创建 Azure 存储帐户和文件共享
3. 将 SQL Server 2008 R2 数据库迁移到 Azure VM 中的 SQL Server

## 方案

你是负责 AdventureWorks 的高级数据库工程师，正在为数据现代化项目做准备。你将准备必要的环境以将一组数据库迁移到 Azure 虚拟机中的 SQL Server，并使用数据迁移助手执行测试迁移。

## 练习 1：在 Azure 上创建新的 SQL Server 2017 虚拟机

在该练习中，你将使用 Azure 门户在 Azure 上创建一个新的虚拟机。

**预计用时：** 20 分钟

本次练习的主要任务如下：

1. 在 Azure 门户中创建新的虚拟机

### 任务 1：预配 SQL Server 2017 虚拟机

**注意：** 如果你在托管实验室环境中运行此实验室，请在预配的实验室环境中执行这些步骤

1. 在 [Azure门户](https://portal.azure.com) 中，选择“**创建资源**”，然后在“商城”搜索框中键入“**Windows 2016 上的 SQL Server 2017**”。
1. 选择“**免费 SQL Server 许可证：Windows Server 2016 上的 SQL Server 2017 Developer**” “**选择软件计划**”下拉菜单中的。
1. 使用预设配置选择“开始”。
1. 在“**选择工作负荷环境**”中选择“**开发/测试**”。
1. 在“**选择工作负荷类型**”中，保留默认项“**常规用途 (D 系列)**”。
1. 选择“**继续创建 VM**”。
1. 在“项目详细信息”窗口中，选择“**新建**”以创建新的资源组。
1. 指定“**DP-050-Training**”作为资源组的名称。
1. 通过提供以下详细信息提供实例详细信息：

    1. 虚拟机名称：**sql2017vm**
    1. 区域：**选择距离您实际位置最近的区域**
    1. 图像：**免费 SQL Server 许可证：Windows Server 2016 上的 SQL Server 2017 Developer**。
    1. 用户名：**sqladmin**
    1. 密码：**Pa55w.rd.123456789**
    1. 确认密码：**Pa55w.rd.123456789**

    **防火墙设置：**

      a. 公共入站端口：选择“允许选定的端口”

      b. 从“选择入站端口”下拉菜单中选择 RDP。

10. 选择 **下一步：磁盘**

    查看磁盘设置，接受但不做任何更改。

1. 选择 **下一步：网络**
1. 选择 **下一步：管理**

    查看管理设置，然后更改启动诊断：

    a. 将启动诊断设置为“**关**”

13. 选择 **下一步：高级**
1. 跳过“高级”选项卡并选择“**SQL Server 设置**”

    在 SQL Server 设置选项卡上，填写以下信息：

    a. SQL 连接性：选择“**公开 (Internet)**”
    b. SQL 身份验证：**选择“启用”**
    c. 在密码框中键入 **Pa55w.rd.123456789**

15. 选择“**查看 + 创建**”

    查看“查看 + 创建”选项卡上的设置

    a. 选择“**创建**”以启动虚拟机 (VM) 创建。

**注意：**这一步大约需要 10 分钟才能完成

16.	完成 VM 创建后，打开虚拟机边栏选项卡。
17.	 选择你创建的 **sql2017vm**。
18.	找到公共 IP 地址并将其写下来以便将来连接
19.	选择“**配置 DNS 名称**”
20.	键入唯一可识别 DNS 名并记下你指定的完整 DNS 名。

    例如：SQL2017VMxxxx.centralus.cloudapp.azure.com

21.	单击“保存”

```shell
结果：完成此练习后，你即拥有了在 Azure 虚拟机中运行的 SQL Server 2017 实例。
```

## 练习 2：创建 Azure 存储帐户和文件共享

在该练习中，你将使用 Azure 门户在 Azure 上创建一个新的虚拟机。

**预计用时：** 15 分钟

本次练习的主要任务如下：

1. 创建 Azure 存储帐户
2. 在 Azure 存储帐户上创建文件共享

### 任务 1：创建 Azure 存储帐户

1. 在 [Azure 门户](https://portal.azure.com)，选择“**存储帐户**”边栏选项卡。
2. 选择“**添加**”。
3. 在“创建存储帐户”窗口中，填写以下信息：
    a. 资源组：选择“**现有**”
    b. 选择你在上一个联系中创建的“**DP-050-Training**”资源组
    c. 存储帐户名：**dp050storagexxxx**（其中 xxxx 是随机字符数）
    d. 位置：选择与在上一个练习中创建虚拟机的位置最接近的位置。
    e. 保留其他默认设置。
4. 单击“**查看 + 创建**”以跳过“高级”和“标记”部分
5. 选择“**创建**”

    **注意：**  此部署可能需要几分钟时间

6. 完成后，单击“**转到资源**”

### 任务 2：创建 Azure 文件共享

1. 在“存储帐户”页面上，选择“**文件**”
2. 在“文件”页面上，选择“**+ 文件共享**”
3. 填写下列信息：
    a. 名称： **backupshare**
    b. 配额： **200 Gib**
4. 选择“**创建**”
5. 创建文件共享后，选择已创建的文件共享右侧的“…”
6. 从下拉式列表中选择“**连接**”

![下拉列表](https://github.com/MicrosoftLearning/DP-050-Migrating-SQL-Workloads-to-Azure/blob/master/images/dropdown.png)

7. 在“连接”边栏选项卡中，选择驱动器号“**U:**”
8. 从下面列出的文本中复制连接命令语法。或者，如果它们的键不以正斜杠开头，请运行此命令：

该文本看起来像下面的命令语法：

```shell
cmdkey /add:dp050storagexxxx.file.core.windows.net /user:Azure\dp050storagexxxx /pass:Hcty8OXn7jcON/ePk3OvswD7eRusfWWAw9lakhX9P9m4MnuqZEt2I8pDYAEiyAZJx4Na39UZEwAyH8PlnVa36Q==

```

9. 打开“**记事本**”
10. 粘贴上述命令的内容并将文件以“**MapNetworkdrive.txt**”为名保存到“**Labfiles**”文件夹
11. 关闭“**记事本**”。
12.	你现在可以关闭 Azure Portal 记事本。

```shell
结果：你现在已成功创建了 Azure 文件共享，它将用作 SQL Server 数据库备份文件的共享访问位置。在下一个练习中，你将配置 SQL 实例以便能够访问共享位置。
```

## 练习 3：为 SQL Server 实例创建连接以连接到 Azure 文件共享

在本练习中，你将配置 SQL Server 环境以便能够访问 Azure 文件共享
**预计用时：** 10 分钟

本次练习的主要任务如下：

1. 通过映射网络驱动器，通过 SQL Management Studio 注册文件共享
2. 在 Azure 存储帐户上创建文件共享

### 任务 1：在 SQL Management Studio 中注册服务器实例

1. 在 SQL Server 2008 R2 实验室环境中，启动 SQL Management Studio
2. 连接到本地实例（伦敦）
3. 在 SQL Management Studio 对象资源管理器中，选择“**连接 | 数据库引擎**”并在“**连接到服务器**”对话框中填写以下信息：
    服务器名：“SQL 2017 VM 完全限定的域名”
    例如 sql2017vmxxxx.centralus.cloudapp.azure.com
4. 选择 SQL Server 身份验证并提供以下用户和密码
    登录：**sqladmin**
    密码：**Pa55w.rd.123456789**

### 任务 2：将 SQL 实例连接到文件共享

**注意：**为了使 SQL Server 能够连接到驻留在文件共享上的驱动器号，你必须映射运行 xp_cmdshell 的网络驱动器。由于安全原因，应限制 SQL Server 服务帐户的命令行访问。默认情况下，SQL 命令行被禁用。

1. 在记事本中打开你在上一部分练习中创建的 MapNetworkdrive.txt。
2.	复制文本
3.	在 SQL Management Studio 中，在连接到本地服务器（伦敦）时创建新查询
4.	将 MapNetworkdrive 中的文本粘贴到查询窗口中
5.	更改查询以反映以下屏幕截图，并通过 xp_cmhdshell 存储过程运行命令行。
 ![sp_configure 选项](https://github.com/MicrosoftLearning/DP-050-Migrating-SQL-Workloads-to-Azure/blob/master/images/cmdshell.png)
6.	运行查询并验证网络驱动器是否可访问
7.	将查询保存在“Labfiles”文件夹中，命名为“**MapNetworkdrive.sql**”
8.	启动新的查询窗口并通过运行以下查询禁用 xp_cmdshell：

```sql
    SP_CONFIGURE ‘xp_cmdshell’,1
```

9.	在“对象资源管理器”中，选择 Azure VM SQL Server 实例并创建新查询
10.	打开保存的文件“Mapnetworkdrive.sql”并运行 SQLS Server 2017 中的脚本
11.	重复此步骤以禁用 SQL Server 2017 实例中的 xp_cmdshell。


## 练习 4：使用 SQL Server 数据执行数据库迁移 

在该练习中，你将使用 Azure 门户在 Azure 上创建一个新的虚拟机。
**预计用时：** 10 分钟

本次练习的主要任务如下：

1. 迁移数据库数据迁移助手
2. 验证数据库的成功迁移

### 任务 1：使用数据迁移助手迁移 SQL 数据库

1.	在 SQL Server 2008 R2 实验室环境中，打开应用程序“**Microsoft 数据迁移助手**”
2.	选择“**+**”，这将打开新项目的对话框，键入以下信息：
    a.	项目类型：**迁移**
    b.	项目名：**迁移到 SQL VM**
    c.	源服务器类型：**SQL Server**
    d.	目标服务器类型：**Azure 虚拟机上的 SQL Server**
3.	单击“**创建**”
4.	单击“**下一步**”
5.	在源服务器详细信息“服务器名”对话框中，输入“**localhost 的服务器名**”
6.	在目标服务器详细信息“服务器名”对话框中，使用其完全限定名输入 Azure SQL 虚拟机的服务器名
7.	在目标服务器“身份验证类型”对话框中，选择“**SQL Server 身份验证**”
8.	在目标服务器详细信息 SQL 身份验证凭据中，键入“**sqladmin**”并提供“**Pa55.w.rd.123456789**”作为密码。
9.	在“连接属性”中，取消选中“**加密连接**”属性。
10.	单击**下一步**。
11.	取消选择以下数据库：
    •	AdventureworksDW2008_4M
    •	Reportserver
    •	ReportServerTempDB
12.	在“共享位置”对话框中键入： **U:\**
13.	查看选择登录窗口
14.	单击“**开始迁移**”

**注意：**所有数据库都将备份到共享网络驱动器（Azure 文件共享）

15.	监测迁移过程。
16.	完成后，关闭数据迁移助手

### 任务 2：验证成功迁移

1.	在 SQL Server 2008 R2 实验室环境中，打开“**SQL Management Studio**”
2.	在“对象资源管理器”中展开“**SQL Server 2017 实例**”上的“**数据库**”列表
3.	验证数据库是否已成功迁移
4.	创建“**新查询**”
5.	键入并执行以下查询以验证每个数据库的数据库兼容级别。

```sql
SELECT name, compatibility_level FROM sys.databases
```

6.	查看查询结果
7.	使用以下查询更改 **AdventureworksLT2008R2** 数据库的数据库兼容级别：

```sql
ALTER DATABASE AdventureWorksLT2008R2
SET COMPATIBILITY_LEVEL = 110;
GO
```

8.	使用以下查询备份 AdventureworksLT2008R2 数据库：
  
```sql
BACKUP DATABASE AdventureworksLT2008R2  
TO DISK = 'U:\AdventureworksLT2008R2'  
   WITH FORMAT,  
      MEDIANAME = 'AdventureworksLT2008R2',  
      NAME = 'Full Backup of AdventureworksLT2008R2;  
```


9.	成功完成备份后关闭 **SQL Management Studio**。

```shell
结果：你现在已将 SQL 2008R2 数据库成功迁移到在 Azure VM 中运行的 SQL Server 2017。
```
