+++
title = '为什么Flutter的project_name应该同时兼容iOS和Android，以避免日后麻烦'
date = 2025-07-17T12:27:19+08:00
draft = false
+++

## 为什么要写这篇文章

为了给 Flutter 应用引入 Firebase，我执行了 `flutterfire configure` 并包含了所有平台（iOS、Android、Web 等），但在 iOS 阶段失败了。

```bash
i Firebase android app com.unanimousworks.nonsense_generator registered.
i Firebase ios app com.unanimousworks.nonsense_generator is not registered on Firebase project nonsensegenerator-d140d.    
⠦ Registering new Firebase ios app on Firebase project nonsensegenerator-d140d.
FirebaseCommandException: An error occured on the Firebase CLI when attempting to run a command.
COMMAND: firebase apps:create ios nonsense_generator (ios) --bundle-id=com.unanimousworks.nonsense_generator --json --project=nonsensegenerator-d140d
```

查看错误日志，显示如下输出：

```log
[debug] [2025-07-16T05:11:47.788Z] >>> [apiv2][body] POST {Firebase API URL} {"displayName":"nonsense_generator (ios)","bundleId":"com.unanimousworks.nonsense_generator"}
[debug] [2025-07-16T05:11:48.587Z] <<< [apiv2][status] POST {Firebase API URL} 400
[debug] [2025-07-16T05:11:48.587Z] <<< [apiv2][body] POST {Firebase API URL} {"error":{"code":400,"message":"Request contains an invalid argument.","status":"INVALID_ARGUMENT"}}
[debug] [2025-07-16T05:11:48.588Z] Request to {Firebase API URL} had HTTP Error: 400, Request contains an invalid argument.
[debug] [2025-07-16T05:11:48.889Z] FirebaseError: Request to {Firebase API URL} had HTTP Error: 400, Request contains an invalid argument.
```

显示"Request contains an invalid argument"。
这里传递的请求体包含以下两个参数：

- `"displayName":"nonsense_generator (ios)"`
- `"bundleId":"com.unanimousworks.nonsense_generator"`

因此，其中一个肯定有问题。
我觉得displayName应该没什么特别的问题，所以怀疑是bundleId，于是调查了一下。

## iOS Bundle ID 的定义

`套装 ID 字符串应为统一类型标识符 (UTI)，只能包含字母数字字符 (A-Z、a-z、0-9)、连字符 (-) 和/或英文句点 (.)。这个字符串应采用反向 DNS 格式。`

参考：<https://developer.apple.com/cn/help/glossary/bundle-id/>

在这种情况下，bundleId 是 `com.unanimousworks.nonsense_generator`，包含了下划线（_）。
这似乎就是导致错误的原因。

我使用了 [Change App Package Name for Flutter](https://pub.dev/packages/change_app_package_name) 工具，通过以下命令仅修正iOS的Bundle ID使其符合要求：
`dart run change_app_package_name:main com.new.package.name --ios`

## Android Application ID的定义

顺便也调查了一下Android Application ID的定义。

```md
- 必须至少包含两段（一个或多个圆点）。
- 每段必须以字母开头。
- 所有字符必须为字母数字或下划线 [a-zA-Z0-9_]。
```

参考：<https://developer.android.com/build/configure-app-module?hl=zh-cn#set-application-id>

## 结论

创建新的Flutter项目时，运行`flutter create project_name`会将Bundle ID和Application ID都设置为 `com.example.project_name`。

Android的Application ID允许使用下划线（_），但iOS的Bundle ID不允许。
相反，iOS的Bundle ID允许使用连字符（-），但Android的Application ID不允许。

由于大多数开发Flutter应用的开发者都计划同时发布到iOS和Android平台，所以Flutter项目名最好只使用字母数字字符。 
