---
layout: post
title: "Breaking Bitcoin: Can we derive the private key?"
author: Sivaram D
date: "2021-10-01 08:32:00 +0000"
categories:
  - "Cryptography"
description: "Let's try to derive the private key from a given public key!"
---

Many technologies like Tor, Bitcoin, and Email use *Asymmetric Cryptography* (also known as *Public Key Cryptography*) for secure data exchanges. If you are reading this blog, your browser has established a secure connection to the blog's server using this cryptography. The fundamental step for such secure connections is the generation of a private and public key pair.

It is known that we cannot derive the private key for a given public key, but there is no mathematical proof to back this up. It is unwise to believe something blindly. So let us write some code to derive the private key from a public key to evaluate the difficulty of this process. Before diving into the code, we need to understand a few basic concepts used in asymmetric cryptography and elliptic curve cryptography.

## Let's learn basic cryptography

> Feel free to skip this section if you know the basics of asymmetric cryptography and elliptic curves.

### Asymmetric Cryptography

It is a type of cryptography that is used widely for secure data transfers. It is an excellent fit for vast and growing environments with frequent data exchanges (i.e., basically the internet). So we use it for email security, web security, etc. It has the following properties.

* Each user generates two keys: *a public key* and *a private key*.
* Public key is mathematically derived from the private key (both keys together are called the key pair).
* The public key is made available to anyone. The private key is kept secret.
* Both keys are used to perform operations like encryption and decryption.

### Public and Private key

Although we use the term *"keys"* a lot, there are no physical keys involved here. Both public and private keys are enormous prime numbers. You can think of the public key as the username of your Twitter account and the private key as its password. Therefore, both of these together define your identity in the network.

I have mentioned before that your browser uses asymmetric encryption to connect to this blog's server. Let's try to find the public key used in this connection.

* **Step 1:** Click the lock button beside the URL of this page.  
  ![step1.jpg](https://cdn.hashnode.com/res/hashnode/image/upload/v1627988561252/zpFiHC6Va.jpeg)
* **Step 2:** Choose the Certificate option and navigate to details.  
  ![step2.jpg](https://cdn.hashnode.com/res/hashnode/image/upload/v1627988571348/6y5QkF0KJ.jpeg)
* **Step 3:** Now, you can view the server's public key.  
  ![step3.jpg](https://cdn.hashnode.com/res/hashnode/image/upload/v1627988577980/joJ0sxSIp.jpeg)

### Elliptic Curve Cryptography

One of the popular methods used to generate the private and public key pair is to use Elliptic Curve Cryptography. We follow the steps shown below to generate a public key from a private key.

* Select an elliptic curve.
* Choose a random point on the curve (Let's say ***G***).
* Add this point with itself a certain number of times (Let's say ***n*** times).
* Return ***n \* G***.

Here, ***G*** = Generator point, ***n*** = private key and ***n \* G*** = public key.  
Now we have covered the required concepts. If you want to learn more about Elliptic Curve Cryptography then, check out this fantastic [blog series](https://andrea.corbellini.name/2015/05/17/elliptic-curve-cryptography-a-gentle-introduction/?ref=blog.summerofbitcoin.org) by Andrea Corbellini.

## How to derive the private key?

We want to derive the private key from a known public key. We can mathematically write this as, given ***G*** and ***E***, find the value of ***n*** in the equation: ***E = n \* G***.  
where ***E*** = public key, ***n*** = private key, and ***G*** = generator point.  
This is called Elliptic Curve Discrete Logarithm Problem (ECDLP). This  [youtube playlist](https://www.youtube.com/watch?v=n41Z0c9Jm4Y&list=PL1xkDS1G9As7E_fPaLaFchq1a27I9a5tO&ref=blog.summerofbitcoin.org) by Leandro Junes covers the topic of the Discrete Logarithm Problem in detail.

This problem doesn't seem that difficult, right? After all, we know both ***G*** and ***E*** and we just need to find the value of ***n***. We can store ***G*** in a variable (Let's say `curr_key`) and keep incrementing this `curr_key` by the value `G` until it equals `E.` Then, the number of increments performed should be equal to the private key. The python implementation of this idea is below.

```python
def find_priv_key(G, pub_key):
    priv_key = 0 # no of increments performed
    curr_key = 0 
    while(curr_key != pub_key):
        curr_key += G # point addition on elliptic curves
        priv_key += 1
        if(curr_key == pub_key): 
            return priv_key
    return -1 # if no key exists

```

The time complexity of this algorithm is `O(n)` where `n = size of the private key.` So, did we derive the private key? Linear-time algorithms should be easy to compute, right? Great question! It will be easily computable if this was a leetcode problem, but that is not the case here. There are two mistakes in this approach.

* **Mistake 1:** The range of ***n*** is [1, 2^256). This range is so large that even if you have Google's computation ability (i.e., ~ millions of GPUs) and start your calculation at the creation of our universe, you still won't be able to finish your calculation by today. I would highly recommend you to watch [this video](https://www.youtube.com/watch?v=S9JGmA5_unY&t=3s&ref=blog.summerofbitcoin.org) by 3Blue1Brown. It will give you a good visual representation for the massive size of 2^256.
* **Mistake 2:** The value of ***x \* G*** starts to repeat after a certain ***x***. That is there can be ***x1*** and ***x2*** for which ***x1 \* G*** = ***x2 \* G***. This is because the addition that we perform here isn't a normal addition. It is a [point addition](https://andrea.corbellini.name/2015/05/17/elliptic-curve-cryptography-a-gentle-introduction/?ref=blog.summerofbitcoin.org#geometric-addition) on elliptic curves. You can relate this to the property of arithmetic modulo operation where 1 (mod 5) = 6 (mod 5). Hence, you need to know the range of possible private keys beforehand. This is generally not a problem since, the elliptic curve specification has this information in a parameter known as "group order".

## How are key pairs generated?

We realized that the linear time solution for private key derivation is computationally infeasible. Doesn't this also make the public key generation in linear time impossible? Yes, It is not feasible to generate a public key in linear time . Then how is a public key generated?

The algorithm used to generate the public key executes in `O(log n)` instead of `O(n),` making this generation process feasible. There does not exist an algorithm to derive the private key in logarithmic time. Hence, making the derivation process infeasible. This `O(log n)` generation process makes use of binary expansion of ***n*** to calculate ***n \* G***. You can find the implementation of this idea is below.

```python
def generate_pub_key(G, n):
    temp = G 
    pub_key = 0 # stores public key
	while(n > 0): # loops over the bits of n
        if(n&1): # add 2^(i) to answer if ith bit is 1
            pub_key += temp
			n = n >> 1 # equivalent to n = n/2
			temp = 2*temp
    return pub_key

```

## Closing note

The final verdict is that it is infeasible to generate a private key given the public key. The security of many technologies relies on this property.

If you want to explore similar topics, I recommend the [Computerphile](https://www.youtube.com/channel/UC9-y-6csu5WGm29I7JiwpnA?ref=blog.summerofbitcoin.org) youtube channel. It has many interesting videos like [Secret Key Exchange](https://www.youtube.com/watch?v=NmM9HA2MQGI&t=15s&pp=sAQA&ref=blog.summerofbitcoin.org), [SQL Injection Attack](https://www.youtube.com/watch?v=ciNHn38EyRc&t=48s&pp=sAQA&ref=blog.summerofbitcoin.org) and [Elliptic Curve Back Door](https://www.youtube.com/watch?v=nybVFJVXbww&t=380s&ref=blog.summerofbitcoin.org).

I started exploring this topic to understand the [secp256k1 library](https://github.com/bitcoin-core/secp256k1?ref=blog.summerofbitcoin.org). I am grateful to my mentor [Jesse Posner](https://twitter.com/jesseposner?ref=blog.summerofbitcoin.org) for encouraging me to write a blog on this topic. I am also grateful to [Adam Jonas](https://twitter.com/adamcjonas?ref=blog.summerofbitcoin.org), [Adi Shankara](https://twitter.com/adi_shankara_?ref=blog.summerofbitcoin.org), and [Caralie Chrisco](https://twitter.com/Caralie_C?ref=blog.summerofbitcoin.org) for providing me this opportunity to participate in the [Summer of Bitcoin 2021](https://summerofbitcoin.org/?ref=blog.summerofbitcoin.org).

## References

* [Intro to public-key cryptography](https://www.youtube.com/watch?v=GSIDS_lvRv4&ref=blog.summerofbitcoin.org)
* [Asymmetric Cryptography definition](https://www.sciencedirect.com/topics/computer-science/asymmetric-cryptography?ref=blog.summerofbitcoin.org)
* [Asymmetric Encryption use cases](https://cheapsslsecurity.com/blog/what-is-asymmetric-encryption-understand-with-simple-examples/?ref=blog.summerofbitcoin.org)
* [Programming Bitcoin Book](https://github.com/jimmysong/programmingbitcoin?ref=blog.summerofbitcoin.org)
