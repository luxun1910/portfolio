+++
title = 'javac不报错但ECJ报错的情况'
date = 2025-04-13T17:54:28+08:00
draft = false
+++

## 写这篇文章的缘由

我正在使用IntelliJ IDEA编辑一个Java 8 + Spring Boot程序。
然而，我想尝试流行的AI编辑器Cursor，所以我在Cursor（VSCode）中打开了相同的程序。
出乎意料的是，这导致了大量错误，使编译变得不可能。
经过一整天调查原因后，我写下这篇文章作为警示。

## 出现错误的代码

简而言之，代码如下：

```java
package com.example;

import java.util.List;

public class ServiceImpl<V> implements IService<V> {
    // 其他成员

    // 在IntelliJ中只显示警告，但在Cursor（VSCode）中导致编译错误的部分
    public List<?> voList(List list) {
        return null;
    }
}

interface IService<V> {
    List<V> voList(List<V> list);
}
```

如注释所述，`public List<?> voList(List list)`这部分在IntelliJ中只会触发警告并且可以正常构建，但在Cursor中会导致错误且无法构建。
通常，将`List<?>`更改为`List<V>`或仅为`List`就可以解决这个问题，但这段代码位于无法编辑的.class文件中。
因此，我想找到一种通过编辑器设置而非修改代码来解决这个问题的方法。

## 原因

结论是，这种现象似乎是由于编译器差异造成的。
IntelliJ默认使用javac，而Cursor（VSCode）使用ECJ（与Eclipse相同的编译器）。
javac将同样的代码视为警告，而ECJ将其归类为错误。
虽然可以将Cursor（VSCode）配置为使用javac，但据说需要JDK 23或更高版本，这对于运行在Java 8（JDK 1.8）上的当前项目来说不是一个选项。

## 最终如何解决

由于想要使用Cursor，但无法从Cursor进行编译，因此我决定采用在Cursor中进行编辑，而在需要运行时使用IntelliJ的方案。
在Cursor编辑文件时会触发构建过程，所以必须在IntelliJ中重新构建后才能运行，这点比较麻烦，但目前看来似乎没有其他办法。

## 参考资料

- <https://github.com/redhat-developer/vscode-java/wiki/Javac%E2%80%90based-compilation-support>
