---
layout: post
title: "The Fuzz Developer Dilemma — Writing Efficient Fuzz Targets"
date: 2023-07-12
author: Ayush Singh
categories: ['Tutorials', 'Bitcoin Core']
---

For the past 2.5 months, I’ve been working as a Summer Intern at Summer of
Bitcoin, where my job is to write Fuzz harness files for the codebase of the
Bitcoin Core Wallet. If this doesn’t make sense right away, don’t worry; I’ll
do my best to explain it once you’ve finished reading this blog.

Before reading this blog, it is necessary to have a basic understanding of
what fuzzing is. If you are unfamiliar with this type of testing, I recommend
reading my previous blog (_here_). If you’re seeking a more comprehensive
understanding, take a look at this: _OSS-Fuzz doc_.

Assuming you are familiar with the fundamentals of fuzzing, let’s explain what
a fuzz target is and how it differs from a fuzz harness.  
Keeping this quick, Fuzz Harness is often the file in which you write the Fuzz
target.

# Fuzz Target

Imagine it as an API or a gateway, facilitating the entry of randomness into
your codebase. Sounds pretty fancy, right? It is really interesting. Let’s get
deeper into this.

OSS-Fuzz defines a Fuzz Target as a function that takes an array of bytes and
applies the API under test to do something fascinating with those bytes. To
clarify, let’s look at an example.

Suppose you want to do Fuzz Testing of the `CreateTransaction` function in the
Bitcoin Core Wallet codebase (Don't worry if you don't understand the Bitcoin
Core codebase, just think of it like any other C++ function you want to fuzz).

Your current task involves calling this function in your Fuzz Harness file,
where random bytes from the Fuzzer serve as input. The challenge is to do so
in a manner that maximizes Code Coverage, leading to an intriguing mess I like
to call “**The Fuzz Developer Dilemma** ”.

# The Fuzz Developer Dilemma

BTW, I’m just making this up; it’s not a thing. I refer to this as a dilemma
because there are two possible types of inputs for the CreateTransaction
function: **Good inputs** and **Random inputs**.

  1. **Good Input:** This type of input contains ideal values for all of the parameters required by CreateTransaction. For example, the Wallet will include funds, and Coin Control will contain valid settings as well as selected Coins to initiate a transaction. If you leave Bitcoin Core running, it will only generate this type of input. Furthermore, because these inputs are valid, they have **Maximum Code Coverage** because the code allows these inputs to propagate deep inside it.  
But is this even Fuzzing if the inputs aren’t random, strange, or even in-
valid sometimes?

  2. **Random Input** : This is the type of input you truly want to give to a function while Fuzzing it. These inputs will discover crashes that the developers might have overlooked.

  * Imagine giving a `Wallet`with 0 funds to `CreateTransaction`.
  * Imagine supplying a `Coin Control`to `CreateTransaction` that has faulty settings and selects random, false coins for Transaction creation.

These are the type of inputs that make your Code go crazy, behaving in a
manner that you would never expect it to. That is exactly what we want while
fuzzing!  
However, this has the **lowest Code Coverage** because your code will avoid
propagating these kinds of inputs deep into the codebase (which is good by the
way) as these inputs will make no sense to it, making the Fuzzing less
effective as the code coverage is low.

**So, what should a Fuzz Developer do to strike the right balance between Good
Inputs and Random Inputs**?

Well, that’s where the art of writing an effective Fuzz Target comes into
play.

An effective Fuzz Target is one that intelligently combines both types of
inputs to achieve a comprehensive Fuzz Testing approach. By strategically
mixing good inputs with random inputs, you can increase the chances of
discovering hidden vulnerabilities while still maintaining decent Code
Coverage.

One can achieve this by using a **_Random Switch Case_** where in one case you
can update the input value to a Good Input and in the other case update it
with a Random Input. So, “sometimes” you are calling `CreateTransaction` with
Good input and “sometimes” not.

We use `CallOneof` function for creating a **_Random Switch Case_** in Bitcoin
Core. `CallOneof` can also be used to generate different **_Code Paths_**
during fuzzing, we will see this later in the blog.

# Ways to improve Fuzz target efficiency?

Until now, we focused mainly on Code Coverage, but there is another important
factor that influences your Fuzzing process, and that is **Speed**.

By Speed, I mean `Execution per second` (exec/sec). The reason why this is
important is that your Fuzz Target file is going to be executed repeatedly
with different mutated inputs. If your file is slow and each execution takes a
significant amount of time, the overall fuzzing process will be greatly
hindered. The time it takes for each execution accumulates as the Fuzzer
explores various inputs, potentially leading to a much longer time to discover
bugs or vulnerabilities. Therefore, it’s crucial to optimize the speed of your
Fuzz Target file to achieve a higher exec/sec rate, allowing the Fuzzer to
cover more test cases efficiently and increase the chances of uncovering
potential issues.

One should aim for at least _1,000 exec/sec_ according to the general
consensus in the ecosystem.

# Here are different ways to speed up your Fuzz target:

  1. **Write efficient Setup:** A setup function is just a function that is executed before the start of the Fuzzing process; its job is to build the environment needed to run that specific Fuzz Target. It is best to only set up things that are essential for Fuzzing. Setting up a variable that will never be used inside the Fuzzing, for example, is pointless because it will take time unnecessarily.
  2. **Move Redundant things inside the****Setup****:** All duplicate setup codes should be moved inside it so that it is not repeated over and again in each Fuzz run.
  3. **Optimize the Fuzzing Algorithm:** Explore different ways to write the same target file and select the one that is the fastest.
  4. **Avoid big Loops inside your target:** One should avoid having unnecessarily large loops inside their Fuzz target as they would take time to execute. But doing so shouldn’t be done by reducing loops when they are required. Priorities the functionality of the test first.
  5. **Use the Singleton Class:** If you find yourself repeatedly creating instances of a particular object during each fuzz run, it can be advantageous to implement the Singleton design pattern. By employing the Singleton class, you ensure that only a single instance of the object exists throughout the entire fuzzing process, saving valuable time spent on redundant object creation and initialization. Check my code for this — here.
  6. **At last, try to think for yourself which part of the code is time-consuming and make it faster!**

# Conclusion

Last but not least, I just wanted to note that many developers don’t make
their Fuzz Target efficient and simply rely on the fuzzing software to do the
job, but in my opinion that is not the way to do it. Writing the best possible
target should always be the goal!