+++
title = 'Flutterのproject_nameはiOSとAndroidどちらにも対応するものにしないと後々面倒になるという話'
date = 2025-07-17T12:27:19+08:00
draft = false
+++

## なぜこの記事を書くに至ったか

Flutter製のアプリにFirebaseを導入するため、`flutterfire configure`を実行し、iOS、Android、Web等全てを含めたところ、iOSの段階で失敗した。

```bash
i Firebase android app com.unanimousworks.nonsense_generator registered.
i Firebase ios app com.unanimousworks.nonsense_generator is not registered on Firebase project nonsensegenerator-d140d.    
⠦ Registering new Firebase ios app on Firebase project nonsensegenerator-d140d.
FirebaseCommandException: An error occured on the Firebase CLI when attempting to run a command.
COMMAND: firebase apps:create ios nonsense_generator (ios) --bundle-id=com.unanimousworks.nonsense_generator --json --project=nonsensegenerator-d140d
```

エラーログを見てみると、以下のように出力されていた。

```log
[debug] [2025-07-16T05:11:47.788Z] >>> [apiv2][body] POST {firebaseのapiのURL} {"displayName":"nonsense_generator (ios)","bundleId":"com.unanimousworks.nonsense_generator"}
[debug] [2025-07-16T05:11:48.587Z] <<< [apiv2][status] POST {firebaseのapiのURL} 400
[debug] [2025-07-16T05:11:48.587Z] <<< [apiv2][body] POST {firebaseのapiのURL} {"error":{"code":400,"message":"Request contains an invalid argument.","status":"INVALID_ARGUMENT"}}
[debug] [2025-07-16T05:11:48.588Z] Request to {firebaseのapiのURL} had HTTP Error: 400, Request contains an invalid argument.
[debug] [2025-07-16T05:11:48.889Z] FirebaseError: Request to {firebaseのapiのURL} had HTTP Error: 400, Request contains an invalid argument.
```

Request contains an invalid argumentらしい。
ここで渡しているリクエストボディは以下の2つ。

- `"displayName":"nonsense_generator (ios)"`
- `"bundleId":"com.unanimousworks.nonsense_generator"`

よって、このどちらかがおかしいということになる。
displayNameが特段問題になるようなことはない気がしたので、bundleIdが怪しいと考え、調べてみた。

## iOSのBundle IDの定義

`英数字（A-Z、a-z、0-9）、ハイフン（-）、ピリオド（.）のみで構成されている必要があります。これはReverse DNSフォーマットの文字列である必要があります。`

参考：<https://developer.apple.com/jp/help/glossary/bundle-id/>

今回、bundleIdは`com.unanimousworks.nonsense_generator`としていたので、アンダーバー（_）が含まれてしまっている。
そのためにエラーとなったようだ。

[Change App Package Name for Flutter](https://pub.dev/packages/change_app_package_name)を用いて、以下のコマンドでiOSのBundle IDのみを、基準に合うように修正した。
`dart run change_app_package_name:main com.new.package.name --ios`

## AndroidのApplication IDの定義

ついでにAndroidのApplication IDの定義も調べてみた。

```md
- 2つ以上のセグメント（1つ以上のドット）が必要
- 各セグメントは文字で始まる必要がある
- 使用できる文字は英数字と下線のみ（a～z、A～Z、0～9、_）
```

参考：<https://developer.android.com/build/configure-app-module?hl=ja#set-application-id>

## 結論

Flutterプロジェクトを新規作成するとき、`flutter create project_name`を叩くと、Bundle ID、Application IDはともに`com.example.project_name`となる。

AndroidではApplication IDに_は使えるが、iOSのBundle IDでは使えない。
逆に、iOSのBundle IDに-は使えるが、AndroidのApplication IDでは使えない。

Flutterでアプリ作成している時点で、ほとんどの場合iOSとAndroidどちらにも公開しようと考えているはずなので、Flutterのプロジェクト名は英数字のみで構成するのが吉だと思われる。

