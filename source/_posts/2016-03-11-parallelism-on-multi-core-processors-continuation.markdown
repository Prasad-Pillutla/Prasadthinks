---
layout: post
title: "Parallelism on Multi-core Processors: Continuation"
date: 2015-10-10 14:29:53 +0000
comments: true
categories: 
---
Thanks for taking time to read through my last post, [Parallelism on Multi-core Processors](http://www.prasadthinks.com/blog/2015/08/04/parallelism-on-multi-core-processors/). I was asked by many on how to handle calls to resources which are not thread-safe and have to be locked either by caller or by library, which exposes the resource. Let me pick the same example that I was using in [previous post](http://www.prasadthinks.com/blog/2015/08/04/parallelism-on-multi-core-processors/) but simplifying it in the interest of time

    for (int i = 0; i < 10000; ++i)
    {
        Console.WriteLine("Test Parallel");
    }

How much time does this for loop take to complete? Ok, How much time would a parallel version of it take to run? Guesses???

    Parallel.For(0, 10000, i =>
    {
        Console.WriteLine("Test Parallel");
    });

On my laptop, simple for loop took **~4 seconds** and parallel for took **~6 seconds**.

If you thought parallel loop will take less time, then you are wrong? Why does parallel version take more time? I’ll try to keep it simple and try my best not to confuse

Let us assume a scenario where there are 2 threads (T1 and T2) and a resource which can be accessed by only one thread at a time, like resources like **console**, **file handle**, etc. In this scenario let us assume that Thread T1 started first and acquired lock on console and then Thread T2 started and tries to acquire lock on console but since it is already taken by Thread T1, Thread T2 will wait for X nanoseconds before attempting again to acquire lock. If lock is still not available Thread T2 will wait for X nanoseconds once again before attempting again to acquire lock and this continues until Thread T2 acquires lock or until it times out(if applicable), whichever is earlier. There is a fixed amount of time that a thread has to wait before attempting to acquire a lock and thread will not be informed about availability of lock so that it can come out of wait/sleep and take the lock. **This is the overhead of contention and cannot be negotiated**. This is what exactly is happening in Parallel For loop and leading to more execution time than simple for loop.

Above explanation is overly simplified contention management in **.Net** but in reality it is much more complex and sophisticated. For the purpose of this topic above explanation serves enough.

To conclude,**whenever there is a contention for resource or a resource can be accessed by only one thread at a time it is always advised to take sequential route than parallel route for performance benefits**.

Let me know what you guys think…