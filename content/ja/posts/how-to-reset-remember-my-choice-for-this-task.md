+++
title = 'VSCodeでのビルド設定エラー時に出てくる"Remember my choice for this task"をリセットする方法'
date = 2025-05-16T09:28:59+08:00
draft = false
+++

## 記事執筆のきっかけ

[CursorでC#のデバッグができるよう試行錯誤](/ja/posts/how-to-debug-csharp-in-cursor/)していて、ビルドボタンを押した際、以下のようなエラーが出ました。

![Build error](/images/how-to-reset-remember-my-choice-for-this-task/error-could-not-find-the-task.png)

ここで、自分は良く分からず誤って「Remember my choice for this task」を押下したのち、「Debug Anyway」を押してしまいました。
その結果、ビルド設定にミスがあるにもかかわらず、ダイアログが出なくなったため、何がエラーの原因なのか分からなくなってしまいました（反省...）。

そこでこの設定を解消しようとしたところ、VSCodeのsettings.jsonに記載されているわけではないようで、解消するのに苦労しました。
おまけに、非常にマイナーなケースであるためか、調べても解決方法が中々出てこず、ChatGPTも嘘情報を教えてくる始末でした。

そのため、今後同じようなケースが発生したときにスムーズに対応できるようにするべく、備忘録として手順を記載します。

## どうすればいいか？

結論から言えば、VSCode（Cursorの場合はCursor）のworkspaceStorage内の当該ワークスペース（「Remember my choice for this task」を押下してしまったワークスペース）に対応するstate.vscdbを削除すれば解決します。

### 手順

#### 1．エディタを閉じる

まずはVSCode（Cursor）を閉じてください。
でないと、後で出てくるstate.vscdbを削除できません。

#### 2. workspaceStorageフォルダを開く

workspaceStorageのパスは以下の通りです。

Windowsの場合:
`%APPDATA%\Code\User\workspaceStorage`

Mac/Linuxの場合:
`~/.config/Code/User/workspaceStorage`

なお、Cursorを使っている方は、上記パスのCodeをCursorに置き換えてください。

Windowsの場合:
`%APPDATA%\Cursor\User\workspaceStorage`

Mac/Linuxの場合:
`~/.config/Cursor/User/workspaceStorage`

すると、英数字の羅列みたいなフォルダ一覧が出てくるはずです。

![Workspace storage folders](/images/how-to-reset-remember-my-choice-for-this-task/workspace-storage-folders.png)

このフォルダー1つが、1つのWorkspaceの各種履歴やキャッシュなどを管理してる、というイメージです。

#### 3. 更新履歴が最新のフォルダを開く

この英数字の羅列がどのWorkspaceに結びついているか確認する方法もあるようですが、手順が複雑です。
そこで、更新履歴が最新のもの＝最近開いたものと判断する方法をとります。

#### 4. state.vscdbを削除する

中にstate.vscdbというファイルがあるはずなので、削除してください。

#### 5. エディタを起動する

VSCode（Cursor）を再度起動し、「Remember my choice for this task」を押下してしまったワークスペースを開いてください。
再度ビルド設定エラーが発生した際、しっかりダイアログが表示されると思います。

## まとめ

まさかビルド設定エラー時のダイアログの「Remember my choice for this task」の設定がワークスペースのストレージに記録されているとは思いませんでした...。
非常にレアケースだと思いますが、同じような状況に陥って解決したい方は、是非参考にしてみてください。
