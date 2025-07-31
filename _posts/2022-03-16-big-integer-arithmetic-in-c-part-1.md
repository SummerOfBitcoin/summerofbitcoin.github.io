---
layout: post
title: Big Integer Arithmetic in C - Part 1
author: Sivaram D
date: "2022-03-16 08:37:00 +0000"
categories:
  - "Tutorials"
description: "Let's learn how to add large numbers that cannot be stored in a standard integer type provided by the C language."
---

In cryptography, you typically work with large numbers to prevent the adversaries from brute-forcing a solution. If you are working with Python or Java, you are in luck. They can handle huge numbers by default.

Python supports a `bignum` data type that can work with arbitrarily large numbers, and Java has a `BigInteger` Class. For C, you have to implement a code that can handle arithmetic operations on big integers or use a library that provides this feature.

> **Note:** This article references 256-bit integers for big numbers, but the concepts discussed here can be easily extended to 128-bit or 512-bit integers.

## The problem with big integers

Before diving into implementation details, let's first understand why C does not support big integers by default.

According to [C89 standard](http://port70.net/~nsz/c/c89/c89-draft.html?ref=blog.summerofbitcoin.org#3.1.2.5), there are five standard integer types:

1. `signed char` (8-bit)
2. `short int` (16-bit)
3. `int` (16-bit or 32-bit)
4. `long int` (32-bit)
5. `long long int` (64-bit or 32-bit)

We can clearly see that there is no mention of a 256-bit integer data type.

C is a relatively "low-level" language so, the way it manipulates data is very similar to that of a computer processor. The processors manage data using the ***registers*** (tiny, high-speed storage units). These registers are typically 8-bit, 16-bit, 32-bit, 64-bit, 128-bit, or 256-bit in size.

Wait a minute! There is a 256-bit register? Yes, the `AVX` register on Intel and AMD microprocessor is an excellent example of this. Then, why don't we use this register to store a 256-bit integer? Because the Arithmetic Logic Units (ALU) in a microprocessor cannot perform arithmetic operations on 256-bit registers (only on 8-bit, 16-bit, 32-bit, 64-bit).

Hence, C can't leverage the `AVX` register to support big integers. Now that we established why C doesn't have big integers data types let's implement the code to overcome this issue.

## Representing a 256-bit integer

We realized that we couldn't fit a 256-bit integer in a single data type provided by C. Therefore, the only option is to break it into chunks of smaller sizes that can fit into a single data type. These chunks are usually called **limbs**.

For example, we can split a 256-bit integer into four 64-bit limbs and store them using the `uint64_t` data type.

> **Note:** If you want your code to be highly portable then, it is recommended to use eight 32-bit limbs to represent a 256-bit number since [some microcontroller compilers](https://www.microchip.com/forums/m908450.aspx?ref=blog.summerofbitcoin.org) will not support `unit64_t`.

Consider the following 256-bit unsigned integer (hexadecimal form):

`79BE667 EF9DCBBAC 55A06295 CE870B07 029BFCDB 2DCE28D9 59F2815B 16F81798`

Breaking this into 4 x 64-bit limbs will look like:

* Limb 1: `59F2815B 16F81798` *(least significant)*
* Limb 2: `029BFCDB 2DCE28D9`
* Limb 3: `55A06295 CE870B07`
* Limb 4: `79BE667E F9DCBBAC` *(most significant)*
{% raw %}
```c
// C struct for storing 256-bit unsigned integer
typedef struct {
    uint64_t d[4];
} big_int256;

// macro to create a big_int256
#define GET_BIG_INT256(d7, d6, d5, d4, d3, d2, d1, d0) {{
    (d0) | (((uint64_t)(d1)) << 32),
    (d2) | (((uint64_t)(d3)) << 32),
    (d4) | (((uint64_t)(d5)) << 32),
    (d6) | (((uint64_t)(d7)) << 32)
}}
```
{% endraw %}

This form of a 256-bit integer is called a **\( 2^{64} \) radix** representation (due to 64-bit limbs), and this is not the only way to represent a 256-bit integer.

## Adding two 256-bit integers

Since we are implementing a custom integer representation, we also need to take care of the arithmetic operations, unlike a `unit64_t` where C takes care of all these operations by default.

For adding two `big_int256` (our user-defined struct), we can add each limb separately from least significant to most significant while transferring the carry to the next limb (if generated).

```c
big_int256* add (big_int256 *a, big_int256 *b) {
    big_int256 *r = malloc(4 * sizeof(big_int256)); //stores the result
    int carry = 0;

    // add least significant limb first
    r->d[0] = a->d[0] + b->d[0];
    carry = (r->d[0] < a->d[0]);

    // add remaining limbs + carry from prev limb
    for (int i = 1; i <= 3; i++) {
        r->d[i] = a->d[i] + b->d[i] + carry;
        carry = (r->d[i] < a->d[i]);
    }

    /* can also store this result in an additional argument instead 
    of printing an error msg */
    if (carry) {
        fprintf(stderr, "overflow occurred!!");
    }

    return r;
}
```

Is this the most optimal `add` function? This should be, right? since adding two numbers is a fundamental operation. Unfortunately, this is not the most optimal solution. Indeed, we can't optimize this function in terms of the algorithm, but we can cleverly modify the usage of `carry` to generate the most optimal code. We will see how to do this in part 2 of this blog series.

## Conclusion

We have seen how to handle big integers in C and why C does not support it by default. You can also take a look at the source code of [libgmp](https://gmplib.org/?ref=blog.summerofbitcoin.org) and [libsecp256k1](https://github.com/bitcoin-core/secp256k1?ref=blog.summerofbitcoin.org) (C libraries) to see how they handle big integers.

Before I finish this article, I will leave you with the following questions to ponder:

* How `carry = (r->d[i] < a->d[i])` calculates the carry? Is `r->d[i] < b->d[i]` or `UNIT64_MAX - a->d[i] < b-> d[i]` a valid alternative?
* How will the optimal `add` function look like?

> **Hint:** Think about the `add` and `adc` operations used when our code is converted to an assembly file (during compilation).

## Reference

* [Efficient implementation of finite field arithmetic](https://cryptojedi.org/peter/data/pairing-20131122.pdf?ref=blog.summerofbitcoin.org)
* [C89 standard draft](http://port70.net/~nsz/c/c89/c89-draft.html?ref=blog.summerofbitcoin.org)
* [Inconviniences of using `unit64_t`](https://stackoverflow.com/questions/64702944/inconveniences-of-using-uint64-t?ref=blog.summerofbitcoin.org)
* [Large numbers in python](https://stackoverflow.com/questions/538551/handling-very-large-numbers-in-python?ref=blog.summerofbitcoin.org)
