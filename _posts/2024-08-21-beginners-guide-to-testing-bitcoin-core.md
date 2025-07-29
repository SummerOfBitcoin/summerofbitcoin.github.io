---
layout: post
title: "Beginner's Guide to Testing Bitcoin Core"
date: 2024-08-21
author: "Prabhat Verma"
categories: [Tutorials, Bitcoin Core]
---

With this article , I will be sharing how you can start testing in Bitcoin Core , specifically Unit testing and Functional testing .

### Setup

To start with Bitcoin core you should have a local copy of Bitcoin-core on your machine. I have a Linux machine so I will be sharing the steps for that , for other machines you can look [here](https://github.com/Prabhat1308/bitcoin/tree/master/doc#building)

To start with Bitcoin Core, you should have a local copy on your machine. Here are the steps for a Linux machine:

Even before starting this you need to have some dependencies installed in your machine . For that you can use the following commands in your terminal.

```plaintext
sudo apt-get install build-essential libtool autotools-dev automake pkg-config bsdmainutils python3
```

  
Fork the Bitcoin Core repository on GitHub.

1. Clone the repository to your local machine:
    
    ```plaintext
git clone git@github.com:YOURGITHUB/bitcoin.git
    ```
    
2. Navigate to the cloned directory and build Bitcoin Core:
    
    ```plaintext
cd bitcoin
git checkout 27.x
./autogen.sh
./configure --disable-bench --disable-fuzz-binary --enable-debug --without-gui --enable-suppress-external-warnings
make -j 8
make install 
    ```
    
    If there were no errors with the build process then you have built the repo successfully.  
    

### Units tests vs Functional Tests

Both types of tests are crucial for Bitcoin Core. Unit tests ensure that the written code functions correctly and works as expected. Functional tests check if different layers or sets of code can work together and operate smoothly. Bitcoin core uses python for the functional tests and C++ tests using libboost for writing unit tests.

### Running functional tests in bitcoin-core  

For this step you need to have python 3.9 or above installed in your machine .

To run the bitcoin-core functional tests , from the bitcoin directory write this command.  

```plaintext
test/functional/test_runner.py
```

To run extra tests you have to pass the --extended flag while running this command.

```plaintext
test/functional/test_runner.py --extended
```

You should get a similar response for the terminal .(some of the tests can be skipped because of various reasons. As long as there is no test failing , you should be good)

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1717965437620/fc5ff0ed-da88-4b29-8945-41b1856a0bda.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1717965689087/2fcc1f7a-72ec-4d18-bc4e-58caad9feb2b.png align="center")

### Running Unit tests in Bitcoin core

Unit tests are written using the Boost libraries , make sure you have their latest version installed on your machine .  
  
For running unit tests you can just run

```plaintext
make check -j 8
```

### Running Unit tests in Bitcoin core with debuggers

Both lldb and gdb are supported to be used as debuggers in Bitcoin core , For the purpose of this article , I will be using gdb.

For using gdb , it is essential that you build bitcoin-core without optimisations because some variables might have been optimised out and lead to not so informatory results.

```plaintext
cd bitcoin
make clean
./configure --disable-wallet --disable-fuzz -disable-fuzz-binary --enable-debug --with-gui=no CXXFLAGS="-O0 -ggdb3"
make -j 8
```

Now to use gdb , from bitcoin directory run the following command on your terminal

```plaintext
gdb src/test/test_bitcoin
```

After that , you can run test in from any test files. for eg. I want to run the ShouldFanoutToTest in txreconciliation_tests.cpp. In gdb , type this in your terminal

```plaintext
run --run_test=txreconciliation_tests/ShouldFanoutToTest
```

You should get a similar response on your screen.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1717966818046/7cd23fbf-478f-4b69-a457-0b70a9900ae6.png align="center")

This is it . Now you know everything to running unit and functional tests in Bitcoin-core.

```