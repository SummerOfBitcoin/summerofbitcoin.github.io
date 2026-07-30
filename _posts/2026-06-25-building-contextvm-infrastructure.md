---
layout: post
title: "Shrinking SDKs and Shipping Bundles: My First 6 Weeks at ContextVM"
date: 2026-06-25
author: Abhay Gupta
categories: [Development, Open-Source, Stories]
image: ../assets/images/blog_content/2026-06-25-shrinking-sdks-shipping-bundles.jpg
---

Hello world! I am Abhay Gupta, and for the last six weeks, I have been hacking away as a Summer of Bitcoin contributor (Batch of 2026). My summer is entirely dedicated to the core infrastructure of ContextVM, a fascinating ecosystem built around decentralized application deployment and the Model Context Protocol (MCP).

My biggest takeaway so far is that building a protocol is only half the battle. The real magic is building the tools and developer experience that make people want to use it!

Here is a deep dive into what I have been building, breaking, and learning over the first half of my summer.

---

## The Mission: Making MCP Click

The Model Context Protocol (MCP) is great at connecting AI models to data sources. But deploying those servers seamlessly into a decentralized ecosystem requires a lot of heavy lifting. 

My project revolves around solving three tricky challenges:

*   **Interoperability:** How do we ensure that multiple servers offering similar capabilities can be discovered and swapped effortlessly?
*   **Distribution:** How do we safely package and execute these MCP servers over decentralized networks like Nostr?
*   **Developer Hygiene:** How do we keep the core SDK footprints impossibly light while ensuring the onboarding website feels premium and accessible?

---

## What I Have Built So Far

The first half of this summer has been an absolute sprint across multiple repositories. Here are the major technical milestones I am most proud of:

**The Magic of `.mcpb` Server Bundling**  
To make MCP servers easily distributable, I built out the `cvmi pack` subcommand for our CLI. This command takes a project directory and packages it into a tightly compressed `.mcpb` zip archive. I also upgraded the `cvmi serve` command so it can securely extract these bundles into a temporary directory, parse custom configuration metadata, and execute the server natively over the Nostr network. This work is currently proposed in an [open PR under review](https://github.com/ContextVM/cvmi/pull/4).

**Putting the SDK on a Diet**  
I conducted a major dependency audit of the official TypeScript SDK. By safely abstracting heavy validation libraries like Zod and Ajv out of standard dependencies and shifting them into optional peer dependencies, I successfully reduced the default installation footprint by roughly 3.5 MB without breaking existing implementations.

**Overhauling the Frontend Experience**  
A strong protocol needs a strong landing page. I completely redesigned the site’s primary header for a cleaner user experience and implemented dynamic SEO and Open Graph upgrades. I also paid down a mountain of technical debt by taking a monolithic 800-line Svelte landing page and componentizing it into maintainable, bite-sized sections.

**Formalizing the Future with CEP-15**  
I helped draft the Common Tool Schemas proposal (CEP-15). This specification leverages deterministic hashing and CEP-6 tags to allow clients to seamlessly discover and switch between different standardized service providers in a decentralized marketplace.

---

## The Technical Trenches: Zod, Zips, and Mistakes

Without a doubt, the most technically demanding challenge was engineering the `.mcpb` bundling support inside the CLI. 

The problem was safely extending Anthropic's official MCP specifications. We needed a way to embed ContextVM-specific configuration like encryption preferences into the manifest file without breaking standard MCP compatibility. I had to build rigorous Zod validation schemas for a newly introduced custom namespace, handle the intricacies of cross-platform zip compression in Node.js, and wire up an extraction sequence that meticulously wiped secure temporary directories when the server shut down.

Along the way, I learned how important backward compatibility and dependency sizes really are. 

My mentor helped me understand the difference between frontend bundle sizes and backend disk bloat. A frontend tool like Vite can easily remove unused code, but a backend developer running `npm install` still has to download the entire package. Moving heavy libraries to peer dependencies taught me that every kilobyte matters in infrastructure projects.

I also learned that Git staging can be unforgiving! Early on, I accidentally committed auto-generated build files by using `git add .`. Learning to carefully review my staging area and write clean, focused pull requests has really improved my engineering habits.

---

## What Is Next?

With the base infrastructure and distribution tooling stabilized, the second half of my summer is shifting gears entirely toward monetization. 

My primary goal for the final evaluation is to fully implement the CEP-8 Payment Gateway natively within the `cvmi` command-line interface. This integration will allow developers to effortlessly monetize their bundles by broadcasting and verifying Lightning Network payment requests over Nostr, turning independent servers into a seamless micro-economy. 

---

## Let's Connect!

I would love to connect with other developers, open-source enthusiasts, or anyone interested in Bitcoin and MCP!

*   **Website:** [abhayakg.me](https://abhayakg.me)
*   **Email:** [abhayakg123@gmail.com](mailto:abhayakg123@gmail.com)

**Check out the Code:**
*   **Website Redesign & Refactoring:** [ContextVM/contextvm-site#30](https://github.com/ContextVM/contextvm-site/pull/30) and [#28](https://github.com/ContextVM/contextvm-site/pull/28)
*   **SDK Dependency Optimization:** [ContextVM/mcp-sdk#3](https://github.com/ContextVM/mcp-sdk/pull/3)
*   **CLI Bundling:** [ContextVM/cvmi#4](https://github.com/ContextVM/cvmi/pull/4)
*   **CEP-15 (Common Tool Schemas):** [CEP-15 Specification](https://github.com/ContextVM/contextvm-docs/blob/master/src/content/docs/reference/ceps/cep-15.md)
