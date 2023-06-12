---
layout: post
title: Using UIViewController's viewIsAppearing method in Xcode 14 and earlier
keywords: xcode14, uikit, viewIsAppearing, wwdc2023, uiviewcontroller, xcode15, ios17
---

During WWDC 2023, Apple announced the introduction of a new method on `UIViewController` called [`viewIsAppearing(_:)`](https://developer.apple.com/documentation/uikit/uiviewcontroller/4195485-viewisappearing).

If you've ever spent way too much time trying to perfect appearance animations within your apps, this new method may just be the lifecycle callback that you had been looking for since it's called prior to the actual appearance on-screen but after receiving the initial layout and traits.

What is even better is that it was announced that this method has been back-deployed all the way down to iOS 13, which is great, but if like me you want to use it in your apps today, you'll find that unfortunately you still need to wait for Xcode 15 and the iOS 17...

Or do you?

## What does being back-deployed actually mean here?

While Apple mentioned that this method _back-deploys all the way to iOS 13_, this is a little bit confusing. If you follow along with Swift Evolution proposals, you may well have understood this statement to have meant that the API was built using the new `@backDeployed` attribute that was proposed in [SE-0376](https://github.com/apple/swift-evolution/blob/main/proposals/0376-function-back-deployment.md) and implemented in Swift 5.8. This however is not the case here.

In this instance, the `viewIsAppearing(_:)` method has existed in UIKit since the iOS 13 SDK first shipped but the method was not made visible in the public headers that our code can see.

In the iOS 17 SDK, Apple have finally declared this method in the public SDK headers meaning that our code can now reference the previously private implementation that has been shipping since iOS 13. In fact, you can look for yourself in class dumps from older versions of the iOS SDK ([example](https://developer.limneos.net/?ios=13.1.3&framework=UIKitCore.framework&header=UIViewController.h)).

## Using the method in Xcode 14 or earlier

So if the method already existed in the iOS 13, 14, 15 and 16 SDKs, you might wonder what is stopping you from using it? Well it turns out that there is not a lot thanks to the fact that this portion of UIKit is still written in Objective-C!

In your project, add a new file called **UIViewController+UpcomingLifecycleMethods.h**:

```objc
#import "Availability.h"

#if defined(__IPHONE_17_0)
#warning "UIViewController+UpcomingLifecycleMethods.h is redundant when compiling with the iOS 17 SDK"
#else

@import UIKit;

@interface UIViewController (UpcomingLifecycleMethods)

/// Called when the view is becoming visible at the beginning of the appearance transition,
/// after it has been added to the hierarchy and been laid out by its superview. This method
/// is very similar to -viewWillAppear: and is always called shortly afterwards (so changes
/// made in either callback will be visible to the user at the same time), but unlike
/// -viewWillAppear:, at the time when -viewIsAppearing: is called all of the following are
/// valid for the view controller and its own view:
///    - View controller and view's trait collection
///    - View's superview chain and window
///    - View's geometry (e.g. frame/bounds, safe area insets, layout margins)
/// Choose this method instead of -viewWillAppear: by default, as it is a direct replacement
/// that provides equivalent or superior behavior in nearly all cases.
///
/// - SeeAlso: https://developer.apple.com/documentation/uikit/uiviewcontroller/4195485-viewisappearing
- (void)viewIsAppearing:(BOOL)animated API_AVAILABLE(ios(13.0), tvos(13.0)) API_UNAVAILABLE(watchos);

@end

#endif
```

The next step depends on your current project setup:

- **If you have an Objective-C project**, you can go ahead and import `UIViewController+UpcomingLifecycleMethods.h` in any view controller that you need to access this method within and if you have a Swift project, you can expose this via the Bridging Header.
- **If you have a Swift project and don't currently use a Bridging Header**, open the Build Settings for your target and set _Objective-C Bridging Header_ (`SWIFT_OBJC_BRIDGING_HEADER`) to `$(SRCROOT)/Path/To/UIViewController+UpcomingLifecycleMethods.h`.
- **If you have a Swift project and are already using a Bridging Header for other reasons**, import `UIViewController+UpcomingLifecycleMethods.h` within your existing Bridging Header.

In the header, we defined a category (extension in Swift) with the private (but soon-to-be-public) method signature. We don't provide the method implementation because it already exists so this alone is enough to expose the method to the rest of our project.

In your `UIViewController` subclasses, you can now go ahead and override the method just like you can when using Xcode 15:

```swift
override func viewIsAppearing(_ animated: Bool) {
    super.viewIsAppearing(animated)
    prepareForAppearance()
}
```

Because the header file is checking for the `__IPHONE_17_0` definition, which is only available as part of Xcode 15 and the iOS 17 SDK, this header becomes redundant once you start using Xcode 15. I've used `#warning` to trigger a custom warning message that can serve as a helpful reminder to come back in September and clean up.

> **Warning**: While you can do this with many other private methods, remember that private API wasn't necessarily designed to be consumed by other developers and its behavior might well change (or be removed entirely) in future releases, which would likely break your app.
>
> In this instance, we don't have these same concerns because we know that Apple is making this API public moving forward. But you should still remember that this API is technically _private_ today meaning that there is still a small chance that it might be rejected during App Review. I haven't yet verified myself that apps currently referencing this API won't be rejected, so submit for review at your own discretion.





