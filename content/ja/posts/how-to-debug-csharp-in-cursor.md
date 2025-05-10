+++
title = 'CursorでC#のデバッグをする方法'
date = 2025-05-09T14:58:54+08:00
draft = false
+++

## CursorではC#のデバッグができない！

CursorでC#のコンソールアプリを書き、デバッグボタンを押したところ、以下のエラーが出力されました。
![Cursorのエラーメッセージ（C#はVSCodeでのみデバッグ可）](/images/how-to-debug-csharp-in-cursor/error-message-by-cursor.png)

```txt
Unable to start debugging. .NET Debugging is supported only in Microsoft versions of VS Code. See https://aka.ms/VSCode-DotNet-DbgLicense for more information.
```

なんと、どうやらCursorでは.NETのデバッグができないらしい...。

この記事は、どうにかしてCursorでC#をデバッグ・実行できるようにするまでの記録です。

## 解決方法

結論から言うと、以下の記事を参考にすることで解決しました。

- <https://guiferreira.me/archive/2025/how-to-debug-dotnet-code-in-cursor-ai/>
- <https://engincanveske.substack.com/p/debug-your-net-apps-in-cursor-code>

しかし自分の環境だと、他にも色々工夫しなければ上手くいかないことも多々あったので、備忘録も兼ねて自分が何をしたか共有します。

### 前提

前提として、VSCodeから移行するなどして、通常C#の実行に必要な拡張機能は入っているものとします。

また、自分の実行環境は以下の通りです。

- Windows 11
- Cursor 0.49.x
- PowerShell 7.5.0

### 1. Samsungの.NET Core Runtimeデバッガーを導入する

どうやらあのSamsungが、.NET Coreのデバッガーをオープンソースで公開しているらしいです。
こちらを使用すればCursorでもデバッグ・実行が可能になる、という原理です。

Windowsの場合は、以下のURLから最新版の`netcoredbg-win64.zip`をダウンロードし、任意の場所に解凍してください。
<https://github.com/Samsung/netcoredbg/releases>

### 2. .vscode/launch.jsonを編集する

以下のように設定しました。

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

サイトによって書いてある内容にバラツキがあるのですが、自分の環境では上記で上手くいきました。

以下注意点です。

- `preLaunchTask`には`build`ではなく`dotnet: build`を指定する
  - .vscode/tasks.jsonを作成する必要もない
- `program`には実行したいアプリケーションの.dllファイルを指定する
  - 通常は`アプリケーション名.dll`という形式になるはず
- `pipeTransport.debuggerPath`には、手順1でダウンロードしたnetcoredbg.exeへのファイルパスを指定する
  - .exeは省略可

### 3. 実行する

左メニューの`Run and Debug`から`netcoredbg`を選択して、実行します。

![CursorでC#が動いた瞬間](/images/how-to-debug-csharp-in-cursor/csharp-debug-works-in-cursor.png)

無事に動きました！

## まとめ

Cursorでも、Samsungのデバッガーを導入することでC#のデバッグができるようになります。
C#はWeb開発をはじめ、ゲーム制作やWindowsアプリケーション、モバイルアプリなど、様々なフレームワークで活用できる言語です。
CursorでC#を動かせるようになることで、AIアシストを活用しながらC#の開発・デバッグができるようになり、開発の幅が大きく広がります。
本当はMicrosoftがCursorでもC#のデバッグができるようにしてくれるのが一番ではありますが、しばらくはこの方法で制限を回避するのが良いでしょう。
