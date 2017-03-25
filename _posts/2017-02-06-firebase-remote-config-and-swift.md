---
layout: post
title: Firebase Remote Config and Swift
keywords: firebase, firebase remote config, swift, iOS, watchOS, macOS, tvOS
---

So you've seen Firebase Remote Config and decided that it would go great in your Swift project? You're right, it will but if like me you've noticed that the design of the SDK doesn't play very well with all of your other beautiful looking strictly typed Swift code then this post is for you.

_**Note:** I don't cover setting up Firebase in this post, just using `FIRRemoteConfig` in a configured project. Check out [one][1] [of][2] [these][3] guides for help with the setup part._

Lets take this basic example:

``` swift
let count = FIRRemoteConfig.remoteConfig()["maximum_item_count"].numberValue?.intValue ?? 10
```

There are a few problems here:

* Damn it doesn't look great.
* `numberValue` returns an optional.
* The key/value pattern means that you can't guarantee that you've not made a typo or updated the key name somewhere else.
* Because of the optional, you either have to force unwrap or have a fallback.
* Carelessly force unwrapping is never a good thing and defeats the object of Swift's type safety.
* Having a fallback defeats the object of the nice default values that you specify upon initialising Remote Config.
* Littering all your classes with `import FirebaseRemoteConfig` will probably be a pain to undo once you decide to move away from Firebase.

It would be a lot nicer if we could do something like this instead:

``` swift
let count = Config.shared.maxItemCount
```

------

# Config.swift

The interface for my `Config` class is pretty simple:

``` swift
import Foundation
import FirebaseRemoteConfig

final class Config {

    /// The shared instance of config to use
    static let shared: Config = Config()

    /// The maximum number of items that are allowed in this mystery app
    let maxItemCount: Int

    /// The initialiser is private as intended use is via the `shared` static property.
    private init() {

        ...
    }
}
```

The idea is simple: Initialise all the properties on the shared instance after performing the initial fetch and then trigger another fetch after so that we can be ready to load any changes **the next time the app launches**.  

This is intentional to ensure that the values in `Config` are all fetched from a consistent data source (i.e to avoid accidentally reading one default value before calling `activateFetched` and then another remote value after the fetch completed).

As a result, the initialiser looks like this:


``` swift
// 1. Configure for dev mode if we need it, otherwise a 1 hour expiration duration
let remoteConfig = FIRRemoteConfig.remoteConfig()
#if DEBUG
    let expirationDuration: TimeInterval = 0
    remoteConfig.configSettings = FIRRemoteConfigSettings(developerModeEnabled: true)!
#else
    let expirationDuration: TimeInterval = 3600
#endif

// 2. Set our default values and keys
remoteConfig.setDefaults([
    "maximum_item_count": 42 as NSNumber
])

// 3. Activate any fetched values before we read anything back
remoteConfig.activateFetched()

// 4. Now set the properties on config based on what we have currently
self.maxItemCount = remoteConfig["maximum_item_count"].numberValue!.intValue

// 5. Perform the next fetch so that it's ready when we re-launch
remoteConfig.fetch(withExpirationDuration: expirationDuration) { status, _ in
    print("[Config] Fetch completed with status:", status, "(\(status.rawValue))")
}
```

Here is a breakdown of what we are doing:

1. If the app is running in debug mode, I enable dev mode and disable the `expirationDuration` so that the config refreshes each time. This is very handy during development but will get you throttled server side if you release something like that to production.
2. Set the default keys and values. I've opted to do this in code and not by using a plist so that I can have visibility of all the keys and values later on when I fetch them.
3. Activate any fetched parameters from the previous launch before we attempt to read them.
4. Read the fetched or default parameters back and set them as instance variables.
5. Perform a fetch asynchronously to get any changes that we can then activate the next time we launch the app.

There are still a few non-swifty looking bits here because I'm not using any form of constants to define the duplicate usage of `maximum_item_count` and I'm also force unwrapping the value however it does come with the following upsides:

* Only requires a single unit test to ensure that any of the force unwrapping isn't causing a crash.
* All the non-swifty looking code is isolated in a single file instead of across my entire project.
* I could easily update the `Config` class in the future to completely remove the dependancy of Firebase from my project if I wanted to.
* The rest of my code looks fabulous (kinda).

The complete class can be found [here][4] if you wish to grab a copy. Enjoy!

------

### Note

Due to the nature of Swift, the static `shared` property won't be initialised until you try to access it. This means that it might be useful doing something like the following in your AppDelegate if you want to ensure that the next fetch is performed as soon as possible:

``` swift
func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplicationLaunchOptionsKey: Any]?) -> Bool {

    FIRApp.configure()
    _ = Config.shared

    return true
}
```

I'm sure that could be done in a nicer way but I'll let you figure that out :)



[1]: https://www.raywenderlich.com/143712/firebase-remote-config-tutorial-for-ios
[2]: https://firebase.google.com/docs/remote-config/use-config-ios
[3]: https://www.youtube.com/watch?v=zdVc8aZZT-I
[4]: https://gist.github.com/liamnichols/4f1122cef22d3ddafc8d0b87f034914c
