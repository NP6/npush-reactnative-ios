# NPush SDK for IOS 
NP6 IOS SDK for Push Notification - The game changing libraries for building Blazingly fast 🚀 Highly interactives notification user experiences. Made with ❤️ 

## Introduction 
This library is a part of NP6 Push Notifications service, it allow you to interact with your users via Push Notifications sended via NP6 CM. 

## Table of content
1.	[Prerequisites](https://github.com/NP6/npush-ios#prerequisites)
2.	[Installation](https://github.com/NP6/npush-ios#installation)
3.	[Troubleshooting]()


## Prerequisites
Whe are going to review all the steps needed to be done before installing NPush SDK.


### Apple Push Notifications certificates

Before continuing, you need to add remote push notification permissions for your application. If you haven't done this step before please follow this 
[tutorial]().


### Add AppDelegate 

Add the following lines of code to your project. If your already have an application delegate, skip this part.

#### Swift 
```swift 
class MyAppDelegate: NSObject, UIApplicationDelegate, UNUserNotificationCenterDelegate {    
    func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey : Any]? = nil) -> Bool {
        UNUserNotificationCenter.current().delegate = self
        return true;
    }
}

@main
struct demoApp: App {
    
    @UIApplicationDelegateAdaptor(MyApplicationDelegate.self) var appDelegate
    
    ...
}
```

#### Objective-c 

```objective-c 
@interface AppDelegate () <UNUserNotificationCenterDelegate>

@end

@implementation AppDelegate


- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
    return YES;
}


@end
```


### add dependency 

Right click on project tree -> Add Packages -> tap **https://github.com/NP6/npush-ios/** in search bar
and get latest swift package dependency



## Installation 


### implement specific AppDelegate methods

In your application delegate add following lines of code :

#### Swift

Create and set your configuration 

```swift 
class MyAppDelegate: NSObject, UIApplicationDelegate, UNUserNotificationCenterDelegate {
    
    func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey : Any]? = nil) -> Bool {
      NPush.requestNotificationAuthorization(application)
      
      let config = Config("<identity>", application: UUID(uuidString: "<application-id>") ?? throw ...)
      NPush.instance.InitWithConfig(config: config)

      return true;
    }
    
    ...
}

```

Handle delegate Notification Center  

```swift 
class MyAppDelegate: NSObject, UIApplicationDelegate, UNUserNotificationCenterDelegate {
    
    ...
        func application(_ application: UIApplication, didReceiveRemoteNotification userInfo: [AnyHashable : Any], fetchCompletionHandler completionHandler: @escaping (UIBackgroundFetchResult) -> Void) {
        defer {
            completionHandler(.newData)
        }
        NPush.instance.willPresent(userInfo)
    }
    
    func userNotificationCenter(_ center: UNUserNotificationCenter, didReceive response: UNNotificationResponse, withCompletionHandler completionHandler: @escaping () -> Void) {
        defer {
            completionHandler()
        }
        NPush.instance.didReceive(response)
    }
    
    func userNotificationCenter(_ center: UNUserNotificationCenter, willPresent notification: UNNotification, withCompletionHandler completionHandler: @escaping (UNNotificationPresentationOptions) -> Void) {
        defer {
            completionHandler([.badge, .alert])
        }
        NPush.instance.willPresent(notification.request.content.userInfo)
    }
    
    func application(_ application: UIApplication, didRegisterForRemoteNotificationsWithDeviceToken deviceToken: Data) {
        NPush.instance.SetToken(deviceToken: deviceToken)

    }
    
    ...
}

```

```objective-c

@import npush; // Add sdk dependencie 

@interface AppDelegate () <UNUserNotificationCenterDelegate>


@end


@implementation AppDelegate


- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {

    // Request notifications authorizations 
    [NPush requestNotificationAuthorization:application];

    [[UIApplication sharedApplication] registerForRemoteNotifications];


    NSUUID *uuid = [[NSUUID UUID] initWithUUIDString:@"<application-id>"];

    Config* config = [[Config alloc] init:@"<identity>" application: uuid];
    
    NPush *npush = [NPush instance];
    
    [npush InitWithConfigWithConfig:config];
    return YES;
}

- (void)application:(UIApplication*)app didRegisterForRemoteNotificationsWithDeviceToken:(NSData*)devToken
{
    [NPush SetTokenWithDeviceToken:devToken];
}

- (void)application:(UIApplication *)application didReceiveRemoteNotification:(NSDictionary *)userInfo fetchCompletionHandler:(void (^)(UIBackgroundFetchResult))completionHandler
{
    [NPush willPresent:userInfo];
    completionHandler(UIBackgroundFetchResultNewData);
}

- (void)userNotificationCenter:(UNUserNotificationCenter *)center willPresentNotification:(UNNotification *)notification withCompletionHandler:(void (^)(UNNotificationPresentationOptions))completionHandler
{
    [NPush willPresent:notification.request.content.userInfo];
    completionHandler(UNAuthorizationOptionAlert | UNAuthorizationOptionBadge);
}

- (void)userNotificationCenter:(UNUserNotificationCenter *)center didReceiveNotificationResponse:(UNNotificationResponse *)response withCompletionHandler:(void (^)(void))completionHandler
{
    [NPush didReceive:response];
    completionHandler();
}

@end
```


### Attach contact to device subscription 

Suppose we have an application with a login & register form and we want to attach the current device subscription to the logged user.
We could only identify the users by hash, id or unicity criteria. 

**Note : All of these identifiers are strongly linked to the NP6 CM platform. Please be sure to have one of this 3 identifiers in your user representation before continue. **

### Create react package  
Declare a new ReactPackage by creating a new file implementation file called **NPushModule.h**

```objective-c
#import <React/RCTBridgeModule.h>
@interface RCTNPushModule : NSObject <RCTBridgeModule>
@end
```

let’s start implementing the native module. Create the corresponding implementation file, RCTNPushModule.m, in the same folder and include the following content:

```objective-c
#import <Foundation/Foundation.h>
#import "RCTNPushModule.h"

@implementation RCTNPushModule

// To export a module named RCTNPushModule
RCT_EXPORT_MODULE(RCTNPushModule);

@end
```

The native module can then be accessed in JS like this:

```objective-c
const {NPushModule} = ReactNative.NativeModules;
```

### Implement contact methods   

Use one of these functions depending on the type of credential you are using.

Example attaching device subscription by hash

```objective-c
RCT_EXPORT_METHOD(setContactByHash:(NSString *)hash)
{
    [npush SetContact :ContactTypeUnicityRepresentation value:@<hash>];
}
```

 Example attaching device subscription by unicity
```objective-c
RCT_EXPORT_METHOD(setContactByUnicity:(NSString *)unicity)
{
    [npush SetContact :ContactTypeUnicityRepresentation value:@<unicity>];
}
```


 Example attaching device subscription by id
```objective-c
RCT_EXPORT_METHOD(setContactById:(NSString *)id)
{
    [npush SetContact :ContactTypeIdRepresentation value:@<id>];
}
```

The last step is calling one of previous declarated methods in react native as follow :

### Example using native module attaching device subscription by id

``` javascript
// Example using native module attaching device subscription by id 
const {NPushModule} = ReactNative.NativeModules;
...  
 NPushModule.setContactById('000T39KL');
...  
```

If everything is done. You will see the following lines in your application log :

```
I/np6-messaging: Subscription created successfully
```
