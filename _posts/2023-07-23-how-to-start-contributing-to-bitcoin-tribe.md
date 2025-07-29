---
layout: post
title: "How to start contributing to Bitcoin Tribe"
date: 2023-07-23
author: Raihan Khan
categories: ['Stories', 'Wallets']
---

## Prerequisites

Make sure you have the following installed:

  * [Node](http://nodejs.org/en) v10 and above
  * [Yarn](https://yarnpkg.com/lang/en/)
  * [CocoaPods](https://cocoapods.org/) (in the case of iOS)
  * [Xcode](https://developer.apple.com/xcode/) (in the case of iOS)
  * [Android Studio](https://developer.android.com/studio)

# Setting up Bitcoin Tribe Locally

  1. Head over to the [GitHub Repo](https://github.com/bithyve/bitcointribe) of Tribe and fork the repository.
  2. Clone the forked repository.

    
    
    git clone https://github.com/bithyve/bitcointribe.git  
    cd bitcointribe

3\. Install the required dependencies.

    
    
    yarn install

## Configuring Environment variables

Copy the contents on `.env.example` to a new `.env` file.

    
    
    cp .env.example .env

# Building and Running Bitcoin Tribe

## Start the React Native Server:

    
    
    npm start

<figure>
<img src="https://miro.medium.com/v2/resize:fit:1400/format:webp/1*dmbNkD5D-u45r44go_cf0g.png"/>
</figure>

Your terminal should look something like this once the server is up and
running!

# Running the Wallet on the Simulator

## iOS

    
    
    yarn run ios --simulator="iPhone 14 Pro"

Change **iPhone 14 Pro** to the simulator of your choice.

<figure>
<img src="https://miro.medium.com/v2/resize:fit:1400/format:webp/1*fP7qLtYQ5AxZMcqist_Q5A.png"/>
</figure>

Alternatively, XCode can also be used to build and run in a simulator.

<figure>
<img src="https://miro.medium.com/v2/resize:fit:1400/format:webp/1*yq61z3T20R6hKm29XU44kQ.png"/>
</figure>

## Android

    
    
    yarn android or yarn androidDevelopmentDebug

Alternatively, Android Studio can also be used to build and run in a
simulator.

> **Now you can start contributing to the open-source repository of Bitcoin
> Tribe and make an impact in bitcoin space!**

