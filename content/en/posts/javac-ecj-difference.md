+++
title = 'Code that compiles with javac but throws errors with ECJ'
date = 2025-04-13T17:54:28+08:00
draft = false
+++

## Background

I was editing a Java 8 + Spring Boot program in IntelliJ IDEA.
However, I wanted to try the trending AI editor Cursor, so I opened the same program in Cursor (VSCode).
Surprisingly, this caused a large number of errors, making compilation impossible.
After spending a full day investigating the cause, I'm posting this as a cautionary tale.

## The problematic code

In summary, the code was as follows:

```java
package com.example;

import java.util.List;

public class ServiceImpl<V> implements IService<V> {
    // Other members

    // This shows as a warning in IntelliJ but causes a compilation error in Cursor (VSCode)
    public List<?> voList(List list) {
        return null;
    }
}

interface IService<V> {
    List<V> voList(List<V> list);
}
```

As noted in the comment, the `public List<?> voList(List list)` part only triggers a warning in IntelliJ and builds fine, but it causes an error in Cursor and fails to build.
Normally, changing `List<?>` to either `List<V>` or just `List` would solve the issue, but this code was in a class file that couldn't be edited.
Therefore, I wanted to find a way to fix this through editor settings rather than code changes.

## The cause

The conclusion is that this phenomenon appears to be due to differences in compilers.
IntelliJ uses javac by default, while Cursor (VSCode) uses ECJ (the same compiler used by Eclipse).
The same code that javac treats as a warning, ECJ classifies as an error.
While it's possible to configure Cursor (VSCode) to use javac, it apparently requires JDK 23 or higher, which wasn't an option for this project running on Java 8 (JDK 1.8).

## What Did I End Up Doing

Since I want to use Cursor but can't compile from it, I settled on a strategy of editing in Cursor while using IntelliJ for execution.
It's inconvenient that I have to rebuild in IntelliJ before running because builds are triggered when editing files in Cursor, but there doesn't seem to be any other solution at present.

## References

- <https://github.com/redhat-developer/vscode-java/wiki/Javac%E2%80%90based-compilation-support>
