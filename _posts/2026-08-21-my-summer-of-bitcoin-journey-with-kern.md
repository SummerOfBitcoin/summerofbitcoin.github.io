---
layout: post
title: "My Summer of Bitcoin Journey with Kern"
date: 2026-08-21
author: Subinita Ray
categories: [Design, Wallets, Stories]
image: ../assets/images/blog_content/2026-08-21-kern-pin-delay-arc.png
---

<div align="center" style="margin: 4px 0 24px;">
<span style="display:inline-block; background:#fff; border:1px solid #ff6600; color:#ff6600; font-family: ui-monospace, 'SF Mono', Menlo, Consolas, monospace; font-size:13px; font-weight:600; letter-spacing:0.5px; padding:10px 22px; border-radius:999px;">Congratulations! You are accepted in the Summer of Bitcoin Programme</span>
</div>

<div style="background:#fcfcfc; border:1px solid #e5e5e7; border-left:4px solid #ff6600; border-radius:6px; padding:24px 28px; margin:0 0 20px;">

<p style="margin:0 0 16px; color:#222; line-height:1.7;">When I was selected for Summer of Bitcoin, to be very honest I couldn't believe that I could crack a Global open source. My story highlights me from being under confident to a journey of self belief and a realisation I can do that too!</p>

<p style="margin:0 0 16px; color:#222; line-height:1.7;">I was always interested in UI/UX and something that spoke the language of design. I own a brand named OItijhya designs and I have delivered 37+ products all across.</p>

<p style="margin:0; color:#222; line-height:1.7;">I was selected for <a href="https://github.com/odudex/Kern" style="color:#ff6600;">KERN</a> - Icon and Visual Language System and then came the most difficult part. My experience limited me to Figma, canva and basic Frontend development using React framework but at Kern it demanded LVGL and C knowledge. I was initially fearful but later I realised Knowledge is meant to be expanded and never aimed to limit someone's ability to implement meaningful changes.</p>

</div>

<div style="background:#fcfcfc; border:1px solid #e5e5e7; border-left:4px solid #ff6600; border-radius:6px; padding:20px 24px; margin:0 0 20px;">
<div style="font-family: ui-monospace, 'SF Mono', Menlo, Consolas, monospace; font-size:11px; font-weight:600; letter-spacing:2px; color:#ff6600; margin-bottom:10px;">PROJECT SNAPSHOT</div>
<p style="margin:0; color:#222; font-style:italic; line-height:1.7;">Here comes a brief introduction about my project — <strong style="font-style:normal;">KERN</strong> represents a pioneering open-source initiative, serving as an air-gapped signing firmware designed specifically for Bitcoin on ESP32-P4 touchscreen platforms. It features a meticulously crafted LVGL user interface that strictly avoids radio communication, relying exclusively on QR codes and SD cards for data input and output.</p>
</div>

<div style="background:#fcfcfc; border:1px solid #e5e5e7; border-left:4px solid #ff6600; border-radius:6px; padding:24px 28px; margin:0 0 8px;">
<p style="margin:0; color:#222; line-height:1.7;">I am really thankful to Adi Shankara for allowing me this wonderful opportunity and my mentor <a href="https://github.com/odudex" style="color:#ff6600;">Odudex</a> who has been kindly patient with me giving me broader perspective on my PRs, allowing me to make mistakes and learn from them and ideate freely with an open canvas which is named Kern.</p>
</div>

<div align="center" style="display:flex; align-items:center; justify-content:center; gap:10px; margin: 44px 0 2px;">
<svg viewBox="0 0 200 200" width="26" height="26" xmlns="http://www.w3.org/2000/svg">
  <circle cx="100" cy="100" r="88.5" fill="none" stroke="#ff6600" stroke-width="4"/>
  <circle cx="100" cy="100" r="52" fill="none" stroke="#ff6600" stroke-width="8"/>
  <circle cx="100" cy="100" r="29" fill="#ff6600"/>
</svg>
<h2 style="margin:0; font-family: ui-monospace, 'SF Mono', Menlo, Consolas, monospace; letter-spacing: 1px;">What I built that got Shipped?</h2>
</div>

<div align="center">
<div style="width:2px; height:28px; background:#ff6600; margin: 4px auto 22px;"></div>
</div>

My very first PR —

| PR | What it solves |
|---|---|
| [**#31** — battery: replace text-only percentage with icon family](https://github.com/odudex/Kern/pull/31) | Replaces text-only battery percentage with icons for clearer, more compact battery state display. |
| [**#77** — feat(ui): add icons, semantic colours, and responsive flex layout](https://github.com/odudex/Kern/pull/77) | Enhances UI with icons, semantic colours and responsive layout for better presentation and accessibility. |
| [**#84** — docs: add UI design guidelines for colour, icons, typography and dialogs](https://github.com/odudex/Kern/pull/84) | *(duplicate/docs variant)* supplies UI design guidelines for consistent styling. |
| [**#96** — Feat/pin delay arc dots](https://github.com/odudex/Kern/pull/96) | Provides a visual "delay" indicator (arc dots) for PIN input timing. |
| [**#101** — feat(backup): color-coded numbered rows for mnemonic word list](https://github.com/odudex/Kern/pull/101) | Makes mnemonic word lists easier to read by adding color-coded, numbered rows. |
| [**#113** — fix(capture_entropy): add missing back button for cancel path](https://github.com/odudex/Kern/pull/113) | Restores a missing back button on the cancel path so users can navigate back. |
| [**#115** — feat(pin): add step progress bar to PIN setup flow](https://github.com/odudex/Kern/pull/115) | Adds a progress indicator so users know which step they're on during PIN setup. |
| [**#126** — feat(ui): redesign login screen as a 2x2 card grid with hover glow](https://github.com/odudex/Kern/pull/126) | Replaces the old login layout with a 2x2 card grid that adds hover glow for better visual. |
| [**#128** — fix(ui): make textarea border orange](https://github.com/odudex/Kern/pull/128) | Fixes inconsistent textarea styling by applying the expected orange border. |
| [**#133** — feat(ui): highlight primary action keys on button-matrix keypads](https://github.com/odudex/Kern/pull/133) | Improves usability by visually emphasising primary action keys on button-matrix keypads. |
| [**#146** — fix(ui): mask passphrase entry like every other secret field](https://github.com/odudex/Kern/pull/146) | Prevents passphrases from being shown in plaintext by masking the passphrase input. |

<div align="center" style="display:flex; align-items:center; justify-content:center; gap:10px; margin: 44px 0 2px;">
<svg viewBox="0 0 200 200" width="26" height="26" xmlns="http://www.w3.org/2000/svg">
  <circle cx="100" cy="100" r="88.5" fill="none" stroke="#ff6600" stroke-width="4"/>
  <circle cx="100" cy="100" r="52" fill="none" stroke="#ff6600" stroke-width="8"/>
  <circle cx="100" cy="100" r="29" fill="#ff6600"/>
</svg>
<h2 style="margin:0; font-family: ui-monospace, 'SF Mono', Menlo, Consolas, monospace; letter-spacing: 1px;">In this Process the major technical decisions that I took were</h2>
</div>

<div align="center">
<div style="width:2px; height:28px; background:#ff6600; margin: 4px auto 22px;"></div>
</div>

<h3 align="center" style="color:#ff6600; font-family: ui-monospace, 'SF Mono', Menlo, Consolas, monospace; font-size:14px; letter-spacing:1.5px; text-transform:uppercase; margin-bottom:16px;">Login screen unfilled cards redesign</h3>

<div align="center" style="border:1px solid #e5e5e7; border-radius:8px; padding:16px; margin:0 0 32px;">
<img src="../assets/images/blog_content/2026-08-21-kern-login-screen-redesign.png" alt="Login screen redesign — present screen, suggestion for the resting state, and resting/hover states" width="700">
</div>

<h3 align="center" style="color:#ff6600; font-family: ui-monospace, 'SF Mono', Menlo, Consolas, monospace; font-size:14px; letter-spacing:1.5px; text-transform:uppercase; margin-bottom:16px;">Keyboard Redesign</h3>

<div align="center" style="border:1px solid #e5e5e7; border-radius:8px; padding:16px; margin:0 0 12px;">
<img src="../assets/images/blog_content/2026-08-21-kern-keyboard-before.png" alt="Keyboard before redesign" width="260">
</div>

<div align="center" style="border:1px solid #e5e5e7; border-radius:8px; padding:16px; margin:0 0 32px;">
<img src="../assets/images/blog_content/2026-08-21-kern-keyboard-after-numeric.png" alt="Numeric keypad after redesign, with highlighted primary action keys" width="330">
<img src="../assets/images/blog_content/2026-08-21-kern-keyboard-after-qwerty.png" alt="QWERTY keypad after redesign, with highlighted primary action keys" width="330">
</div>

<h3 align="center" style="color:#ff6600; font-family: ui-monospace, 'SF Mono', Menlo, Consolas, monospace; font-size:14px; letter-spacing:1.5px; text-transform:uppercase; margin-bottom:16px;">Pin Delay arc Dots</h3>

<div align="center" style="border:1px solid #e5e5e7; border-radius:8px; padding:16px; margin:0 0 8px;">
<img src="../assets/images/blog_content/2026-08-21-kern-pin-delay-arc.png" alt="Wrong PIN screen showing the delay arc countdown and attempt tracker" width="700">
</div>

<div align="center" style="display:flex; align-items:center; justify-content:center; gap:10px; margin: 48px 0 2px;">
<svg viewBox="0 0 200 200" width="26" height="26" xmlns="http://www.w3.org/2000/svg">
  <circle cx="100" cy="100" r="88.5" fill="none" stroke="#ff6600" stroke-width="4"/>
  <circle cx="100" cy="100" r="52" fill="none" stroke="#ff6600" stroke-width="8"/>
  <circle cx="100" cy="100" r="29" fill="#ff6600"/>
</svg>
<h2 style="margin:0; font-family: ui-monospace, 'SF Mono', Menlo, Consolas, monospace; letter-spacing: 1px;">Reflections</h2>
</div>

<div align="center">
<div style="width:2px; height:28px; background:#ff6600; margin: 4px auto 22px;"></div>
</div>

<div style="background:#fcfcfc; border:1px solid #e5e5e7; border-left:4px solid #ff6600; border-radius:6px; padding:20px 24px; margin:0 0 16px;">
<div style="font-family: ui-monospace, 'SF Mono', Menlo, Consolas, monospace; font-size:11px; font-weight:600; letter-spacing:2px; color:#ff6600; margin-bottom:10px;">ON KERN</div>
<p style="margin:0; color:#222; font-style:italic; line-height:1.65;">Kern represents a pioneering open-source initiative, serving as an air-gapped signing firmware designed specifically for Bitcoin that physically lacks the capacity to connect to any network. While it remains a research-oriented endeavor and not yet a finalized consumer product, it provides a unique platform for experiential learning and testnet operations within the open-source community. The project maintains a rigorous threat model by implementing essential security features like encrypted storage and masked secrets, ensuring that user trust is built upon a foundation of verifiable hardware-based confirmations rather than blind reliance.</p>
</div>

<div style="background:#fcfcfc; border:1px solid #e5e5e7; border-left:4px solid #ff6600; border-radius:6px; padding:20px 24px; margin:0 0 16px;">
<div style="font-family: ui-monospace, 'SF Mono', Menlo, Consolas, monospace; font-size:11px; font-weight:600; letter-spacing:2px; color:#ff6600; margin-bottom:10px;">WHAT I LEARNT</div>
<p style="margin:0 0 14px; color:#222; line-height:1.65;">The things that I learnt is how to implement meaningful changes, adapt good testing practices before actually concluding the implementation. To be continuously involved in feedback for better execution and most important do a lot of research and be able to justify the context of changes and the problem the change will actually solve….</p>
<p style="margin:0; color:#222; line-height:1.65;">Coming from Figma and React, LVGL was a real shift — there's no DOM or CSS, just a C widget tree you build and style directly, with flex layouts, states (hovered/pressed/focused) and event callbacks all wired by hand. The Kern desktop simulator became my actual design tool: build with CMake, run it in an SDL2 window on my laptop, and see every icon, colour and hover state exactly as it would look on the real touchscreen, no hardware required. It's also how I caught a real cross-platform build bug — the simulator's SDL2 setup only worked by accident on Linux, and broke on macOS until I traced it to how Homebrew packages its headers differently. That loop, edit C, rebuild, watch it render, taught me to treat the simulator as ground truth rather than trusting how a mockup merely looks on paper.</p>
</div>

<div style="background:#fcfcfc; border:1px solid #e5e5e7; border-left:4px solid #ff6600; border-radius:6px; padding:20px 24px; margin:0;">
<div style="font-family: ui-monospace, 'SF Mono', Menlo, Consolas, monospace; font-size:11px; font-weight:600; letter-spacing:2px; color:#ff6600; margin-bottom:10px;">WHY IT MATTERS</div>
<p style="margin:0; color:#222; line-height:1.65;">The most beautiful thing is Summer of Bitcoin gave me the identity of an open source contributor and henceforth I see myself being passionately crazy with my thoughts in the contributors section of repositories. I feel obsessed with open source codebases and the amount of joy it gives me whether im able to fix a bug or introduce a feature or simply understand the codebase everything feels like the fascinating Hogwarts to me….</p>
</div>
