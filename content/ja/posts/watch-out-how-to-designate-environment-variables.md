+++
title = 'Windowsのターミナルでは環境変数の指定方法に気をつけろ'
date = 2026-02-15T08:36:55+08:00
draft = false
+++

## 起きた事象

Flutterでアプリケーションを作っていた。
ログインのためにFirebase認証を使っていて、Webでは認証成功するのに、Androidアプリでデバッグした途端、以下のエラーになった。

`PlatformException(sign_in_failed, com.google.android.gms.common.api.ApiException: 10:, null, null)`

どうやら各アプリでFirebase認証を使うには、当該アプリの署名証明書SHA-1を取得する必要がある、ということが分かった。

署名証明書はどう取得するのか調べてみたところ、公式ドキュメントにたどり着いた。

<https://developers.google.com/android/guides/client-auth?hl=ja>

公式ドキュメントには、デバッグ用アプリケーションについて、Windowsでは以下のように取得すると書いてあった。

```shell
keytool -list -v -alias androiddebugkey -keystore %USERPROFILE%\.android\debug.keystore
```

その通りにCursorのターミナルで実行してみたところ...

```shell
keytoolエラー: java.lang.Exception: キーストア・ファイルは存在しません: %USERPROFILE%\.android\debug.keystore
java.lang.Exception: キーストア・ファイルは存在しません: %USERPROFILE%\.android\debug.keystore
        at java.base/sun.security.tools.keytool.Main.doCommands(Main.java:920)
        at java.base/sun.security.tools.keytool.Main.run(Main.java:419)
        at java.base/sun.security.tools.keytool.Main.main(Main.java:400)
```

？？？

試しに、ファイルエクスプローラーに`%USERPROFILE%\.android`と打ってみると、debug.keystoreは存在していた。

？？？？？？？？？

## 原因と解決策

どうやら、上記のコマンドはWindowsのコマンドプロンプト向けの書き方だったようだ。
自分はCursorのターミナルにPower Shellを使っていたため、エラーになった。

そもそも`USERPROFILE`は環境変数として定義されている。
コマンドプロンプトでは`%USERPROFILE%`の形式で環境変数を取得できるが、Power Shellでは`$env:USERPROFILE`と書く必要があるらしい（参考: <https://qiita.com/naka_toshinori/items/c9ea1efc89686ed3d018>）。

Power Shellで`%USERPROFILE%`と書いても、ただの文字列として認識され、`%USERPROFILE%\.android\debug.keystore`を探しに行った結果、そんなファイルは存在しない！と怒られたということだ。
試しに、Power Shellで以下のようにコマンドを打つと、成功した。

```shell
keytool -list -v -alias androiddebugkey -keystore $env:USERPROFILE\.android\debug.keystore
```

また、コマンドプロンプトでは、公式ドキュメントの書き方通りで問題なく動いた。

Mac/Linuxでの開発が盛んなアプリケーションやライブラリの場合、Windowsでのセットアップ方法については、それがPower Shellでの書き方なのか、コマンドプロンプトでの書き方なのか明記されていないことが多々ある気がする。
今後同じようなパターンに遭遇する可能性があり、今回の件をしっかり記憶に留めたい。
