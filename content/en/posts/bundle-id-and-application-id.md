+++
title = 'Why Flutter project_name should be compatible with both iOS and Android to avoid trouble later'
date = 2025-07-17T12:27:19+08:00
draft = false
+++

## Why I wrote this article

While introducing Firebase to a Flutter app, I executed `flutterfire configure` and included all platforms (iOS, Android, Web, etc.). However, it failed at the iOS stage.

```bash
i Firebase android app com.unanimousworks.nonsense_generator registered.
i Firebase ios app com.unanimousworks.nonsense_generator is not registered on Firebase project nonsensegenerator-d140d.    
⠦ Registering new Firebase ios app on Firebase project nonsensegenerator-d140d.
FirebaseCommandException: An error occured on the Firebase CLI when attempting to run a command.
COMMAND: firebase apps:create ios nonsense_generator (ios) --bundle-id=com.unanimousworks.nonsense_generator --json --project=nonsensegenerator-d140d
```

Looking at the error log, it showed the following output:

```log
[debug] [2025-07-16T05:11:47.788Z] >>> [apiv2][body] POST {Firebase API URL} {"displayName":"nonsense_generator (ios)","bundleId":"com.unanimousworks.nonsense_generator"}
[debug] [2025-07-16T05:11:48.587Z] <<< [apiv2][status] POST {Firebase API URL} 400
[debug] [2025-07-16T05:11:48.587Z] <<< [apiv2][body] POST {Firebase API URL} {"error":{"code":400,"message":"Request contains an invalid argument.","status":"INVALID_ARGUMENT"}}
[debug] [2025-07-16T05:11:48.588Z] Request to {Firebase API URL} had HTTP Error: 400, Request contains an invalid argument.
[debug] [2025-07-16T05:11:48.889Z] FirebaseError: Request to {Firebase API URL} had HTTP Error: 400, Request contains an invalid argument.
```

It says "Request contains an invalid argument."
The request body contains the following two parameters:

- `"displayName":"nonsense_generator (ios)"`
- `"bundleId":"com.unanimousworks.nonsense_generator"`

Therefore, one of these must be incorrect.
Since displayName didn't seem to be particularly problematic, I suspected the bundleId and investigated.

## iOS Bundle ID Definition

`A bundle ID string needs to be a uniform type identifier (UTI) that contains only alphanumeric characters (A-Z, a-z, 0-9), hyphens (-), and/or periods (.). The string should be in reverse-DNS format.`

Reference: <https://developer.apple.com/help/glossary/bundle-id/>

In this case, the bundleId was `com.unanimousworks.nonsense_generator`, which contains an underscore (_).
This seems to be why the error occurred.

I used [Change App Package Name for Flutter](https://pub.dev/packages/change_app_package_name) to fix only the iOS Bundle ID to comply with the requirements using the following command:
`dart run change_app_package_name:main com.new.package.name --ios`

## Android Application ID Definition

I also looked up the Android Application ID definition while I was at it.

```md
- It must have at least two segments (one or more dots).
- Each segment must start with a letter.
- All characters must be alphanumeric or an underscore [a-zA-Z0-9_].
```

Reference: <https://developer.android.com/build/configure-app-module#set-application-id>

## Conclusion

When creating a new Flutter project, running `flutter create project_name` sets both Bundle ID and Application ID to `com.example.project_name`.

Android Application ID allows underscores (_), but iOS Bundle ID does not.
Conversely, iOS Bundle ID allows hyphens (-), but Android Application ID does not.

Since most developers creating Flutter apps plan to publish on both iOS and Android, it's best to use only alphanumeric characters for Flutter project names.
