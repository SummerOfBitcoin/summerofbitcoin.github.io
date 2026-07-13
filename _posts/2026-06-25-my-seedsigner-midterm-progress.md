---
layout: post
title: "Alternative SeedSigner UIs Across Display Constraints and Accessibility Modes: My Midterm Progress"
date: 2026-06-25
author: Akhil Dhyani
categories: [Development, Open-Source, Design]
image: ../assets/images/blog_content/seedsigner-constrained-ui-midterm.png
---

## Who I am and what project I am working on
I am Akhil Dhyani, a third-year CSE undergraduate from IIT Jodhpur, and for my Summer of Bitcoin 2026 project, I am working with **SeedSigner**. SeedSigner is an open-source, air-gapped Bitcoin signing device that typically runs on a Raspberry Pi Zero with a 240x240 pixel LCD screen and a camera.

My project, **Alternative SeedSigner UIs Across Display Constraints and Accessibility Modes**, focuses on building an alternative UI runner that allows the entire SeedSigner software stack to run on vastly smaller, cheaper hardware displays (like 16x2 character LCDs and 128x32 OLEDs), along with an audio-based UX for visually impaired users.

## What problem my project is solving
I'm working to make Bitcoin hardware wallets radically cheaper! My project involves building a custom text-rendering engine that allows the SeedSigner software to run perfectly on ultra-cheap, $2 character displays. This lowers the hardware barrier to entry, helping anyone build a fully functional, air-gapped Bitcoin wallet. It also removes supply-chain bottlenecks by providing a hardware-agnostic text rendering approach.

### Hardware Comparison

| Feature | Original SeedSigner | Constrained UI Runner |
| :--- | :--- | :--- |
| **Primary Display** | 240x240 LCD (Waveshare) | 16x2 / 20x4 Character LCD |
| **Screen Cost** | ~$15 - $25 | ~$2 - $5 |
| **Accessibility** | Visual Only | Spatial Audio & Visual |
| **Input/Output** | Camera (QR Codes) | MicroSD |

## What I completed in the first six weeks
- **Bootstrapped the Constrained UI Runner Repository:** Validated low-level I2C hardware drivers for 16x2 LCD, 20x4 LCD, SSD1306 OLED, and passive buzzers.
- **Implemented the Core UI State Machine Engine:** Developed screen state and parser to ingest testing scenarios, manage cursor movement, handle scroll offsets, and enforce display boundaries. Achieved 100% passing unit tests for the core logic.
- **Developed Responsive Text Rendering Algorithms:** Created a unified `TextRenderer` that dynamically formats SeedSigner screens using two strategies: "Block Pagination" (for ultra-constrained 16x2 LCDs) and a "Sliding Window Viewport" (for 20x4 LCDs). 
- **Built the Interactive Desktop Simulator (Dual Runner):** Engineered a Tkinter-based desktop simulator that allows for rapid, hardware-free testing of the constrained UI logic using arrow keys, validating all interactive menus and text input flows.
- **Integrated Physical Hardware Drivers:** Wired the `TextRenderer` to physical LCD hardware proving that complex UI states map perfectly onto real-world character LCDs.
- **Implemented Interactive Keyboard & Entry Screens:** Designed and coded the text-based variants of the Keyboard Screens for complex data entry, including dice rolls, coin flips, and BIP85 numeric inputs.
- **Engineered Spatial Audio Navigation:** Integrated a PWM buzzer to provide spatial tonal navigation cues (varying pitch based on cursor position) and explicit event chimes (e.g., success chords, error buzzes) to improve accessibility.
- **Created an Automated HTML Verification Gallery:** Built a dynamic extraction proxy that intercepted the live LVGL software to generate a unified gallery, proving that our `TextRenderer` successfully maps and renders all the SeedSigner screens without crashing.
- **Finalized the Interactive Seed Mnemonic Entry Flow:** Implemented the complex dual-focus state machine for BIP39 word entry, allowing users to seamlessly type characters and toggle focus to a dynamic autocomplete suggestion list using only a 4-way D-Pad.

## The hardest problem I faced

**The Problem:** 
Translating the SeedSigner's highly detailed graphical menus into pure text was incredibly challenging. The original app was built for a 240x240 pixel screen, meaning it could dynamically resize fonts or scroll pixel-by-pixel to fit long text. 

We had to make this work on a 16x2 text display (which only fits 32 characters) without cutting off important warnings or breaking the user's understanding of where they were in the menu. 

Furthermore, extracting decoupled asynchronous UI threads, like Toast overlays, proved difficult, as our extraction hooks would often fire before the background threads finished rendering their notifications.

**What I Tried:** 
Initially, I tried a script that just chopped off any text longer than 16 characters, but this made critical Bitcoin settings unreadable. I then tried to make the text scroll horizontally like a digital billboard, but the hardware refreshed too slowly over I2C, resulting in a blurry mess. 

For the asynchronous Toasts, checking for `ToastOverlay` objects directly failed because they bypassed the main exception loop, and adding time delays caused race conditions in the test suite. 

I ultimately solved the rendering by writing two distinct math algorithms:
*   **Block Pagination**: Mathematically breaks menus into single-item "pages" for the smallest screens.
*   **Sliding Window**: Moves the menu up and down around a fixed cursor for slightly larger screens. 

For the Toasts, I utilized Python’s `inspect` module to walk backward up the execution stack at runtime, locate the generator's local variables, extract the pending `toast_thread` message string, and inject it into our payload just milliseconds before the system destroyed the memory frame.

**What I Learned:** 
I learned that porting software to highly constrained hardware is a data structure problem. You can't just shrink graphics; you have to extract the underlying semantic data (like the total number of items and the cursor's current index) and feed it into a completely new mathematical formula to generate a UI layout.

## What I learned about Bitcoin Open Source Development

What surprised me most about Bitcoin open-source development is the intense emphasis on security, upstream compatibility, and extreme accessibility. 

Coming into the project, I assumed I would simply rewrite parts of the core SeedSigner codebase to make it support text displays. Instead, I learned that in Bitcoin OSS, you avoid touching core cryptographic or business logic at all costs to prevent introducing vulnerabilities. I was surprised by how much engineering effort goes into building completely decoupled, non-invasive pipelines. 

Furthermore, I discovered that true success in this ecosystem is actually stripping away dependencies until the software can run securely on the absolute cheapest, most primitive hardware imaginable. 

Engineering a complex Bitcoin signing flow to work flawlessly on a two-dollar, 16x2 text LCD taught me that Bitcoin development isn't about building shiny apps, it’s about guaranteeing that self-sovereignty is affordable and accessible to literally anyone in the world, regardless of their access to modern technology.

## What I plan to finish before the final evaluation

In the remaining weeks leading up to the final evaluation, I will execute the hardware integration and polish phases of my implementation plan:

*   **Hardware Integration:** First, I will build the OLED rendering pipeline, deploying our verified text UI logic directly onto 128x32 OLED screens. Concurrently, I will build a secure MicroSD fallback pipeline allowing SeedSigner to mount SD cards, read unsigned PSBTs, and atomically export signed PSBTs when a camera is unavailable.

*   **Design Standards & Stretch Goals:** Following the core hardware integration, I will publish canonical Text UI and Audio UX design guides. I will then explore experimental hardware stretch goals, specifically writing new renderers for E-Paper (1.54" 200x200 partial refresh) and Nokia 5110 (PCD8544) displays, along with an advanced pre-recorded audio soundboard system for enhanced accessibility.

*   **Validation & Polish:** Finally, the project will conclude with intensive cross-target validation testing, running the entire SeedSigner flow interactively across every supported hardware display simultaneously. I will conduct memory profiling on the Raspberry Pi Zero to optimize I2C rendering performance before finalizing the automated test suite and recording the ultimate multi-display demonstration video for the August submission.

## Conclusion

Working on the Constrained UI Runner has been an incredible journey so far. I've learned that true open-source Bitcoin development is about ensuring that critical tools for financial self-sovereignty are accessible and affordable for everyone. I'm excited to dive into the hardware integration phase and finalize the rendering logic for even more displays.

**Want to help make Bitcoin hardware wallets more accessible?** Check out my project repository below to review the code, open an issue with your ideas, or test the simulator yourself. I'd love to hear your feedback!

## Links to my work
- [SeedSigner Constrained UI Runner Repository](https://github.com/aphrodoe/seedsigner-constrained-ui-runner)
- [Demo Video](https://x.com/KeithMukai/status/2069865959368671545)
