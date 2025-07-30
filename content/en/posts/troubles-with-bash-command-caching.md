+++
title = 'Troubles with Bash Command Caching'
date = 2025-07-30T10:00:00+08:00
draft = false
+++

## Background

I'm using Ubuntu on Windows 11 WSL2 and installed qemu via apt.
However, the qemu available through apt was quite old, so I decided to build the latest version from source code.
I uninstalled the existing qemu and installed the newly built latest version of qemu.

When I ran `qemu-system-x86_64 --version` to check if qemu was installed, I got the following error:

```bash
-bash: /usr/bin/qemu-system-x86_64: No such file or directory
```

## Thought Process Leading to the Solution

The environment variables seemed fine.

```bash
echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin: ...
```

Was qemu-system-x86_64 actually installed? I ran the following command to check:

```bash
which qemu-system-x86_64
/usr/local/bin/qemu-system-x86_64
```

"/usr/local/bin/"...?
The earlier error message was looking for "/usr/bin/qemu-system-x86_64", so it seems the correct location of the command wasn't being recognized.

When I asked AI, it suggested creating a symbolic link from "/usr/bin/qemu-system-x86_64" to "/usr/local/bin/qemu-system-x86_64", and indeed the command started working.
However, I didn't think this was a fundamental solution.

## Root Cause Identified

While investigating further, I discovered a new fact.
Apparently, in bash, once a command is executed, the full path of the command executable is recorded in a hash table.
From then on, bash refers to this hash table instead of searching for the executable.

I suspected this might be the cause and displayed the contents of the hash table:

```bash
hash
   1    /usr/bin/find
  21    /usr/bin/qemu-system-x86_64
  14    /usr/bin/ls
   3    /usr/bin/make
   2    /usr/bin/chown
  19    /usr/bin/sudo
```

Bingo! This hash table was probably causing bash to look for "/usr/bin/qemu-system-x86_64" during command execution.

When I first installed the old version of qemu, it was probably installed to "/usr/bin/qemu-system-x86_64".
Since I ran `qemu-system-x86_64 --version` several times while using the old version, it was recorded in the hash table, and even after the new version was installed to "/usr/local/bin/qemu-system-x86_64", bash was still looking for "/usr/bin/qemu-system-x86_64".

## Solution

By running the following command, qemu-system-x86_64 started working correctly:

```bash
PATH="$PATH"
```

This may seem meaningless as it replaces the current environment variable with itself, but this resets the command cache.
I later learned that running `hash -r` can empty the hash table contents, so that would work too.
  
## References

[【Linux】PATHについて #Linux - Qiita](https://qiita.com/ryouya3948/items/8edbd5d744c83dd41141)
[Linuxコマンドはどうやって実行されている？環境変数PATHが設定される流れを調べてみた - Devplatform blog](https://blog.devplatform.techmatrix.jp/blog/env-var-path-bash-001/)
[コマンド hash bash のハッシュテーブル管理用コマンド。bash の内部コマンド。 - UNIX/Linuxの部屋 hashコマンドの使い方](http://x68000.q-e-d.net/~68user/unix/pickup?hash)
