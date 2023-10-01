---
layout: post
title: Taking your Strings Catalogs to the Next Level
keywords: xcode15, xcstrings, strings catalog, wwdc2023, ios17, localization, i18n, nslocalizedstring
---

In Xcode 15 Apple introduced Strings Catalogs, a new file format (`.xcstrings`) that can be used as a single source for all of your apps localized string content.

Prior to Strings Catalogs, you typically defined your standard strings in a `.strings` file and your plural variations in a `.stringsdict` plist. Not only were these formats quite verbose, but you also had to then maintain a copy of each file for every supported language too.

The friction in this process often leads to compromises in one way or another and these compromises ultimately end up impacting the users of our apps, so it's an understatement to say that I'm excited about the improvements that have shipped with Xcode 15 this year 🎉

A single Strings Catalog file can now contain regular strings, plural variations and device variations across all of the languages that your app supports. What is even better is that at compile time in Xcode 15, the contents of your Strings Catalog are converted back into `.strings` and `.stringsdict` resources that allow you to take advantage of the file format without having to change your deployment target!

![A screenshot of the Strings Catalog editor in Xcode](/public/images/xcstrings/strings-catalog.png){: .center-image }

So if a Strings Catalog is so great already, how can we possibly go about taking it to the next level?

The short answer is to use a new tool that I created called [XCStrings Tool](https://github.com/liamnichols/xcstrings-tool), but if you're interested in understanding why I developed this tool, then please do read on:

- [**How did we get here?**](#how-did-we-get-here)
  - [**Localization Workflows - The Apple Way**](#localization-workflows---the-apple-way)
  - [**String Extraction - The downsides**](#string-extraction---the-downsides)
  - [**Creating Strings in your Strings Catalog**](#creating-strings-in-your-strings-catalog)
  - [**So lets define some constants**](#so-lets-define-some-constants)
  - [**But what about the arguments?**](#but-what-about-the-arguments)
  - [**Recap**](#recap)
- [**XCStrings Tool**](#xcstrings-tool)

# How did we get here?

I've been working with Apple's localization tooling for a while now, and if you have too, you might be familiar with some of the topics that I discuss. You might also be aware that there are existing tools out there such as [R.swift](https://github.com/mac-cain13/R.swift) and [SwiftGen](https://github.com/SwiftGen/SwiftGen), but for the sake of a story, lets go all the way back to the beginning.

## Localization Workflows - The Apple Way

When it comes to localizing your projects, Apple have always promoted a localization experience that typically starts with you defining your localized string content directly in your SwiftUI or UIKit view code using string literals.

Lets take some example code from Apple's [Food Truck](https://developer.apple.com/documentation/swiftui/food_truck_building_a_swiftui_multiplatform_app) sample app:

```swift
struct SocialFeedPlusSettings: View {
    @ObservedObject var controller: StoreSubscriptionController
    @AppStorage("showPlusPosts") private var showPlusPosts = false
    @AppStorage("advancedTools") private var advancedTools = true
    @Environment(\.dismiss) private var dismiss

    var body: some View {
        List {
            SubscriptionStatusView(controller: controller)
            Section("Settings") {
                Toggle("Highlight Social Feed+ posts", isOn: $showPlusPosts)
                Toggle("Advanced engagement tools", isOn: $advancedTools)
                NavigationLink("Social-media providers") {
                    EmptyView()
                }
            }
            #if os(iOS)
            Section {
                NavigationLink {
                    StoreSupportView()
                } label: {
                    Label("Subscription support", systemImage: "questionmark.circle")
                }
            }
            #else
            Section("Subscription support") {
                Button("Restore missing purchases") {
                    Task(priority: .userInitiated) {
                        try await AppStore.sync()
                    }
                }
            }
            #endif
        }
        .navigationTitle("Manage Social Feed+")
        .toolbar {
            #if os(iOS)
            let placement = ToolbarItemPlacement.navigationBarTrailing
            #else
            let placement = ToolbarItemPlacement.cancellationAction
            #endif
            ToolbarItemGroup(placement: placement) {
                Button {
                    dismiss()
                } label: {
                    Label("Dismiss", systemImage: "xmark")
                        #if os(macOS)
                        .labelStyle(.titleOnly)
                        #endif
                }
            }
        }
    }
}
```

In the code above, there are 8 strings that need to be localized:

- Settings
- Highlight Social Feed+ posts
- Advanced engagement tools
- Social-media providers
- Subscription support
- Restore missing purchases
- Manage Social Feed+
- Dismiss

You aren't expected to manually copy these strings into your Strings Catalog, instead, building your project with the **Use Compiler to Extract Swift Strings** build setting (enabled by default) will tell Xcode to automatically populate these strings into the Strings Catalog for you.

Sounds great? Well it actually is. But like many things with Apple, we can quickly find that this process doesn't quite scale how we might hope that it does.

## String Extraction - The downsides

_The Apple Way_ holds up pretty well for relatively small apps that don't have a text-heavy UI, but as the complexity grows, the cracks in this process start to show.

Relying on the extraction of string literals alone is a great start, but what if we need to think about some other things:

- Providing context to translators via comments
- Breaking out our translations across multiple localization tables
- Using translations from different targets/modules
- Scenarios where a phrase in the source language could mean different things based on context in another language

For the extraction of a string literal like `Text("Settings")` to be sufficient, a few things have to be true:

1. The localization key is called `Settings`.
2. The value of the localization in the default language is **Settings**.
3. A translator can infer the context from the word **Settings** alone.
4. The localizations for this phrase are in the file called **Localizable.xcstrings**.
5. The Strings Catalog is found in the Apps main bundle (`Bundle.main`).

In one of the more extreme scenarios, you might find that your simple string literal ends up having to become something like the following:

```swift
Text(
  "Settings",
  tableName: "Social", // Social.xcstrings
  bundle: .module, // The correct Bundle for Swift Package target resources
  comment: "A section heading title for the Social Feed Plus configuration options"
)
```

To cover all cases, we need 4-6 lines of code to properly describe the word _Settings_. Considering how far you can get with 4 lines of SwiftUI, it seems absurd that a single localized string might take up this much code in your view.

## Creating Strings in your Strings Catalog

Once you find that defining all of your localized Strings context in your Swift source code isn't good enough, you will instead want to start manually defining the values in your Strings Catalog instead.

By doing so, we instantly gain two benefits:

1. We have more control over the localized string key
2. We've moved our translator comments out of the source code

![A screenshot of the Strings Catalog editor in Xcode](/public/images/xcstrings/settings-string-in-catalog.png){: .center-image }

In the String Catalog above, we've manually added a string with the key `settingsHeading` by clicking the **+** button at the top of the editor. You can tell that this string was manually added because the attributes inspector (right) shows it as **Manually** managed.

> **Note:** If a string is **Automatically** managed, it Xcode will not allow you to edit it's values in the source language within the Strings Catalog and it will delete the string if it cannot find the key referenced at compile time.

The default value of this string (_Settings_) and the comment remain defined in the Strings Catalog, so when we can reference the string in our UI code, we can do so like the following:

```swift
Text("settingsHeading", tableName: "Social", bundle: .module)
```

Essentially, we've ditched the translator comment, which is an improvement, but by following this approach we've also gained another problem. Now that Xcode isn't automatically managing the string in our Strings Catalog, the source of truth has shifted from the Source Code to the Strings Catalog.

This isn't such a bad thing, but our Strings Catalog isn't exposing any Swift code for us to reference in our project. Instead, we have to use string typed keys as if we're living in the stone age.

## So lets define some Constants

When you find yourself having to work with a string typed API repeatedly, especially if you need to reference the same key more than once, a common way to keep this under control is to define a set of constants to help reduce the risk of typos and maintain a bit of consistency in your project. For example:

```swift
struct SocialStrings {
  static let settingsHeading: LocalizedStringKey = "settingsHeading"
}

// ...

Text(SocialStrings.settingsHeading, tableName: "Social", bundle: .module)
```

And starting in iOS 16 and macOS 13, we can even bring the table and bundle configuration into this constant using Foundation's new [`LocalizedStringResource`](https://developer.apple.com/documentation/foundation/localizedstringresource) type:

```swift
struct SocialStrings {
  static let settingsHeading = LocalizedStringResource(
    "settingsHeading",
    table: "Social",
    module: .atURL(Bundle.module.bundleURL)
  )
}

// ...

Text(SocialStrings.settingsHeading)
```

> **Note:** The `LocalizedStringResource` type isn't supported directly by all SwiftUI views/modifiers so sometimes you need some workarounds (FB13221647).
>
> There are a few ways to workaround this in the meantime:
>
> ```swift
> // 1. Use an alternative method/overload that accepts Text
> Button(action: { /* ... */ }) {
>   Text(SocialStrings.dismissTitle)
> }
>
> // 2. Use a method/overload that accepts LocalizedStringKey and then use string interpolation
> Button("\(SocialStrings.dismissTitle)", action: { /* ... */ })
>
> // 3. Use a method/overload that accepts String and resolve the localized value first
> //    Note: by using this approach, custom locale information set in the environment might be ignored
> Button(String(localized: SocialStrings.dismissTitle), action: { /* ... */ })
> ```

This is a great solution that is relatively straightforward to implement. It looks like a winner right?

## But what about the arguments?

We haven't looked at passing arguments into localized strings yet, so lets go back to teh start quickly:

```swift
Text("There are \(items.count) pending posts")
```

With the example above, when the compiler extracts this string, it will assign the key `There are %lld pending posts` inside the Strings Catalog.

But we've decided to manually define our keys using an identifier style format and format specifiers don't really fit in this pattern, so what do we do?

Because each string in the Strings Catalog has a distinct field for the Key and the Value, you can define your strings containing variables like so:

![A screenshot of the Strings Catalog editor in Xcode showing a string with an argument](/public/images/xcstrings/settings-variable.png){: .center-image }

The trick is then to use `LocalizedStringResource`'s `defaultValue` parameter:

```swift
static func feedSummary(_ count: Int) -> LocalizedStringResource {
  LocalizedStringResource(
    "feedSummary",
    defaultValue: "There are \(count) pending posts",
    table: "Social",
    module: .atURL(Bundle.module.bundleURL)
  )
}

// ...

Text(SocialStrings.feedSummary(items.count))
```

The `defaultValue` parameter is not a `String`, but instead it's a [`String.LocalizationValue`](https://developer.apple.com/documentation/swift/string/localizationvalue) type.

This type allows Foundation to track the values that are interpolated into the literal (such as `count`) and then use them when resolving the actual localized string from the Strings Catalog later on.

## Recap

To recap on the journey that we've been on:

- Apple encourage you to define your localized strings in source code and let the compiler copy them into your Strings Catalog.
- But this isn't a great approach for long strings, scenarios where you need to provide comments, or when you need to specify a different table or bundle.
- The alternative is to reference manually managed strings, but this is done using a string-typed key that is prone to typos.
- To reduce the risk of typos, it's a good practice to use static properties or methods to reference the localized strings in Swift instead.
- Using keys as identifiers to makes providing variables/arguments a bit trickier in the modern `LocalizedStringResource` type.

So to conclude, we see a value in making the Strings Catalog the source of truth for all localized string content, but having to manually define helper/boilerplate accessors in Swift still has annoying downsides.

Hopefully you can see where I am going with this (spoiler: it's not a Macro)...

# XCStrings Tool

I created [XCStrings Tool](https://github.com/liamnichols/xcstrings-tool) as a modern solution to generating Swift code to interface with a Strings Catalog.

1. Integrate the XCStrings Tool Plugin to your target
2. Manually define your Strings in your Strings Catalogs
3. Reference your strings using the accessors that XCStrings Tool adds to `LocalizedStringResource`

![An animated gif showing the XCStrings Tool being used to replace string typed keys in an existing Xcode Project](/public/images/xcstrings/demo.gif)

```swift
// Before
Text("settingsHeading", tableName: "Social", bundle: .module)
Text(
  LocalizedStringResource(
    "feedSummary",
    defaultValue: "There are \(count) pending posts",
    table: "Social",
    module: .atURL(Bundle.module.bundleURL)
  )
)

// After
Text(.social.settingsHeading)
Text(.social.feedSummary(items.count))
```

When added to either an Xcode Project or Swift Package target, the build tool will process each Strings Catalog and generate an extension on `LocalizedStringResource` that can be used to access each localized string within that catalog.

To get started with XCStrings Tool, check out the [documentation](https://swiftpackageindex.com/liamnichols/xcstrings-tool/documentation/documentation#getting-started) hosted on the Swift Package Index.

You can also visit the [GitHub Discussions](https://github.com/liamnichols/xcstrings-tool/discussions) for further support or to provide feedback.
