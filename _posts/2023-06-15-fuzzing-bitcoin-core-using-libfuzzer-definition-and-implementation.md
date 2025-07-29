---
layout: post
title: "Fuzzing Bitcoin Core using LibFuzzer: Definition and Implementation"
date: 2023-06-15
author: Ayush Singh
categories: ['Stories', 'Bitcoin Core']
---

## Harness file

The harness file is a small modification of the source code of the file to be
fuzzed that serves as an entry point for fuzzing. It performs the initial
setup required to perform fuzzing on a certain file and also provides a
mechanism to directly fuzz input into the codebase when doing so via the
command line or another traditional method is not possible.

These harness files contain the Fuzz target method implementation, which is
used to enter data into the program.

Once you have the harness file and Clang C compiler installed on your system
you are ready to do some fuzzing!

# libFuzzing the Bitcoin Core

Now let’s see how to do fuzzing of Bitcoin Core software using the libFuzzing
tool.

> _Note: I’ll be using a Linux environment for this tutorial, so if you are on
> Windows kindly run these commands on WSL instead of the Command Prompt._

All the fuzz harness files are present in this directory:

    
    
    src/test/fuzz/

## Installing libFuzzer

Since libFuzzer is already integrated into the Clang compiler, so we only need
to install `clang` and `gcc`.

    
    
    sudo apt install gcc  
    sudo apt install clang

## Let's start fuzzing

As libFuzzer allows us to fuzz individual source code files, for this tutorial
we are going to fuzz the
[_banman.cpp_](https://github.com/bitcoin/bitcoin/blob/master/src/banman.cpp)
file.

The first step is to clone the Bitcoin Core source code from GitHub:

    
    
    git clone https://github.com/bitcoin/bitcoin 

Move to the Bitcoin directory:

    
    
    cd bitcoin

Run autogen.sh script. It is used to generate build configuration files
required for compiling the Bitcoin core source code.

    
    
    ./autogen.sh

Setting variables to set Bitcoin core parameters for building, as well as
configuring the fuzzing flags.

    
    
    CC=clang CXX=clang++ ./configure --enable-fuzz --with-sanitizers=address,fuzzer,undefined

Let’s break down each option here:

  1. `CC=clang` and `CXX=clang++` : These environment variables are used to set the C and C++ compilers to use during the build process.
  2. `./configure` : Used to build configuration.
  3. `--enable-fuzz` : Enable fuzzing support in Bitcoin Core. Ensures necessary dependencies are included in the build.
  4. `--with-sanitizers=address,fuzzer,undefined` : Now these are settings for libFuzzer. This option enables specific sanitizers for detecting different types of errors. `address` is helps detect memory access errors such as out-of-bound, heap overflow, etc. `undefined` is used to detect undefined behavior in the code.
  5. `fuzzer` : with this flag Clang automatically links our program with the fuzzer library present in the Bitcoin core.

After setting all the configuration settings, now it's time to compile the
source code.

    
    
    make

`make` command is used to compile the C++ source code and create executable
programs. When you run _make,_ it reads the instruction present in the
`Makefile` present in Bitcoin Core source code and performs the necessary
compilation.

> Note: do not forget to `./configure` before running `make`

Once compiling is complete you are ready with the executable source code files
necessary to do fuzzing!

As I already mentioned earlier, we are going to fuzz the _Banman.cpp_ class.

[_src/test/fuzz/banman.cpp_](https://github.com/bitcoin/bitcoin/blob/master/src/test/fuzz/banman.cpp)
is the fuzz harness file for the Banman class. Running the following command
will fuzz this class using the libFuzzer:

    
    
    FUZZ=banman src/test/fuzz/fuzz

If your Fuzzing was successful, you will see something like this in your
terminal.

> Note: The fuzzing is intended to continue indefinitely until it finds an
> error, resulting in a system crash. As a result, it is recommended that you
> quit it after a certain period of time, depending on your needs.

<figure>
<img src="https://miro.medium.com/v2/resize:fit:1400/format:webp/1*WPMSmMOXSLDarqJlVMFtVA.png"/>
</figure>

Here’s a little description of what these terms in the image mean:

  1. NEW: this indicates that the fuzzing process has found a new (previously unseen) input that has led to the discovery of a new code path.
  2. REDUCE: This is when libFuzzer tries to simplify complex inputs by removing unnecessary data and simplifying the structure while preserving the ability to trigger the same error. It helps in analyzing and debugging by focusing on the essential elements of the input.

In case of libFuzzer encounters a _bad input_ , the fuzzing process halts and
it reports the type of error encountered. It generates _Crash files_ that
contain detailed information about the crash, such as the input that triggered
it, the stack trace, and other relevant diagnostic data.

This is pretty much what Fuzzing is and how it is implemented in Bitcoin Core
using libFuzzing. I really hope this is easy to understand. I’m also open to
all feedback and would be happy to modify the blog if any errors are found.

Thanks!

## Author

Ayush Singh. I’m a Summer of Bitcoin’23 intern working towards increasing fuzz
coverage over the wallet codebase in Bitcoin Core.

