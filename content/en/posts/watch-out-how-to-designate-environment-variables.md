+++
title = 'Watch Out for How You Designate Environment Variables in the Windows Terminal'
date = 2026-02-15T08:36:55+08:00
draft = false
+++

## What Happened

I was building an application with Flutter.
I was using Firebase Authentication for login; it worked on the web, but as soon as I debugged on the Android app, I got the following error:

`PlatformException(sign_in_failed, com.google.android.gms.common.api.ApiException: 10:, null, null)`

I found out that to use Firebase Authentication in each app, you need to obtain the signing certificate SHA-1 for that app.

I looked up how to get the signing certificate and ended up on the official documentation.

<https://developers.google.com/android/guides/client-auth?hl=en>

The official documentation said that for debug applications on Windows, you can get it as follows:

```shell
keytool -list -v -alias androiddebugkey -keystore %USERPROFILE%\.android\debug.keystore
```

When I ran that in Cursor's terminal as written...

```shell
keytool error: java.lang.Exception: Keystore file was not found: %USERPROFILE%\.android\debug.keystore
java.lang.Exception: Keystore file was not found: %USERPROFILE%\.android\debug.keystore
        at java.base/sun.security.tools.keytool.Main.doCommands(Main.java:920)
        at java.base/sun.security.tools.keytool.Main.run(Main.java:419)
        at java.base/sun.security.tools.keytool.Main.main(Main.java:400)
```

???

When I typed `%USERPROFILE%\.android` in File Explorer, the debug.keystore was there.

??????????

## Cause and Solution

It turned out that the command above was written for Windows Command Prompt.
I was using PowerShell in Cursor's terminal, which is why it failed.

`USERPROFILE` is an environment variable. In Command Prompt you can reference it as `%USERPROFILE%`, but in PowerShell you need to use `$env:USERPROFILE` (see: <https://qiita.com/naka_toshinori/items/c9ea1efc89686ed3d018>).

In PowerShell, `%USERPROFILE%` is treated as a literal string, so it looked for a file literally named `%USERPROFILE%\.android\debug.keystore`, and of course that file does not exist.
When I ran the following in PowerShell, it worked:

```shell
keytool -list -v -alias androiddebugkey -keystore $env:USERPROFILE\.android\debug.keystore
```

In Command Prompt, the command from the official documentation works as written.

For apps and libraries where Mac/Linux development is dominant, Windows setup instructions often don't specify whether the syntax is for PowerShell or Command Prompt. I expect to run into this pattern again, so I want to keep this lesson in mind.
