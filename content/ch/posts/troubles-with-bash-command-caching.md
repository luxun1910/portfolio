+++
title = 'bash命令缓存问题'
date = 2025-07-30T10:00:00+08:00
draft = false
+++

## 背景

我在Windows 11的WSL2上使用Ubuntu，通过apt安装了qemu。
但是通过apt安装的qemu版本相当旧，所以我决定直接下载源代码进行构建。
我卸载了现有的qemu，安装了构建的最新版本qemu。

为了确认qemu是否已安装，我执行了`qemu-system-x86_64 --version`，结果出现了以下错误：

```bash
-bash: /usr/bin/qemu-system-x86_64: No such file or directory
```

## 解决过程中的思考

环境变量看起来没有问题。

```bash
echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin: ...
```

qemu-system-x86_64真的安装了吗？为此我执行了以下命令。

```bash
which qemu-system-x86_64
/usr/local/bin/qemu-system-x86_64
```

“/usr/local/bin/”...？
刚才的错误信息是在寻找“/usr/bin/qemu-system-x86_64”，所以似乎没有识别到命令的正确位置。

询问AI后，得到的建议是创建从“/usr/bin/qemu-system-x86_64”到“/usr/local/bin/qemu-system-x86_64”的符号链接，确实命令能够运行了。
但是，我认为这绝不是根本性的解决方案。

## 原因查明

在进一步调查的过程中，发现了新的事实。
原来，在bash中一旦执行了命令，命令执行文件的完整路径会被记录在哈希表中。
此后不再搜索执行文件，而是查看该哈希表。

我认为这可能是原因，于是显示了哈希表的内容。

```bash
hash
   1    /usr/bin/find
  21    /usr/bin/qemu-system-x86_64
  14    /usr/bin/ls
   3    /usr/bin/make
   2    /usr/bin/chown
  19    /usr/bin/sudo
```

答案找到了！可能是由于这个哈希表的缘故，在执行命令时会去寻找“/usr/bin/qemu-system-x86_64”。

最初安装旧版本qemu时，应该是安装到了“/usr/bin/qemu-system-x86_64”。
在使用旧版本期间也多次执行了`qemu-system-x86_64 --version`，所以被记录到了哈希表中，即使新版本安装到了“/usr/local/bin/qemu-system-x86_64”，还是会去寻找“/usr/bin/qemu-system-x86_64”。

## 解决方案

通过执行以下命令，qemu-system-x86_64能够正确执行了。

```bash
PATH="$PATH"
```

虽然这是用当前环境变量替换当前环境变量这种看似无意义的操作，但这样可以重置哈希表的内容。
后来得知，执行`hash -r`可以清空哈希表内容，所以那样也可以。
  
## 参考文献

[【Linux】PATHについて #Linux - Qiita](https://qiita.com/ryouya3948/items/8edbd5d744c83dd41141)
[Linuxコマンドはどうやって実行されている？環境変数PATHが設定される流れを調べてみた - Devplatform blog](https://blog.devplatform.techmatrix.jp/blog/env-var-path-bash-001/)
[コマンド hash bash のハッシュテーブル管理用コマンド。bash の内部コマンド。 - UNIX/Linuxの部屋 hashコマンドの使い方](http://x68000.q-e-d.net/~68user/unix/pickup?hash) 