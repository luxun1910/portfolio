+++
title = 'javacだとエラーにならないが、ECJだとエラーになることがあるらしい'
date = 2025-04-13T17:54:28+08:00
draft = false
+++

## この記事を書くに至った経緯

Java8 + Spring Bootで書かれたプログラムを、IntelliJ IDEAで開いて編集していた。
しかし、話題のAIエディタCursorを使用したくて、同じプログラムをCursor(VSCode)で開いてみた。
すると、なぜか大量のエラーが発生し、コンパイルできなくなった。
原因を探るのに1日溶かしてしまったので、戒めとして投稿する。

## エラーが起こったコード

要約すると、以下のようなコードである。

```java
package com.example;

import java.util.List;

public class ServiceImpl<V> implements IService<V> {
    // 他のメンバ

    // IntelliJでは警告になるが、Cursor(VSCode)だとコンパイルエラーになる個所
    public List<?> voList(List list) {
        return null;
    }
}

interface IService<V> {
    List<V> voList(List<V> list);
}
```

上記コメントの通り、`public List<?> voList(List list)`の部分で、IntelliJだと警告が出るだけで、問題なくビルドできるのだが、Cursorだとエラーになり、ビルドできない。
本来、上記コードの`List<?>`を`List<V>`か`List`とすれば問題ないのだが、上記コードはclassファイルとなっており、編集することができなかった。
そのため、コードを編集することなく、エディタ側の何かしらの設定をいじって対応できないかと考えた。

## 原因

結論から言うと、この現象はどうやらコンパイラーの違いに起因するらしい。
IntelliJでは標準でjacacが使われているのに対し、Cursor(VSCode)ではEclipseと同じECJが使用される。
javacだと上記コードは警告で済むのに対し、ECJではエラーと判定される、ということらしい。
Cursor(VSCode)でもjavacを使えるようにできるが、JDK23以上でないといけないらしく、Java8（JDK1.8）の今回のプロジェクトではCursor(VSCode)でjavacを使うことはできないようだ。

## 結局どうしたか

Cursorは使用したいが、Cursorからだとコンパイルはできないということなので、Cursorで編集を行いつつ、実行するときはIntelliJを使う、という方針に落ち着いた。
Cursorでファイルを編集するタイミングでビルドが走ってしまうので、IntelliJでリビルドをしてからでないと実行できないというのが面倒だが、現状だと他にどうしようもなさそうだ。

## 参考文献

- <https://github.com/redhat-developer/vscode-java/wiki/Javac%E2%80%90based-compilation-support>
