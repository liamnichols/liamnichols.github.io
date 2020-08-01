---
layout: post
title: Building Swift Packages as a Universal Binary
keywords: xcode12, spm, apple silicon, universal binary, swift package, swift build, arm64, intel
---

So following Apple's announcement during WWDC 2020 that they'll be transitioning the Mac away from Intel processors to Apple Silicon it's now time for everybody to get their software ready.

The transition this time can be considered somewhat easier for most people, especially those who are already supporting arm64 on iOS but there is still work to be done to ensure that tooling and pre-compiled distributions support both architectures ready for when Mac using Apple Silicon are made publicly available. If you haven't already seen it, a lot of this is covered in the [Port your Mac app to Apple Silicon](https://developer.apple.com/videos/play/wwdc2020/10214/) WWDC Session Video.

If you're using Xcode to compile your command line tools then things are pretty simple as long as you are setting the `ARCHS` build setting to `$(ARCHS_STANDARD)` (the default). In Xcode 12, this value is described as **Standard Architectures (64-bit Intel and ARM)** but if you're using Swift Package Manager to build and distribute your binary or library, there is no such option.

Instead, starting in Swift Package Manager for Swift 5.3 (Xcode 12), the `swift-build` executable has now introduced the `--arch` option ([apple/swift-package-manager#2787](https://github.com/apple/swift-package-manager/pull/2787)).

## Building a Universal Binary

Firstly, make sure that you are using the correct version of Xcode/Swift:

```
$ xcrun swift build --version
Swift Package Manager - Swift 5.3.0
```

_**Note:** If this is not Swift 5.3 or greater, use `xcode-select -s` to switch to the Xcode 12 beta._

Now, when compiling your package, specify both architectures to compile a Universal Binary:

```
$ xcrun swift build -c release --arch arm64 --arch x86_64
```

To verify that your built binary contains both architectures, you can use the `lipo -info` command to inspect a binary and confirm:

```
$ lipo -info .build/apple/Products/Release/swiftlint
Architectures in the fat file: .build/apple/Products/Release/swiftlint are: x86_64 arm64
```

And there you have it, building your Swift Package as a Universal Binary is as simple as that!
