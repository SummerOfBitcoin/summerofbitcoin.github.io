---
layout: post
title: "Fuzzing Bitcoin: My Summer of Bitcoin Journey with BitcoinFuzz"
date: 2024-07-20
author: "Kartik Agarwala"
categories: [Stories, Bitcoin-Fuzz]
---

### INTRODUCTION

Hello all, my name is Kartik Agarwala (hax0kartik) and over the last few months, I have been working on a project called as bitcoinfuzz under the mentorship of Bruno Garcia. I am working on this project as a part of the Summer of Bitcoin program and in this blog post, I am going to talk in brief about my project and my progress so far.

### WHAT IS SUMMER OF BITCOIN?

Summer of Bitcoin is a global, online internship focused on bringing in new contributors into the bitcoin ecosystem. For those of you who are aware of GSoC, it is basically the same thing but for Bitcoin albeit, with a much more complicated application process. This year, there were over 5000 applicants from over 50 different countries and my application for bitcoinfuzz was accepted after several rounds of screening.

### WHAT ARE BITCOINFUZZ AND MY PROJECT?

Before I talk about bitcoinfuzz, it is important to know what fuzzing, coverage-guided fuzzing and differential-fuzzing are. Fuzzing is the “infinite-monkey-theorem” of testing techniques in which a program is subjected to malformed and manipulated inputs repeatedly. Coverage-guided fuzzing is a “smarter” fuzzing technique in which inputs are manipulated in such a way so that more code blocks are covered. Usually, fuzzing is useful in strengthening programs against complex inputs, however fuzzing might also be helpful in other conexts. Differential-fuzzing is one such variation of fuzzing in which same input is given to multiple programs and the differences in their outputs is observed.

Bitcoinfuzz is a open-source project which allows to differential-fuzz different bitcoin implementations and libraries. At the time of writing, bitcoinfuzz supports differential-fuzzing of BTCD, bitcoin-core and Rust-bitcoin. Under the hood, it uses libfuzzer, a coverage-guided fuzzing engine and has various “harnesses” which help target specfic functionality such as transaction deserailization, block deserialization, etc.

Under the SoB program, I sent a proposal to work on bitcoinfuzz and the major aim of my work was to add a harness to support fuzzing bitcoin network protocol messages. Additionally, when I started working on bitcoinfuzz, it only supported rust-bitcoin and bitcoin core and it was also my responsibility to add support for btcd.

### PROGRESS SO FAR

The very first thing that I worked on was to add support for btcd. The issue is that the harnesses are written in C++ while btcd is written in Golang and thus you need some kind of way to do interop between the two lanaguages. Fortunately, CGO makes it simple to do C/Golang interop. All I needed to do was to write a wrapper around the Go functions in btcd, and then use cgo to export the wrapper functions as C functions. This is also where I ran into issues with dynamic linking of libraries on macos. Due to some reason, Golang was not exporting the correct rpath for the dylib on macos, and I had to manually use the install_path_name tool as a workaround for this issue. Since I did not have a local macos machine, I quickly created a github CI script to test changes on a macos runner which also get merged after I polished it a bit. 😃

After some discussion with my mentor, I imported rust-bitcoin and btcd as git submodules and changed both the wrapper libraries to target these submodules instead. I am well aware of the fact that git modules are generally frowned upon, however, in our case, it was largely benefecial as it allows to target specific commits which might not be a part of a release yet(This itself is beneficial when a bug might exist in a release but have been fixed in a latest commit).

The next major thing that I did was to add instrumentation for rust-bitcoin and btcd. Bitcoinfuzz uses coverage-guided fuzzing in the background and this requires instrumented binaries so that the fuzzer can learn more about the various code paths and coverage. Till this point, bitcoinfuzz was only instrumenting the bitcoin-core code and was missing instrumentation for btcd and rust-bitcoin which was counter-intuitive. I searched the internet about related terms and while they were many posts which talked about fuzzing go code, I did not find any material which talked about only adding the instrumentation. This is the point where I decided to go through golang’s source code and I came to realization that golang compiler would add the instrumentaion if proper tags and flags are passed (-tags=libfuzzer, -gcflags=all=-d=libfuzzer). I should note that it is slightly more complicated and I’ll probably make a blog post about it soon. Adding instrumentation for rust-bitcoin was much easier, I only needed to pass the correct llvm-args flags to the rust compiler. In the process, I also started linking the libraries statically as opposed to dynmically and while this inflated the size by quite a bit, we are only interested in making the fuzzing process better, so we can ignore the size. In the end, I was hapy with the results as the PC table size grew by almost ~3x!

Ever since I imported rust-bitcoin and btcd as submodules, I had wondered if it was possible to do the same for bitcoin-core as well. I wanted to do this as there were many benefits. With the current code, we were manually importing the required files one-by-one from bitcoin-core and this was really hectic due to dependencies. One file would require some other file, which itself would depend on another file and this continued for a bit. Furthermore, to complete my major aim, I would need to import the net code from bitcoin-core and I wanted to avoid that. The problem with importing bitcoin-core as a submodule is that bitcoin-core is not a library, i.e, it simply does not give you a static/dynamic library you can link against. Initially, I thought I could get past this issue by grepping for cpp files and including them, and I couldn’t be more wrong as I would end up including files where int main() was defined and then the linker would complain about multiple main() symbols. I tried getting past this issue by changing the grep flags a little to exclude files where int main was found but then it would start giving some other issues. On re-reading the bitcoin-core makefile, I came to know that recently, bitcoin core has started work on something they call the libbitcoinkernel which gives a library that has all the core functionality of bitcoin. However, the related documentation also said that it was unstable and not ready for use. Nonetheless, I set the required flags for configure and then ran make -j and linoing against the generated library worked! I discussed this with Bruno, and he told me that this change was acceptable. 😃 So after polishing the code a bit, my changes were merged.

I have a history with AFL (American Fuzzy Lop) and I was curious whether I could make switch out libfuzzer with afl in bitcoinfuzz. Luckily, bitcoin core’s fuzz.cpp already supported afl, libfuzzer and honggfuzz. Knowing this, I made our harnesses compatible with fuzz.cpp and then simply linked to this file instead of fuzzer.cpp(equivalent of fuzz.cpp in our project). To my surprise, it compiled on the first try. Next I compiled the AFL++ project and used the afl-clang-lto++/afl-clang-lto compilers and it almost worked. The problem was that afl’s instrumenattion is different from libfuzzer and the go wrapper library was not be able to find some symbols while final linking. I looked around the net a bit, but I did not find anything conclusive, so for now, only rust-bitcoin and bitcoin-core are instrumented when running in afl mode. Honggfuzz, on the other hand, works without issues once the correct compilers are set.

Aside from this, I have worked on some minor things, which are probably not worth mentioning in a blogpost. If you want to look at all the changes I have made, you can look at my PR(s) which have been merged.

### CONCLUSION

I would like to start this section by thanking my mentor, Bruno Garcia, who has helped me immensely in the last few months. None of this would be possible without his help!

I cannot underestimate the amount of things I’ve learned about bitcoin and fuzzing in general while working on this project and during my tenure as a mentee, we have discovered multiple bugs/inconsistencies in btcd and rust-bitcoin (close to ~10). We’ve been working on reporting these issues to the required parties and it showcases how well differential-fuzzing performs in this context. (If I get the time, I would also make a blogpost to discuss some of our findings.)

With the contributions I have made so far, I believe I have completed about ~80-90% of the work required to implement my original bitcoin network protocol message harness and I am happy with my progress.

Thank you for reading my blog post! 😁
