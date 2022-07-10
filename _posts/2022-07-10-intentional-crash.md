---
layout: post
author: Igor Belov
tags: Swift
title: Ways to cause an intentional crash in Swift
---

Sometimes in development we have the task of causing an **intentional crash of an application** when a condition occurs. This is usually used to debug a piece of code that the user should never get into, or to verify that your [Firebase Crashlytics][firebase] configuration works, for example. 

I recently encountered exactly the second case, and at the same moment I wondered, **what is the shortest code that can cause a crash?** There is an answer to this question, but first I would like to share all the options that can be used for development.

<!-- more -->

## Built-in ways 📦

There are several functions in the standard Swift library that can be used to cause an intentional crash: `fatalError`, `preconditionFailure`, `assertionFailure`.

### fatalError(_:file:line:)

This is the most popular function that is used for this purpose. It takes as input the message that the developer will see at the time of the crash, as well as an indication of the file and line number of the place where it is called:

```swift
struct NetworkManager {
    let serverUrl = "https://api.github.com/zen"

    public func performRequest() {
        guard let url = URL(string: serverAddress) else {
            fatalError("serverUrl is invalid!")
        }
        // Continuation of the request...
    }
}
```

As we can see, in case we can't make the URL from the embedded string (that is a developer error), we cause the application to crash. That way we can be sure that the application won't go to production unless the string is correct.

Before Swift 3.0, this function was denoted by the `@noreturn` decorator, which indicated that there would be no return to the place of call. However, it was [proposed][never] to introduce a `Never` type to make this mechanism more transparent to developers.

> It is worth remembering that the **user will also have calls to this function in the production application**, so you have to be as careful as possible!

### preconditionFailure(_:file:line:)

This function's behavior is similar to that of `fatalError`. However, you should use it to notify the developer about the incorrect use of some API you created. And in that case, you should write explicitly about the limitations of use in the documentation strings:

```swift
enum MediaType {
    case image, video, audio
}

/// Adds an overlay on top of the current video.
/// - Parameter overlayType: Type of overlay (video or image only)
func blendVideoWithOverlay(overlayType: MediaType) {
    guard [.image, .video].contains(overlayType) else {
        preconditionFailure("Unsupported type `\(overlayType)`!")
    }
    // Continuation of the video processing...
}

blendVideoWithOverlay(overlayType: .audio) // Will fail!
```

Here we create a method that supports only two types of media. If an unsupported type is received, we signal this to the developer by calling `preconditionFailure` with message.

> Just like in the case of fatalError, **this call will be on the user side too.** So it's important to make sure that in production the API will be used correctly and there won't be a crash!

### assertionFailure(_:file:line:)

This is the most harmless way to call the application crash for internal checks, because **this function is only called when running in debug mode**. So at development time you can specify some condition that you expect to be fulfilled:

```swift
class TabCoordinator {
    let tabItems: [String] = ["All", "Unread", "Done"]
    
    var lastUsedTab: String? {
        didSet {
            if let lastUsedTab = lastUsedTab, !tabItems.contains(lastUsedTab) {
                assertionFailure("Invalid lastUsedTab: \(lastUsedTab)")
            }
        }
    }
    
    // Business logic goes here...
}

let coordinator = TabCoordinator()
coordinator.lastUsedTab = "Home"
```

Here we implement some business logic for switching between tabs in our application, remembering the last one used. If somehow we remember a tab that is not in the list of tabs, it is a developer error and we will know about it by calling `assertFailure`.

### The exotic way

Most Apple operating systems are based on the [Darwin][darwin] platform, whose system calls we can access directly from Swift by importing a package with the corresponding name. An example of such a call is the `exit()` function, which tells the operating system that **the current process is finishing with some code** that is passed as a parameter. If you have worked with languages of C family, such procedure will be very familiar to you.

So you can also use (but please never do 🙏🏻) this function to cause the application crash:
```swift
import Darwin
exit(1)
```

## The shortest way 🤯

There are countless ways to cause an application to crash without using pre-built functions, but I was interested in the shortest one. I got curious and tried to figure it out on my own, but only came up with the following result:
```swift
// This was the first attempt:
Optional<Int>(nil)!
// And this is a better one that followed:
1...0
```
Everything is clear with the optional (we just force unwrap `nil` value), but in the case of the `Range` it is important to know and remember that it requires `lowerBound` to be less or equal to `upperBound` otherwise it will cause a crash. However, I wondered if there was something shorter, and then I googled it. Finally, here we go:

```swift
[][0]
```

When the execution "goes to" this line of code, a crash will occur because **we are trying to access the first element of an empty array**.

## Conclusion

Causing an application to crash is not a very common development task, but you have to do it once in a while. In this article, I have shared with you a few ways you can do this.

First, there are functions built into the standard Swift library: `fatalError`, `preconditionFailure`, `assertionFailure`. Remember, however, that the first two of these are also present in the production build of the application, so it may crash on the user as well. 

Second, you can also use features of the language to cause a crash. Hence, the shortest way (which is what I was initially trying to find) to do this was to access the first element of an empty array.

That's all, but be careful about causing these crashes, because at some point **you might forget about it and mess up your AppStore stats**.


[never]: https://github.com/apple/swift-evolution/blob/master/proposals/0102-noreturn-bottom-type.md
[firebase]: https://firebase.google.com/docs/crashlytics
[darwin]: https://en.wikipedia.org/wiki/Darwin_(operating_system)