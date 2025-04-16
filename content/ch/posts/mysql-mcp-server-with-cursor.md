+++
title = '在Cursor中使用MySQL MCP服务器'
date = 2025-04-15T11:56:28+08:00
draft = false
+++

## 引言

大家是否正在使用Cursor呢？
我最近开始使用它，发现非常方便，因此订阅了Pro会员。

最近，我了解到了一种称为MCP服务器的工具，它可以连接AI聊天工具与外部工具。
我尝试设置了MySQL的MCP服务器，能够直接从Cursor的聊天界面访问MySQL数据库。
这使Cursor能够直接访问数据库并为您执行查询。
在出现错误时，它可以访问数据库获取数据或修改表定义。
非常方便。

在本文中，我将介绍如何设置MySQL MCP服务器以及如何从Cursor访问它，部分作为自己的记录。

## 什么是MCP服务器？

那么，什么是MCP服务器呢？根据我的理解，它是连接"AI生成工具"和"外部工具"的系统。

以前已经存在类似的机制，如ChatGPT的Function Calling和插件。
然而，这些机制对现在存在的众多其他生成式AI（如Claude、DeepSeek等）并不通用。
因此，当尝试与其他生成式AI连接时，你必须每次重写代码。

随着MCP服务器的出现，这些标准得到了统一。
换句话说，一旦你按照标准编写了连接生成式AI和外部工具的代码，无论是Claude、DeepSeek还是其他工具，你都可以无限期地重复使用它。

如果我在这部分过多展开，会偏离本文的主题，所以如果您想了解更多，请参考以下文章：
<https://zenn.dev/nanami_bitwise/articles/966efd5438e75b>

## 使用MySQL MCP服务器

对于本文，我将使用在网上搜索时排名第一的MCP服务器：

<https://github.com/designcomputer/mysql_mcp_server>

请使用您喜欢的方法下载：GitHub Desktop、命令行、Visual Studio等。

### 前提条件：执行环境

我的执行环境如下：

- Windows 11
- Cursor 0.48.9
- PowerShell 7.5.0
- 方法1需要
  - Python 3.13
  - pip 25.0.1
  - uv 0.6.13
- 方法2需要
  - npm 10.9.2

要使用MCP服务器，您需要Python、pip和uv，或者npm。
请确保您能在本地环境中运行这些工具。

### 方法1：在本地下载并运行MCP服务器应用程序

首先，在本地运行以下命令。
由于稍后将使用执行此命令的路径，请将其保存在您方便管理的位置。

```pws
pip install mysql-mcp-server
```

接下来，从Cursor的设置中，选择MCP→Add new global MCP server，并按如下配置：

```json
{
  "mcpServers": {
    "mysql": {
      "command": "uv的完整路径",
      "args": [
        "--directory",
        "src\\mysql_mcp_server的完整路径",
        "run",
        "mysql_mcp_server"
      ],
      "env": {
        "MYSQL_HOST": "主机名",
        "MYSQL_PORT": "3306（根据您的主机设置调整）",
        "MYSQL_USER": "用户名",
        "MYSQL_PASSWORD": "密码",
        "MYSQL_DATABASE": "数据库名称"
      }
    },
  }
}
```

以下是几个需要注意的点：

#### 指定uv的完整路径

在Windows环境中，如果只写"uv"，会出现找不到命令的错误。
因此，您需要指定uv命令（uv.exe）的完整路径。
它应该看起来像`C:\\Users\\用户名\\.local\\bin\\uv`。

#### 关于路径表示法

在Windows环境中，您通常应使用\（反斜杠），但似乎当前的Windows shell能很好地处理转换。
因此，/（正斜杠）也能被正常识别。
我为了安全起见使用了双反斜杠（用于JSON转义），但您可以使用您喜欢的方式。
我希望将来能进一步验证这一部分。

#### MySQL主机名和数据库名

我曾误解了这部分，花了相当多的时间。
我尝试连接的数据库URL在程序中如下所示：

jdbc:mysql://hoge.com:3306/fuga

这个字符串表示使用jdbc通过mysql连接到hoge.com上的fuga表。
所以在这种情况下，主机名是hoge.com，数据库名是fuga。
在Cursor设置文件中，您需要为MYSQL_HOST指定hoge.com，为MYSQL_DATABASE指定fuga。

### 方法2：通过Smithery启动

这种方法应该消除了本地下载数据的需要，但在我的环境中运行不太好。
如果您想尝试，请查看这里：

<https://smithery.ai/server/mysql-mcp-server>

## 验证

首先，检查Cursor的MCP设置部分是否如下所示：
如果有绿色圆圈，通常就没问题。

![Cursor的MCP设置截图](/images/mysql-mcp-server-with-cursor/mysql-mcp-setting.png)

接下来，在Cursor的聊天中，尝试询问如下内容：

`请从XX表中检索数据`

如果它按如下方式返回数据，则您已成功：

![使用Cursor检索MySQL数据的截图](/images/mysql-mcp-server-with-cursor/mysql-mcp-screen-shot.png)

## 如果它不工作

按`Ctrl + Shift + U`打开"输出"，并从右上角的下拉菜单中选择"Cursor MCP"。
那里会输出MCP相关的日志，这应该能为解决错误提供线索。

另外，如果它仍然不工作，请尝试在Cursor的MCP设置中按更新按钮或重启Cursor。
由于Cursor是一个仍在积极开发中的编辑器，这种行为可能有些不稳定。

## 结论

虽然我发现Cursor非常方便，但我想知道它是否也能自动处理数据库操作。
使用这个MCP服务器，AI现在可以为您执行数据库操作。
在实际业务应用中使用时，您需要考虑安全性和幻觉等问题，但我认为这是一个具有巨大潜力的优秀MCP服务器。
