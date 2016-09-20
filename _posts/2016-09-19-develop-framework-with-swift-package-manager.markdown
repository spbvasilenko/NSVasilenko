---
layout: post
title:  "Develop your framework with Swift Package Manager"
date:   2016-09-19 00:00:00
categories: swift ios swift-package-manager
---

The Swift Package Manager is a tool for managing the distribution of Swift code. It’s integrated with the Swift build system to automate the process of downloading, compiling, and linking dependencies.

<b>Why?</b>

Many existing Swift packages are available, but currently no central listings service like NPM exists, so finding them can be hard. One option is The IBM Swift Package Catalog, but it contains a mixture of CocoaPods, Carthage and Swift packages. When we develop its own framework , we must ensure that we will deliver it to our users. There many of post's about using SPM with your web-apps (lol :D), but nothing about develop own framework with SPM. 

<b>The problem</b>

It's really painfull build our Package of framework in terminal via ```swift build``` without autocompletion and all features of IDE. 

It's easy to fix in two steps:

<b>Let's try</b>

The first step to using Swift is to <a href="https://swift.org/download">download<a/> and install the compiler and other required components. Go to the Download page and follow the instructions for your target platform.

The default location for the downloadable toolchain on macOS is /Library/Developer/Toolchains. You can make the tools available for use from the terminal with the following command:

```shell
export PATH=/Library/Developer/Toolchains/swift-latest.xctoolchain/usr/bin:"${PATH}"
```

<b>First step</b>

Imagine we have already folder of our framework with some class, like on screenshot

<img src="https://habrastorage.org/files/589/977/e11/589977e1114149768ba18f6cf368ba1b.png"/>

Firstly, we should init Package for your framework with command in Terminal

```shell
swift package init --type executable
```

After installing Package for our Framework we have a structure as on screenshot:

<img src="https://habrastorage.org/files/58e/c3d/efa/58ec3defa3ec48b697fbca65bff6d6e4.png"/>

Let's move our some class to Sources folder

So, now we have executable Package of our framework, also we can build it with command: 

```shell
swift build
```

After the command finishes, the built products will be available in the .build directory. Run the Framework program with the following command:

```shell
.build/debug/Framework
```

<b>Second step</b>

In our folder of Framework run command for generate xcproj with Package via command:

```shell
swift package generate-xcodeproj
```

<img src="https://habrastorage.org/files/89e/aa6/1db/89eaa61dbac04288b1ddc750ef75658e.png"/>

Open generated project and setup main scheme for executable type:

<img src="https://habrastorage.org/files/7c9/740/208/7c9740208d9d42abb7dfc5b97338a44f.png"/>

After all step we now can easily develop framework in Xcode with all features of IDE like debugging, etc.

<img src="https://habrastorage.org/files/ebb/afd/d59/ebbafdd596fc4a118765e9619a2b6e6d.png"/>
