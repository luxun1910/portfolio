+++
title = '存在するはずのLinuxコマンドが見つからないときの対処法'
date = 2025-07-30T10:00:00+08:00
draft = false
+++

## 経緯

Windows11のWSL2でUbuntuを使っており、qemuをaptでインストールした。
しかしapt経由でインストールできるqemuはバージョンがかなり古かったので、直接ソースコードを落としてきてビルドすることにした。
既存のqemuはアンインストールし、ビルドした最新版のqemuをインストールした。

qemuがインストールされたか確認するため`qemu-system-x86_64 --version`を実行したところ、以下のエラーが出た。

```bash
-bash: /usr/bin/qemu-system-x86_64: No such file or directory
```

## 解決策に至るまでの思考

環境変数は問題なさそう。

```bash
echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin: ...
```

そもそもqemu-system-x86_64は本当にインストールされたのか？ということで以下のコマンドを実行してみる。

```bash
which qemu-system-x86_64
/usr/local/bin/qemu-system-x86_64
```

「/usr/local/bin/」...？
さっきのエラーメッセージは「/usr/bin/qemu-system-x86_64」を探しに行ってたので、コマンドの正しい場所が認識されていないらしい。

AIに聞いたところ、「/usr/bin/qemu-system-x86_64」→「/usr/local/bin/qemu-system-x86_64」へのシンボリックリンクを作成すれば解決するとのことで、確かにコマンドは動くようになった。
しかし、それが根本的な解決策とは到底思えなかった。

## 原因判明

色々調べていくうちに、新たな事実が判明した。
どうやら、bashでは一度コマンドを実行すると、ハッシュテーブルにコマンド実行ファイルのフルパスが記録されるらしい。
以降は実行ファイルを探しに行かずそのハッシュテーブルを見るようになるとのこと。

これが原因なのではないかと思い、ハッシュテーブルの内容を表示してみた。

```bash
hash
   1    /usr/bin/find
  21    /usr/bin/qemu-system-x86_64
  14    /usr/bin/ls
   3    /usr/bin/make
   2    /usr/bin/chown
  19    /usr/bin/sudo
```

ビンゴ！おそらくこのハッシュテーブルのせいで、コマンド実行時に「/usr/bin/qemu-system-x86_64」を探しに行っていたのだと思われる。

最初に旧バージョンのqemuをインストールしたときは、「/usr/bin/qemu-system-x86_64」にインストールされたのだろう。
旧バージョン使用中も`qemu-system-x86_64 --version`を何度か実行したので、それがハッシュテーブルに記録され、新バージョンが「/usr/local/bin/qemu-system-x86_64」にインストールされた後も「/usr/bin/qemu-system-x86_64」を探しに行っていたのだと思われる。

## 解決策

以下のコマンドを実行することで、qemu-system-x86_64が正しく実行できるようになった。

```bash
PATH="$PATH"
```

現在の環境変数を現在の環境変数で置き換えるという一見意味のないことをしているが、これによってハッシュテーブルの内容がリセットされる。
後から知ったが、`hash -r`を叩けばハッシュテーブルの内容を空にできるので、そちらでもいいだろう。
  
## 参考文献

[【Linux】PATHについて #Linux - Qiita](https://qiita.com/ryouya3948/items/8edbd5d744c83dd41141)
[Linuxコマンドはどうやって実行されている？環境変数PATHが設定される流れを調べてみた - Devplatform blog](https://blog.devplatform.techmatrix.jp/blog/env-var-path-bash-001/)
[コマンド hash bash のハッシュテーブル管理用コマンド。bash の内部コマンド。 - UNIX/Linuxの部屋 hashコマンドの使い方](http://x68000.q-e-d.net/~68user/unix/pickup?hash)
