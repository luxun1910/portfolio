+++
title = 'Windows 终端中指定环境变量时要注意写法'
date = 2026-02-15T08:36:55+08:00
draft = false
+++

## 发生的情况

我在用 Flutter 做应用。
登录用了 Firebase 认证，在 Web 上能成功，但一在 Android 应用上调试就出现下面的错误：

`PlatformException(sign_in_failed, com.google.android.gms.common.api.ApiException: 10:, null, null)`

查了一下发现，各应用使用 Firebase 认证时，需要取得该应用的签名证书 SHA-1。

查了如何取得签名证书后，找到了官方文档。

<https://developers.google.com/android/guides/client-auth?hl=zh-cn>

官方文档里写的是，在 Windows 上获取调试用应用的证书可以这样执行：

```shell
keytool -list -v -alias androiddebugkey -keystore %USERPROFILE%\.android\debug.keystore
```

我按文档在 Cursor 的终端里执行后……

```shell
keytool错误: java.lang.Exception: 密钥库文件不存在: %USERPROFILE%\.android\debug.keystore
java.lang.Exception: 密钥库文件不存在: %USERPROFILE%\.android\debug.keystore
        at java.base/sun.security.tools.keytool.Main.doCommands(Main.java:920)
        at java.base/sun.security.tools.keytool.Main.run(Main.java:419)
        at java.base/sun.security.tools.keytool.Main.main(Main.java:400)
```

？？？

试着在文件资源管理器里输入 `%USERPROFILE%\.android`，发现 debug.keystore 是存在的。

？？？？？？？？？

## 原因与解决办法

原来上面的命令是面向 Windows 命令提示符的写法。
我使用的是 Cursor 终端里的 PowerShell，所以会报错。

`USERPROFILE` 本身是环境变量。在命令提示符里可以用 `%USERPROFILE%` 的形式引用，而在 PowerShell 里需要写成 `$env:USERPROFILE`（参考：<https://qiita.com/naka_toshinori/items/c9ea1efc89686ed3d018>）。

在 PowerShell 里写 `%USERPROFILE%` 会被当成普通字符串，于是去查找名为 `%USERPROFILE%\.android\debug.keystore` 的文件，结果当然找不到。
在 PowerShell 里改成下面这样执行就成功了：

```shell
keytool -list -v -alias androiddebugkey -keystore $env:USERPROFILE\.android\debug.keystore
```

在命令提示符里按官方文档的写法执行是没有问题的。

以 Mac/Linux 开发为主的应用或库，Windows 的配置说明里经常不写清楚是 PowerShell 写法还是命令提示符写法。以后很可能还会遇到类似情况，所以想把这次的事记下来。
