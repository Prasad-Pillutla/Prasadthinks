---
layout: post
title: "Parallelism on Multi-core Processors"
date: 2015-08-04 16:55:13 +0000
comments: true
categories: Performance, Parallel Programming
---
Parallelism, a dark art, no one gets this right first time and many times after that as well. This is one area in software that cannot be understood without understanding underlying hardware(processor). To begin with let me explain the evolution of hardware and then dive into software aspects.

Until early 2000’s processors were single core and were capable of executing only one instruction at a time. Focus of Intel and AMD was on adding more cycles to processor so that more instructions can be executed in a second and thereby reduce overall time that a program takes to execute. After reaching to certain point, heat sink issues didn’t allow adding any more cycles to processor.

Then came multi-core processors, started with 2 cores and now we have 8 cores in consumer devices. But how many cores we can add? Each core produces certain amount of heat in a given time, which needs to be dissipated at equal rate so that the processor doesn’t meltdown. With every additional core that gets added there is a need to add more infrastructure to dissipate heat generated processor, which becomes unmanageable both from size and cost perspectives. This is when hardware geeks say we are done and we can’t improve processor to give more cycles because they are constrained by __Laws of physics.__

Now it is up to the software pros, to show some skill. We’ll come to skill part in a minute but before that do software programmers understand underlying hardware? Do they write hardware optimized code; I meant do they write programs that use all the cores that are given to them today? Good question, let me try out an example

    namespace CompareTPL
    {
        class Program
        {
            static int addPara(int num)
            {
                int j = 0;
                for (int i = 0; i < 10000000; ++i)
                {
                     j = i * i;
                }

                return j;
            }

            static void Main(string[] args)
            {
                for (int i = 0; i < 1000; ++i)
                {
                    addPara(5);
                }
            }
        }
    }

A simple for loop, which madly iterates over a large number and computes product of two numbers. When I run this program on an __Intel I7 quadcore machine with 16GB RAM__, what do I see in [perfmon](https://msdn.microsoft.com/en-us/library/ff727783.aspx)

![](/images/Seq_processing.jpg)
Histogram bars tell me that above program is not utilizing all cores efficiently and also the load is unevenly distributed. This means that above program will take longer time to execute.

Now I’ll modify program to use multiple threads to achieve parallelism by replacing __for__ with __Parallel.For__

    namespace CompareTPL
    {
        class Program
        {
            static int addPara(int num)
            {
                int j = 0;
                for (int i = 0; i < 10000000; ++i)
                {
                    j = i * i;
                }

                return j;
            }

            static void Main(string[] args)
            {

                Parallel.For(0, 1000, i =>
                {
                    addPara(5);
                });
            }
        }
    }

Now what does [perfmon](https://msdn.microsoft.com/en-us/library/ff727783.aspx) show
![](/images/Parallel_Processing.jpg)

Load is equally distributed among 4 cores and all cores are utilized to their maximum limit. This means the program will run faster. On my machine (__Intel I7, 4 cores, 16GB RAM__) it took 2.487 seconds to complete __Parallel__ version and 5.320 seconds to complete __simple for__ loop version. It tells me that using all cores effectively will execute program in less time. I’ve added __.NetCLRLocksAndThreads__ counters to detect any contentions (in yellow) during execution of this program. Even though there are multiple threads executing same set of instructions there are no contentions reported.

Let’s make this more interesting by replacing i*i with a console statement

    class Program
        {
            static int addPara(int num)
            {
                int j = 0;
                for (int i = 0; i < 10000000; ++i)
                {
                    Console.WriteLine("Test Parallel");
                }

                return j;
            }

            static void Main(string[] args)
            {
                Stopwatch s = new Stopwatch();
                s.Start();
                Parallel.For(0, 1000, i =>
                {
                    addPara(5);
                });
                s.Stop();
                Console.WriteLine(s.ElapsedMilliseconds);
            }
        }
    }

What do I see from perfmon now? Utilization of cores is 100% and load is evenly distributed among 4 cores.
![](/images/Contention.jpg)

Total Number of contentions (Total # of Contentions) have gone up significantly and Contentions Rate Per Second has also gone up. Which resource is causing the contention? Console, yes console is the resource under contention. Only one thread can write to console at any moment. For writing to console, a thread has to request for lock on __Console__ and wait until it gets lock. There are multiple threads waiting for the same lock/resource and this has a profound impact on performance and execution time.

This means that threads created by __Parallel.For__ are waiting for Console to write to it. The same program (with Console) takes more than 30 minutes to complete, which earlier took 2.487 seconds (with no console but multiplication of two numbers i*i). Locks and Contentions are very costly and can kill an application by choking the performance.

Other kind of dependencies that slow down the application are
    int op1 = a + b;
    int op2 = op1 + c;

In the above code fragment, second expression(op1 + c) cannot be executed until first expression(op1 = a + b) is executed because second expression needs result of first expression(op1) as input. This instruction level dependency clips compiler and forces it to generate sequential instructions which operating system cannot execute in parallel on available cores.

These are the challenges that a hardware engineer cannot address and only software engineer can. No matter how many more cycles we add to processor we cannot improve program’s performance because there is a flaw in the program that is stopping it from being executed in parallel on available cores.

From above examples, it is evident that we have enough processing power today to get things done decently fast and we are not writing programs to make full use of underlying processors. Hence I support hardware guys not adding any more cycles to processor because they already have added more cycles than what a fairly complex program needs.

The point I was trying to explain is that no matter how much parallelism we introduce if there is a contention for resource we’ll not reap benefits. To conclude, I’ll summarize things that one needs to remember when introducing parallelism to gain performance

* Today’s Processors and Operating systems are smart enough to distribute load evenly among to all available cores.

* Programmer has to make sure that all threads or tasks that he/she are introducing are fairly independent so that there are no contentions. This will allow compiler to break instructions into independent instructions which Operating System can execute in parallel on different cores.

* Deploying an under-performing application on a multi-core platform will not improve performance beyond certain limit. Application has to go through refactoring to reap maximum benefits of multi-core platforms on which it is deployed.

* Look into Contentions and processor utilization from Perfmon to evaluate your program’s performance. If you notice contentions or underutilization of available processor cores, then focus on refactoring code.

In next blog I’ll try to explain differences between Parallel and Asynchronous programming; and also explain when to use which paradigm.