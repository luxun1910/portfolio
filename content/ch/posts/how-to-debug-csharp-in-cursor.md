+++
title = '如何在 Cursor 中调试 C#'
date = 2025-05-09T14:58:54+08:00
draft = false
+++

## Cursor 中无法调试 C#！

当我在 Cursor 中编写 C# 控制台应用程序并按下调试按钮时，显示了以下错误。
![Cursor 错误信息（C# 只能在 VSCode 中调试）](/images/how-to-debug-csharp-in-cursor/error-message-by-cursor.png)

```txt
Unable to start debugging. .NET Debugging is supported only in Microsoft versions of VS Code. See https://aka.ms/VSCode-DotNet-DbgLicense for more information.
```

显然，.NET 调试在 Cursor 中不起作用...

本文记录了我是如何设法在 Cursor 中使 C# 调试和执行工作的。

## 解决方案

总而言之，我通过参考以下文章解决了问题：

- <https://guiferreira.me/archive/2025/how-to-debug-dotnet-code-in-cursor-ai/>
- <https://engincanveske.substack.com/p/debug-your-net-apps-in-cursor-code>

然而，在我的环境中，还有许多其他事情我必须调整才能使其工作，所以我将分享我所做的以作备忘。

### 先决条件

假设您已经安装了 C# 执行所需的扩展，例如，通过从 VSCode 迁移。

另外，我的执行环境如下：

- Windows 11
- Cursor 0.49.x
- PowerShell 7.5.0

### 1. 安装三星的 .NET Core 运行时调试器

三星似乎开源了一个 .NET Core 调试器。
原理是通过使用它，即使在 Cursor 中也可以进行调试和执行。

对于 Windows，请从下面的 URL 下载最新的 `netcoredbg-win64.zip` 并将其解压缩到任何位置。
<https://github.com/Samsung/netcoredbg/releases>

### 2. 编辑 .vscode/launch.json

我按如下方式配置了它：

```json
{
    "version": "0.2.0",
    "configurations": [
        {
            "name": "netcoredbg",
            "type": "coreclr",
            "request": "launch",
            "preLaunchTask": "dotnet: build",
            "program": "${workspaceFolder}\\ConsoleApp1\\bin\\Debug\\net9.0\\ConsoleApp1.dll",
            "args": [],
            "cwd": "${workspaceFolder}",
            "pipeTransport": {
                "pipeCwd": "${workspaceFolder}",
                "pipeProgram": "powershell",
                "pipeArgs": ["-Command"],
                "debuggerPath": "C:\\netcoredbg\\netcoredbg.exe",
                "debuggerArgs": ["--interpreter=vscode"],
                "quoteArgs": true
            },
            "env": {
                "DOTNET_ENVIRONMENT": "Development"
            }
        }
    ]
}
```

不同网站的信息各不相同，但以上配置在我的环境中有效。

以下是一些需要注意的点：

- 为 `preLaunchTask` 指定 `dotnet: build` 而不是 `build`
  - 无需创建 .vscode/tasks.json
- 对于 `program`，指定要运行的应用程序的 .dll 文件
  - 通常，它应该是 `ApplicationName.dll` 格式
- 对于 `pipeTransport.debuggerPath`，指定在步骤 1 中下载的 netcoredbg.exe 的文件路径
  - .exe 可以省略

### 3. 执行

从左侧菜单的 `Run and Debug` 中选择 `netcoredbg` 并执行它。

![C# 调试在 Cursor 中工作](/images/how-to-debug-csharp-in-cursor/csharp-debug-works-in-cursor.png)

它成功运行了！

## 总结

通过安装三星的调试器，即使在 Cursor 中也可以调试 C#。
C# 是一种可用于各种框架的语言，包括 Web 开发、游戏开发、Windows 应用程序和移动应用程序。
能够在 Cursor 中运行 C# 使您可以在利用 AI 辅助的同时开发和调试 C#，从而极大地扩展了您的开发可能性。
理想情况下，微软会在 Cursor 中启用 C# 调试，但目前，此方法是绕过限制的好方法。 