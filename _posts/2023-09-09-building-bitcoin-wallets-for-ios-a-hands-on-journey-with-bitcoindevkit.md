---
layout: post
title: "Let's break new ground: Create an iOS Demo wallet with BDK"
date: 2023-09-09
author: Temiloluwa Yusuf
categories: ['Tutorials', 'Wallets']
image: ../assets/images/blog_content/2023-09-09-building-bitcoin-wallets-for-ios-a-hands-on-journey-with-bitcoindevkit_daa1817d.jpg
---

In this article I will share how I created an iOS Demo wallet with Swift,
SwiftUI and the BitcoinDevKit (BDK) framework.

Marvellous May turned out to be an exciting month as I received a
congratulatory mail for being accepted as a 2023 summer of bitcoin intern
contributing to the “Create Swift iOS Demo wallet project”. As a junior iOS
Developer I have always wanted to gain first class experience writing iOS apps
and developing open source software, and the 2023 summer of bitcoin internship
made my summer an interesting one to remember.

I got my summer underway with an onboarding meeting with my mentors, Steve
Myers and Matthew Ramsden. I was delighted to speak with them about what we
are working on this summer. The meeting was engaging as it lasted for two
hours. We talked about everything bitcoin, iOS and the deliverables of the
project for the summer. I was assigned my first task during the meeting and I
was delighted to set up my dev environment.

# Setting up development environment

> **Step 1** **:** Install rust via the [rustup](https://rustup.rs/) tool.
    
    
    curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh

> **Step 2** : Install bitcoind and bitcoin-cli locally.
    
    
    brew install bitcoin

> Step 3 : [Setup local regtest bitcoind
> wallet](https://gist.github.com/notmandatory/a7cade3468e90c699037292123a1ca1a)

After a week of the onboarding and setting up my dev environment, I was added
to the project repo on GitHub by my mentor (Matthew Ramsden) and my second
task was to create the project readme with the markdown language. It was an
easy task so I did it in a jiffy and got my first pull request (PR) merged.

# Create readme with markdown language

## [BDKSwiftExampleWallet/README.md at main · reez/BDKSwiftExampleWalletBDK
Swift Example Wallet. Contribute to reez/BDKSwiftExampleWallet development by
creating an account on
GitHub.github.com](https://github.com/reez/BDKSwiftExampleWallet/blob/main/README.md?source=post_page
-----c24689f30376---------------------------------------)

The next task assigned to me was to create the demo wallet onboarding screen
and it was interesting to work on. The bitcoin dev kit and walletUI frameworks
were seamless to use so developing the screen was a breeze. What made the
whole development process interesting was my mentor’s comments in my PR, lol
he left so many comments in my PR which made me work harder and better. I was
thrilled I was corrected when I made mistakes. Here is the link to the code on
GitHub.

# Creating the Onboarding screen

## [BDKSwiftExampleWallet/BDKSwiftExampleWallet/View/OnboardingView.swift at
main ·…BDK Swift Example Wallet. Contribute to reez/BDKSwiftExampleWallet
development by creating an account on
GitHub.github.com](https://github.com/reez/BDKSwiftExampleWallet/blob/main/BDKSwiftExampleWallet/View/OnboardingView.swift?source=post_page
-----c24689f30376---------------------------------------)

<figure>
<img src="https://miro.medium.com/v2/resize:fit:1400/format:webp/1*Ncci2GkkFJ-cPqqy6WbmlA.jpeg"/>
</figure>

**OnboardingView in Dark mode**

# Creating the Wallet View

After finishing the onboarding screen of the wallet, I was assigned to build
the wallet view of the project which was a very interesting view to build. It
was my first time implementing a design done by a community of professionals.
The wallet view was interesting to work on because it was technical, I love
working on technical views as an iOS developer.

The Bitcoin design guide was so easy to follow so it sped up my dev process.
Here is the link to the code on GitHub.

## [BDKSwiftExampleWallet/BDKSwiftExampleWallet/View/WalletView.swift at main
·…BDK Swift Example Wallet. Contribute to reez/BDKSwiftExampleWallet
development by creating an account on
GitHub.github.com](https://github.com/reez/BDKSwiftExampleWallet/blob/main/BDKSwiftExampleWallet/View/WalletView.swift?source=post_page
-----c24689f30376---------------------------------------)

<figure>
<img src="https://miro.medium.com/v2/resize:fit:1400/format:webp/1*dmbNkD5D-u45r44go_cf0g.png"/>
</figure>

**WalletView in Dark mode**

# “School of thought”

Something to keep in mind is that building a wallet like this has some caveats
and developers must be ready to face them. I was faced with a lot of version
control ( git ) problems for the wallet and it took me sometime to get them
fixed. The 3 way merge may be frustrating at first but it is exciting to use
once you wrap your head around it. Another problem I ran into was changing the
repo from private to public ha ha ha, even git itself got lost at this point.
My commits and PR couldn’t go to the upstream main so I had to fork the
project from my mentor’s repository again.

# Conclusion

Building an iOS App is great but building a bitcoin iOS app is greater. There
are so many high quality features coming to the wallet and more complex
screens still to be added. That will be all for this blog post.

