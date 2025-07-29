---
layout: post
title: "Midway Through Summer of Bitcoin: Reflections on Building with Coinswap and BDK"
date: 2024-07-16
author: "Divyansh Gupta"
categories: [Stories, Coinswap]
---

Hey everyone! Wow, it's already mid-summer 2024, and I’m here to share some reflections on my journey with the Summer of Bitcoin internship. It’s been a whirlwind so far—filled with coding challenges, inspiring chats, and plenty of learning. The most exciting part? I’m actually seeing the impact of my work on real-world projects! I’ve realized that having a vision for the future is crucial. And through this internship, I feel like I’m shaping that vision. So, let’s dive into what I’ve been up to and what’s next on the horizon!

But let's first take a look at my organization.

## **A Peek Into My Organization: Coinswap**

### **What’s Coinswap All About?**

Coinswap is a cool Rust project implementing the Maxwell-Belcher Coinswap Protocol. This protocol is a fancy twist on atomic swaps, using Hash Time-Locked Contracts (HTLCs) on Bitcoin. Our goal? To create minimal-viable binaries and libraries for this trustless, peer-to-peer protocol.

### **So, What’s the Maxwell-Belcher Coinswap Protocol?**

In a nutshell, CoinSwap is all about boosting Bitcoin privacy. It lets users swap coins in a way that hides ownership. Even though transactions look normal on the blockchain, the actual ownership is obscured, giving a major privacy boost.

### **How Does CoinSwap Improve Privacy?**

By swapping coins in a way that makes it nearly impossible to trace transactions back to the original owners, CoinSwap ensures your transactions blend seamlessly into the blockchain. This enhances privacy and makes Bitcoin more secure and fungible.

### **Want to Try It Out?**

You can get started with Coinswap by building the source code locally. Just a heads up—it currently only works on Linux/Mac, so Windows users might need to sit this one out for now. Fork and clone our repo, and you’re good to go!

### **How It Works**

Our integration tests are a great way to see CoinSwap in action. They show how the protocol works using just Rust and Cargo—no extra setup or Bitcoin Core configuration required. Check out our tests and see how CoinSwap boosts privacy and functionality!

### **Thinking About Contributing?**

We have plenty of good first issues up for grabs. Your contributions could help shape the future of privacy in Bitcoin transactions. Interested? Here’s the link to our repo.

## **Let’s Chat About My Project: BDK Wallet Infrastructure**

### **What’s the Project?**

I’m working on enhancing the Coinswap Wallet by integrating Bitcoin Dev Kit (BDK) chain interfaces. The aim is to make the wallet more secure, efficient, and standardized by leveraging BDK’s solutions and tackling existing vulnerabilities.

### **Work Done So Far**

#### **Tasks Completed**

**Deep Dive into BDK Codebase:** Thoroughly explored the BDK codebase, focusing on BDK Wallet APIs, BDK chain interfaces, and other internal structures. This exploration was crucial in understanding how the BDK framework operates and identifying potential challenges in adapting it for our Coinswap wallet.

**Key Insights:** Discovered several potential blockers related to integrating BDK with our custom Coinswap requirements. For example, differences in keychain management and descriptor handling necessitated innovative solutions to ensure compatibility.

**Analysis of Existing BDK-Based Wallets:** Investigated various BDK-based wallets, including the Mutiny Node developed for the Fedimint protocol. This study provided valuable insights into the architectural design of Bitcoin wallets and highlighted best practices for integrating BDK with different wallet functionalities.

**Key Takeaways:** Gained a deeper understanding of how existing implementations handle complex scenarios, which informed our approach to designing and implementing our Coinswap wallet.

**Development of Fundamental APIs:** Created essential APIs such as Wallet::init and Wallet::load. These APIs are foundational to the functionality of our wallet, enabling operations like wallet initialization and loading of wallet data.

**Testing and Validation:** Conducted extensive testing to ensure these APIs perform as expected under various conditions. This included verifying their integration with the broader BDK ecosystem and ensuring compatibility with our custom requirements.

### **Challenges Faced & Solutions**

**Custom Keychains vs. Default BDK Keychains:**

**Challenge:** BDK Wallet supports only external and internal keychains by default. Our project requires additional custom keychains to handle specific use cases, which necessitated a deeper dive into BDK’s chain interfaces and internal structures.

**Solution:** Investigated and implemented a custom Keychain enum to accommodate additional keychains. This new enum structure enables the creation of various keychains with unique attributes, such as Fidelity and SwapCoin, tailored to our specific needs.

```rust
pub enum Keychain {
    External,
    Internal,
    Fidelity { index: u32 },
    SwapCoin { index: u32 },
    Contract { index: u32 },
}
```

**Outcome:** This approach allowed us to manage and utilize custom keychains effectively, aligning with BDK’s existing structures while addressing our unique requirements.

**Descriptor and Miniscript Use:**

**Challenge:** Initially considered using raw scripts to avoid the complexity of descriptors and miniscripts. However, BDK’s reliance on descriptors made it necessary to integrate them despite the added complexity.

**Solution:** Adopted descriptors and miniscripts to ensure compatibility with BDK’s descriptor-based model. This decision was driven by the need to work within BDK’s framework and leverage its robust features.

**Outcome:** Integrated descriptors and miniscripts into our design, facilitating compatibility with BDK while maintaining the integrity of our custom keychains and their functionalities.

**Use of Rust-Miniscript:**

**Challenge:** Our custom keychains involve complex constructs like HTLCs and timelocks, which require specific policy definitions and miniscript representations. This complexity posed a challenge in aligning with Rust-Miniscript’s capabilities.

**Solution:** Utilized Rust-Miniscript to generate and manage relevant policies and miniscripts for our custom keychains. This allowed us to handle the intricacies of contracts and timelocks effectively.

**Outcome:** Successfully integrated Rust-Miniscript into our workflow, enabling precise control over contract and timelock implementations while ensuring compatibility with BDK.

### **What’s Next?**

After tackling a bunch of challenges and diving deep into the project, I’m feeling pretty confident about where we’re headed. We’ve figured out some solid techniques to make our Coinswap wallet compatible with BDK chain interfaces and have implemented the basic APIs. This really sets us up for the next phase of our work and gives a big boost to our speed and efficiency as we move forward.

As I look ahead, I’m also taking some time to reflect on the broader impact of the technology we’re working with. It’s amazing to think about how Bitcoin, the foundation of our project, has evolved and influenced so many aspects of technology and finance.

### **My Thoughts on Bitcoin**

**Foundational Concepts:** Bitcoin builds on existing cryptographic concepts like Blockchain and Peer-to-Peer Networks. What sets Bitcoin apart is its unique blend of these ideas to tackle the issues faced by earlier systems.

**Contributors’ Impact:** Bitcoin owes much of its success to the open-source contributors who worked on it selflessly. Their dedication and hard work have made Bitcoin what it is today and inspire me to keep contributing to open source.

So, that’s where I’m at with my Summer of Bitcoin journey so far. I’ve shared my thoughts on Bitcoin and what’s coming next, but I’d love to hear from you too. What are your thoughts on Bitcoin, open source contributions, or anything related to this space? Let’s keep the conversation going!