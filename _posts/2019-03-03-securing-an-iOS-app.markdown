---
layout: default
title:  "Securing an iOS App"
date:   2019-03-03 15:00:00 -0300
categories: development
---
# Securing an iOS App

Developing an iOS app that meet security standards like PCI-DSS, HIPAA, FISMA among others is challenging. The main reason is that we need to understand the main vectors of attack to an app and sometimes that comes with experience and pentests.

In this article I try to summarize some lessons I learned over the course of several security reviews in making your iOS app more secure. 

This article focuses mostly on iOS 11 and 12 and the goal is how to leverage Apple's current security requirements to avoid unintended data leakage.

## Don't leave files behind

I strongly recommend that you purge any locally stored data upon user logout or finishing a session.

There are lots of temporary data stored in your sandbox environment that need to be wiped or an attacker gaining control of the device can recover important information.

Here some things worth cleaning.

### Network caches

I use the following Obj-C code to accomplish it:

```objc
+ (void)destroyNetworkCache
{
    NSString *caches = [NSSearchPathForDirectoriesInDomains(NSCachesDirectory, NSUserDomainMask, TRUE) objectAtIndex:0];
    NSString *appID = [[NSBundle mainBundle] infoDictionary][@"CFBundleIdentifier"];
    NSString *path = [NSString stringWithFormat:@"%@/%@", caches, appID];

    [[NSFileManager defaultManager] removeItemAtPath:path error:nil];
}
```

### The whole Sandbox

Using a helper method that navigate folders cleaning the content you can use the following code to wipe your Sandbox.

```objc
+ (void)emptySandbox
{
    NSArray *paths = NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES);
    NSString *documentsDirectory = [paths objectAtIndex:0];

    [self cleanFolder:documentsDirectory];
    NSString *tmpDirectory = [documentsDirectory stringByReplacingOccurrencesOfString:[documentsDirectory lastPathComponent] withString:@"tmp"];
    [self cleanFolder:tmpDirectory];
}
```

If you want some files to persist this removal you can preserve then instead of removing the whole `documentsDirectory`.

## Avoid screenshots of sensitive data

When the user closes an App by using the home button iOS takes a screenshot in order to create the closing app effect and to display the app for multitasking.

For most apps this is okay, but if you are developing an app that handles financial data, medical data or even classified documents you don't want screenshots of this floating around.

In order to achive this you can implement the method `applicationWillResignActive:` in you Application Delegate and hide the window content. Then you show the content you masked by implement `applicationDidBecomeActive:` and  `applicationWillEnterForeground:`

A sample implementation of hide/show screen content can be found below.

```objc
- (void)hideContent
{
    if (!self.hideContentWindow)
    {
        self.hideContentWindow = [[UIWindow alloc] initWithFrame:[UIScreen mainScreen].bounds];

        UIView *launchScreen = [[[NSBundle mainBundle] loadNibNamed:@"Launch Screen" owner:nil options:nil] objectAtIndex:0];
        launchScreen.autoresizingMask = UIViewAutoresizingFlexibleWidth | UIViewAutoresizingFlexibleHeight;
        launchScreen.frame = [UIScreen mainScreen].bounds;

        UIViewController *controller = [UIViewController new];
        [controller.view addSubview:launchScreen];
        self.hideContentWindow.rootViewController = controller;
    }
    [self.hideContentWindow makeKeyAndVisible];
}

- (void)unhideContent
{
    self.hideContentWindow.hidden = YES;
    self.hideContentWindow = nil;
}
```

## Use certificate pinning

Certificate Pinning is the process of associating a host with their expected X509 certificate or public key, according to [OWASP](https://www.owasp.org/index.php/Pinning_Cheat_Sheet).

The goal here is to avoid a Man in the middle kind of attack. If you don't use a certificate pinning and the certificate trust chain of your device got compromised then an attacker can install a Certification Authority in the device and decrypt SSL traffic using an SSL MITM proxy.

[TrustKit](https://github.com/datatheorem/TrustKit) is a very well maintened and secure library for iOS that will do all the heavy lifting on checking the network connections for a proper SSL certificate.

After extracting all the data you need from the certificate (here I strongly recomend reading the README on the TrustKit repository), implementing certificate pinning in your application is as simple as calling the method below:

```objc
+ (void)initTrustKit
{
    NSDictionary *trustKitConfig =
      @{
        kTSKSwizzleNetworkDelegates: @YES,
        kTSKPinnedDomains : @{
            @"api.mydomain.com" : @{
                kTSKPublicKeyAlgorithms : @[kTSKAlgorithmRsa2048,
                                            kTSKAlgorithmRsa4096,
                                            kTSKAlgorithmEcDsaSecp256r1],
                kTSKPublicKeyHashes : @[
                    @"yixTXUagjRlIlDRB6QOVVxBVqwPAxptoDeutKmJx4=",
                    @"0FBf3cRRadZJaMxoF19oC73VMavLxj/N7WBNbeNzkR8=",
                ],
                kTSKEnforcePinning : @YES,
            },
        }
    };

    [TrustKit initializeWithConfiguration:trustKitConfig];
}
```

## Encrypt your databases

On iOS there are two levels of protection that we can apply to persistent data on a database: Filesystem Encryption and Data Encryption.

On Filesystem Encryption, [Apple allow the developer](https://developer.apple.com/documentation/uikit/core_app/protecting_the_user_s_privacy/encrypting_your_app_s_files?language=objc) to select one of four kinds of security for each file you create:

- *No protection*. The file is always accessible.

- *Complete until first user authentication. (Default)* - The file is inaccessible until the first time the user unlocks the device. After the first unlocking of the device, the file remains accessible until the device shuts down or reboots.

- *Complete unless open.* You can open existing files only when the device is unlocked. If you have a file already open, you may continue to access that file even after the user locks the device. You can also create new files and access them while the device is locked or unlocked.

- *Complete*. The file is accessible only when the device is unlocked.

The *Complete* level is, of course, the more secure but it forces you to manage the locking and unlocking events by implementing the App delegate methods `applicationProtectedDataWillBecomeUnavailable:` and `applicationProtectedDataDidBecomeAvailable`.

My current recommendation is to leave the default file protection and implement full database encryption.

For Core Data that's not an ideal solution because you'll have to rely on a specific library that create a layer over Core Data NSIncrementalStorage. This library, [Encrypted Core Data](https://github.com/project-imas/encrypted-core-data), now lacks frequent updates and suffers from performance issues and limitations on numbers of parameters passed in a query.

In our main project we switched completely from Core Data to [Realm](https://realm.io/products/realm-database/) as a persistence layer for that reason.

Realm comes with support to AES-256 encryption that's easily added to the database with few lines of code:

```objc
    RLMRealmConfiguration *config = [RLMRealmConfiguration new];
    config.encryptionKey = key;
    Realm * realm = [RLMRealm realmWithConfiguration:config error:nil];
```

## Conclusion

There are a number of other details you need to care about when implementing a secure iOS application. I chose tho list some that had a more significant impact in making our app safer.

Below you can find more resources I recommend if you want to dive deeper in the subject.

- [Apple's iOS Security Guide](https://www.apple.com/business/site/docs/iOS_Security_Guide.pdf) it's like a version of the [Human Interface Guidelines](https://developer.apple.com/design/human-interface-guidelines/) but for security.

- [Data Theorem's blog](https://blog.datatheorem.com/blog) - This is the maker of TrustKit and they have a blog on mobile security
