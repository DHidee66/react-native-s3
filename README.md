# React Native AWS S3

[![NPM version](http://img.shields.io/npm/v/react-native-s3.svg?style=flat)](https://www.npmjs.com/package/react-native-s3)
[![Build Status](https://travis-ci.org/mybigday/react-native-s3.svg)](https://travis-ci.org/mybigday/react-native-s3)
[![Dependency Status](https://david-dm.org/mybigday/react-native-s3.svg)](https://david-dm.org/mybigday/react-native-s3)
[![devDependency Status](https://david-dm.org/mybigday/react-native-s3/dev-status.svg)](https://david-dm.org/mybigday/react-native-s3#info=devDependencies)

A React Native wrapper for AWS [iOS](https://github.com/aws/aws-sdk-ios)/[Android](https://github.com/aws/aws-sdk-android) S3 SDK.

We currently implements `TransferUtility`, see [iOS](http://docs.aws.amazon.com/mobile/sdkforios/developerguide/s3transferutility.html)/[Android](http://docs.aws.amazon.com/mobile/sdkforandroid/developerguide/getting-started-store-retrieve-s3-transferutility.html) docs for more information.

## Installation

```bash
$ npm install react-native-s3 --save
```

## Setup

#### iOS

In XCode, in the project navigator:

* Right click `Libraries` ➜ `Add Files to [your project's name]`, Add `node_modules/react-native-s3/ios/RNS3.xcodeproj`.
* Add `libRNS3.a` to your project's `Build Phases` ➜ `Link Binary With Libraries`
* Add `$(SRCROOT)/../node_modules/react-native-s3/ios/Frameworks` to your project's `Build Settings` ➜ `Framework Search Paths`
* Add `node_modules/react-native-s3/ios/Frameworks/*.framework`, `libsqlite3.tbd`, `libz.tbd` to your project's `Build Phases` ➜ `Link Binary With Libraries`
* Edit `AppDelegate.m` of your project

```objective-c
#import "RNS3TransferUtility.h"

......

- (void)application:(UIApplication *)application handleEventsForBackgroundURLSession:(NSString *)identifier completionHandler:(void (^)())completionHandler {
  [RNS3TransferUtility interceptApplication:application
        handleEventsForBackgroundURLSession:identifier
                          completionHandler:completionHandler]
}
```

* __*[Optional]*__ you can set the credentials in `AppDelegate.m`

```objective-c
#import "RNS3TransferUtility.h"

......

- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions
{
  [[RNS3TransferUtility nativeCredentialsOptions] setObject:@"eu-west-1" forKey:@"region"];
  [[RNS3TransferUtility nativeCredentialsOptions] setObject:[NSNumber numberWithInt:[RNS3TransferUtility credentialType:@"BASIC"]] forKey:@"type"];
  [[RNS3TransferUtility nativeCredentialsOptions] setObject:@"your_access_key_here" forKey:@"access_key"];
  [[RNS3TransferUtility nativeCredentialsOptions] setObject:@"your_secret_key_here" forKey:@"secret_key"];
  ......
}
```

#### Android

* Edit `android/settings.gradle` of your project:

```gradle
...
include ':react-native-s3'
project(':react-native-s3').projectDir = new File(settingsDir, '../node_modules/react-native-s3/android')
```

* Edit `android/app/build.gradle` of your project:

```gradle
...
dependencies {
    ...
    compile project(':react-native-s3')
}
```

* Add package to `MainActivity`

```java
......

import com.mybigday.rn.*;   // import

public class MainActivity extends ReactActivity {

    ......

    @Override
    protected List<ReactPackage> getPackages() {
        return Arrays.<ReactPackage>asList(
            new MainReactPackage(),
            new RNS3Package()   // add package
        );
    }
}
```

You can use [rnpm](https://github.com/rnpm/rnpm) instead of above steps.

* Edit `android/app/src/main/AndroidManifest.xml` of your project:

```xml
<service
  android:name="com.amazonaws.mobileconnectors.s3.transferutility.TransferService"
  android:enabled="true" />
```

* __*[Optional]*__ you can set the credentials in `MainActivity`:

```java
@Override
public void onCreate(Bundle savedInstanceState) {
  super.onCreate(savedInstanceState);

  RNS3TransferUtility.nativeCredentialsOptions.put("region", "eu-west-1");
  RNS3TransferUtility.nativeCredentialsOptions.put("type", RNS3TransferUtility.CredentialType.BASIC);
  RNS3TransferUtility.nativeCredentialsOptions.put("access_key", "your_access_key_here");
  RNS3TransferUtility.nativeCredentialsOptions.put("secret_key", "your_secret_key_here");
}

```

## Usage

```js
import { transferUtility } from 'react-native-s3';
```

## API

#### `transferUtility.setupWithNative()`

Return: Promise - will resolve arguments:
* Boolean - `true` or `false` depending on the setup successful.

#### `transferUtility.setupWithBasic(options)`

* `options` Object
  * `region` String - a S3 Region (default: eu-west-1)
  * `access_key` String - the AWS access key ID
  * `secret_key` String - the AWS secret access key
  * `session_token` String - (optional) __(Android)__

Return: Promise - will resolve arguments:
* Boolean - `true` or `false` depending on the setup successful.

#### `transferUtility.setupWithCognito(options)`

* `options` Object
  * `region` String - a S3 Region (default: eu-west-1)
  * `identity_pool_id` String - the Amazon Cogntio identity pool
  * `caching` Boolean - use `CognitoCachingCredentialsProvider` instead of `CognitoCredentialsProvider` __(Android)__

See AWS CognitoCredentialsProvider ([iOS](http://docs.aws.amazon.com/AWSiOSSDK/latest/Classes/AWSCognitoCredentialsProvider.html)/[Android](http://docs.aws.amazon.com/AWSAndroidSDK/latest/javadoc/com/amazonaws/auth/CognitoCredentialsProvider.html)) for more information.

Return: Promise - will resolve arguments:
* Boolean - `true` or `false` depending on the setup successful.

#### `transferUtility.upload(options)`

Create an upload task.

* `options` Object
  * `bucket` String - a S3 bucket name
  * `key` String - the object key in the bucket
  * `file` String - the file path to upload
  * `meta` Object
    * `contentType` String - the file content-type
    * `contentMD5` String - the file md5 hash (optional)

Return: Promise - will resolve, see following arguments:
* Object - a [Task](#the-task-object-structure) object

or reject.

#### `transferUtility.download(options)`

Create a download task.

* `options` Object
  * `bucket` String - a S3 bucket name
  * `key` String - the object key/destination in the bucket
  * `file` String - donwload save file path

Return: Promise - will resolve, see following arguments:
* Object - a [Task](#the-task-object-structure) object

or reject.

#### `transferUtility.pause(id)`

* `id` Number - a Task id

#### `transferUtility.resume(id)`

* `id` Number - a Task id

#### `transferUtility.cancel(id)`

* `id` Number - a Task id

#### `transferUtility.deleteRecord(id)` __(Android)__

* `id` Number - a Task id

Return: Promise - will resolve, see following arguments:
* Boolean - `true` or `false` depending on the delete task record successful.

#### `transferUtility.getTask(id)`

Gets a Task object with the given id.

* `id` Number - a Task id

Return: Promise - will resolve, see following arguments:
* Object - a [Task](#the-task-object-structure) object

#### `transferUtility.getTasks(type, idAsKey)`

Gets a Task object list with the type.

* `type` String - enum: `upload`, `download`
* `idAsKey` Boolean - true: return Object with id as key, false: return Array

Return: Promise - will resolve, see following arguments:
* Array - a [Task](#the-task-object-structure) object list

#### `transferUtility.subscribe(id, eventHandler)`

Subscribe the task changes with the given id.

* `id` Number - a Task id
* `eventHandler` Function - arguments:
  * `task` Object - a [Task](#the-task-object-structure) object

#### `transferUtility.unsubscribe(id)`

Unsubscribe the task changes with the given id.

* `id` Number - a Task id

## The `Task` object structure

```js
{
  id: Number,
  state: String, // task state
  // progress of task: bytes / totalBytes
  bytes: Number,
  totalBytes: Number,
  // iOS only part, waiting https://github.com/aws/aws-sdk-android/pull/105
  bucket: String,
  key: String,
}
```

It will not be immediately refresh, you must `subscribe` or call `getTask(id)` to replace it.

## The `Task` states

* `waiting`
* `in_progress`
* `pause`
* `canceled`
* `completed`
* `failed`

## The `nativeCredentialsOptions` type

* `BASIC`
* `COGNITO`

## Roadmap

#### iOS

- [x] TransferUtility
- [ ] TransferManager
- [ ] Bucket Control
- CredentialsProvider
  - [ ] STS

#### Android 

- [x] TransferUtility
- [ ] TransferManager (Deprecated)
- [ ] Bucket Control
- CredentialsProvider
  - [ ] STS

## License

[MIT](LICENSE.md)
