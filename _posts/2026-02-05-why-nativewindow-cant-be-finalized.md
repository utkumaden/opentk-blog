---
layout: post
title:  "Why `NativeWindow` can't be finalized"
date: 2026-02-05 21:35:00 +0100
categories: support-tips
tags: glfw
author: noggin_bops
commentIssueId: 11
excerpt: "In this post we will look at why [`NativeWindow`](https://opentk.net/api/OpenTK.Windowing.Desktop.NativeWindow.html) can't be finalized."
---

In the [previous post]({% post_url 2025-12-28-why-does-glfwprovider-checkformainthread-exist %}) we looked at [`GLFWProvider.CheckForMainThread`](https://opentk.net/api/OpenTK.Windowing.Desktop.GLFWProvider.html#OpenTK_Windowing_Desktop_GLFWProvider_CheckForMainThread).
There we concluded that most GLFW functions need to be called from the main thread.
In this post we are going to look at a another consequence of that: [`NativeWindow`](https://opentk.net/api/OpenTK.Windowing.Desktop.NativeWindow.html) cannot be finalized.

If you ever try to do this by letting the garbage collector collect a [`NativeWindow`](https://opentk.net/api/OpenTK.Windowing.Desktop.NativeWindow.html), you will be greeted with a [`GLFWException`](https://opentk.net/api/OpenTK.Windowing.GraphicsLibraryFramework.GLFWException.html) saying:
> You can only dispose windows on the main thread. The window needs to be disposed as it cannot safely be disposed in the finalizer.

So, why is this? Well, a simple answer is provided in [the previous blog post]({% post_url 2025-12-28-why-does-glfwprovider-checkformainthread-exist %}). Most GLFW APIs need to be called on the main thread. But finalizers in C# are not run on the main thread, they are run in their own special [finalizer thread](https://devblogs.microsoft.com/dotnet/finalization-implementation-details/). This means that we cannot call GLFW functions in the finalizer, which means we can't destroy the GLFW window. So, instead we throw an exception to hopefully notify the programmer of their wrongdoing.

But now you might be asking? I don't dispose of my [`NativeWindow`](https://opentk.net/api/OpenTK.Windowing.Desktop.NativeWindow.html) and I don't get any crashes, what gives?

Well, dotnet makes no guarantee that the garbage collector will actually run the finalizer of an object before the program exits. From the [documentation on finalizers](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/finalizers#:~:text=you%20can%27t%20guarantee%20the%20garbage%20collector%20calls%20all%20finalizers%20before%20exit%2C%20you%20must%20use%20Dispose%20or%20DisposeAsync%20to%20ensure%20resources%20are%20freed.):
> ... you can't guarantee the garbage collector calls all finalizers before exit, you must use Dispose or DisposeAsync to ensure resources are freed.

So, make sure to dispose your [`NativeWindow`](https://opentk.net/api/OpenTK.Windowing.Desktop.NativeWindow.html) either with `using` or by manually calling `.Dispose()`.