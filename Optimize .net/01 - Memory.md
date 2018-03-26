### Forewords

This is the first blog post of a series about understanding how to improve performances by developping with .net core and C# 7.2.

This series is intended for any kind of readers, especially the ones who a not familiar with the topic and are willing to understand the basics.

For the experts on the matter, you may find these posts are lacking depth, but it's on purpose: the goal is not to thouroughly explain everything, it would be too big and ending up confusing most readers, but instead explaining what matters, why and how to deal with it.

1. Understanding the memory.
2. The benefis of working with `struct`.
3. Working with Data Stores.
4. Working with `Memory<T>` and `Span<T>`.

### Introduction

It's not a secret that Microsoft decided to focus on improving performance for the 2.1 release of .net core.

The main driver is to improve asp.net core but it doesn't mean the new features only target the web server. Most of the time you have to dig to the lowest layer in order to bring game changer when it's about performances and this time was no exception.

What is interesting, from my point of view, is that we're starting to see some features that bring us closer to low level/high performance language such as C++.

The goal of this post series is to :

1. Explain what matters when were dealing with optimization.
2. How you can use the new features (and also some of the old ones) to improve the code speed while keeping things clean and well designed.

C# is about writing clean code to achieve high maintainability and meet good programming practices/standards. Writing optimized code often drives you away from these, finding the right balance is definitely a key aspect for the programmer.

### Why .net is slower than C++ ?

Well, there are many reason and I won't detail all of them, mostly because I couldn't, but there's some of them we can focus on:

1. Seamless control over the lifetime of object through the use of Garbage Collection afraid people who are performance/real-time driven.
2. No direct access to memory, through pointer and boundless check (we are not considering unsafe .net, of course).
3. It is easy to not pay attention to the layout of the data.
4. A lot of implicit memory copy. Things are easy to develop, but under the hood you don't realize all the memory bandwidth that is consumed.
5. A JIT that doesn't generate code as efficient as a pre-compiled language.

C# is a pretty high level programming language, it's pretty easy/safe to use, that's why you have things like the bullets #1 to #4 above. On the other hand it's also easy to not being aware of what matters to optimize things up.

Let's not focus on the #5, because there's few things we can do about it. If we take a close look at #1 to #4 we will see there's a common theme: memory!

Is memory important? **Yes, you bet!**

#### A bit of talk about memory

CPUs are getting more and more powerful the years passing by, but we don't see the same trend going on for memory, see below:

![Processor vs. memory speeds][1]

<p style="padding-left: 30px;">
  <em><cite>Computer Architecture: A Quantitative Approach</cite> by John L. Hennessy, David A. Patterson, Andrea C. Arpaci-Dusseau</em>
</p>

It means that in order to keep the CPU busy we have to develop our code & data in a memory friendly way, because accessing data directly to memory will cost more than you may think!

There's a very good analogy that you can [read here][2] that basically gives you crucial information and with a sense of perspective.

**Let's summarize it.**

Today, most of the CPU instructions that don't involve memory access or very complex computation will take one cycle to execute, you have a 4Ghz CPU so it's 4 billions instructions per second per logical core (so 32 billions for a hyper-threaded quad cores).

Let's scale things to understand their impact better:

* We pretend that **one CPU Cycle** is taking **1 second** (in reality it's 0.4ns).
* A **Level 1** Cache access would be then **2 seconds** (in reality 0.9ns).
* A **Level 2** Cache access would be **7 seconds** (2.8ns).
* A **Level 3** Cache (assuming you have L3) access would be **1 minutes** (28ns).
* **Main memory** access would be **4 minutes** (~100ns).

Compared to a CPU cycle, L1 access is 2x slower, L2 access is 7x, L3 access is 60x and memory access is **250x** slower!

So yes, you can understand that the more you are memory friendly (we'll explain roughly what it implies) the better you'll have chances to hit the CPU cache, getting you significant performance boost!

A L2 access is 38 times faster than a memory access, a L3 is 3.5 times faster (having an app 3.5 times faster is something that many will be happy about, I'm sure!).

So worrying about the JIT not being fast enough may not be the main reason, you can leverage thing yourself, improving your code!

### Enough of the theory, how could we make things faster in .net?

#### Minimizing the usage of the Garbage Collection

Yes, the GC is a very nice and handy feature, but as each feature, it's not a silver bullet, it's not something you have to rely on 100% of the time, **and definitely not in .net!** The GC is only used when you involve `class` based types, `struct` ones are not. So yes, there're ways to minimize pressure on the GC and you should know about them!

#### Direct/fast memory access, avoiding copies

It's easy to copy data, to isolate it for the sake of a good design (or easy and well readable code), it may not harm when the size is small and the frequency of the operation is low, but when one of these two factor increase, things amplify and performances are dropping. Luckily for us we have new weapons to improve things on this area.

#### Designing the data in a more memory friendly way

C# is a high-level language, we don't pay attention to how we define the data in the types we design and it's a big mistake when we want things to be driven by performances. Again, this is more about convenience, because the language don't prevent you to improve things: you just don't know/care to do it.


 [1]: https://assets.bitbashing.io/images/mem_gap.png
 [2]: http://www.prowesscorp.com/computer-latency-at-a-human-scale/