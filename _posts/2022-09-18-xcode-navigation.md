---
layout: post
author: Igor Belov
title: Efficient navigation in Xcode
---

When you have been developing and growing your project for a long time, eventually, it becomes difficult to search sources or rename entities across many files. In this article I would like to talk about the available Xcode tools for quick and easy navigation within the project environment, **which I use daily myself.**

<!-- more -->

## Navigation in project structure

### Open Quickly

The most frequent tool I use is Open Quickly, which appears as a prompt in the center of the editor and allows you to search for files, class declarations, functions, and more. One way to call it is to go to the `File` menu and press the `Open Quickly` button, but the fastest and most convenient option, in my opinion, is a keyboard shortcut `Cmd+Shift+O` (letter O, not zero) that does the same thing:

![Open Quickly](/media/xcode-navigation/quick-open.gif)

### Sidebar Filter

An alternative to the previous method for **finding the files** is also **the filter bar**, which you can find at the very bottom of the project navigator in the sidebar. This field allows you to filter files by some part of the name or, using the two buttons to the right of it, show **recently used files** or **changed files** (information about which comes from the local version control system). I often use this tool to find the files I need in the complex project hierarchy:

![Sidebar Filter](/media/xcode-navigation/filter.gif)

### Reveal in Project Navigator

When you work with a large number of files at once, it is easy to *get lost in the Project Navigator*, as its focus gets lost when you switch file tabs. However, you can quickly find the opened file in the Project Navigator using the `Reveal in Project Navigator` button from the `Navigate` menu or the quick and easy keyboard shortcut `Cmd+Shift+J`:

![Find](/media/xcode-navigation/reveal.gif)

## Text search and replacement

I guess you already know that the main tool for searching and replacing text is in **the fourth tab of the sidebar**, but to get there some developers usually make a few clicks and only then start printing the request. Instead, you can use the `Cmd+Shift+F` shortcut that is intuitive enough:

![Find](/media/xcode-navigation/find.gif)

However, despite the popularity of this tool in Xcode, not many people know that this text search and replace tool also supports the same operations with **regular expressions.** In this combination, you get a powerful text processing tool that can reduce the time it takes to complete your task!

Here I can quickly and easily change the `Just` prefix to `Some` among the many entities with a similar name:

![Regex](/media/xcode-navigation/regex.gif)

## Conclusion

So, I have shown you some tools that I use daily myself to navigate efficiently and quickly within Xcode. I am constantly on the lookout for new solutions to optimize my processes, so if I find something new worthy of attention, I will be sure to share it with you! In the meantime you can try to see what the `Cmd+L` key combination does...
