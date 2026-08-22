---
layout: post
title: "Fixing SeedSigner’s Navigation Maze with Static Analysis"
author: "Kshitij Dhamanikar"
date: 2026-08-23
categories: [Development, Testing, UX]
---

## The Spark: A Manual Discovery

During my early work with SeedSigner, I noticed that testing an air-gapped hardware wallet requires more than just checking the "happy path." To ensure the device is completely robust, we need to know exactly what happens in every possible edge case. 

This became very clear when I was manually testing the device and stumbled upon a subtle UX flaw. I discovered that cancelling out of entering a seed phrase improperly dropped the user straight to the main menu, wiping out their ongoing workflow. I fixed this manual discovery in [Pull Request #900](https://github.com/SeedSigner/seedsigner/pull/900). 

While that PR patched the immediate problem, it raised a much bigger question: *If this one navigation trap was lurking in the code, how many others were hiding in the hundreds of different screen transitions?* Finding them manually would be a nightmare. I needed a way to map out the entire application's navigation programmatically.

## Uncovering Hidden Paths with AST

To solve this, I built the **AST Navigation Graph Extractor**. Instead of manually clicking through menus, this tool leverages an Abstract Syntax Tree (AST) to read and parse the actual Python code of the project. By analyzing the structure of the code statically—without ever running the application—the tool programmatically mapped out every single screen and every possible transition between them. 

This generated a complete "map" of the application's navigation. When I compared this map against our existing automated tests, two major issues stood out: several screen transitions had zero test coverage, and more importantly, some deeply nested workflows contained hidden navigation bugs.

## Escaping the Navigation Traps

SeedSigner features complex, deeply nested workflows, such as verifying a multisig address or signing a transaction. The most interesting problem the AST tool uncovered was what happens when a user navigates away from these deep menus.

In several edge cases, taking an unexpected action—like scanning an invalid QR code or abandoning a flow midway—didn't gracefully return the user to the previous screen. Because SeedSigner is highly secure and stateless, dropping back too far instantly wiped out all of the user's ongoing progress. In other instances, users would encounter warning screens that lacked proper routing instructions entirely, effectively trapping them on those screens and forcing a device restart.

The solution was to carefully trace these broken paths and rewrite the routing logic. Now, when a user encounters an edge case, the device intelligently walks back up its history stack, restoring the exact previous screen and keeping the user's workflow perfectly intact.

## Fixing the Flows

Armed with the data from the AST tool, I put together 10 separate pull requests to patch these navigation gaps. I grouped the fixes into focused chunks so that the project maintainers could easily review and test them independently. You can see the full list in my [tracking issue (#1022)](https://github.com/SeedSigner/seedsigner/issues/1022).

These fixes spanned across several key areas:
- **QR Scanning:** Fixed situations where an invalid QR code or incomplete scan wiped the user's progress ([#983](https://github.com/SeedSigner/seedsigner/pull/983), [#984](https://github.com/SeedSigner/seedsigner/pull/984), [#998](https://github.com/SeedSigner/seedsigner/pull/998)).
- **Address Verification:** Rerouted canceled address verifications (including complex multisig setups) so they correctly return the user to their previous step ([#1000](https://github.com/SeedSigner/seedsigner/pull/1000), [#1009](https://github.com/SeedSigner/seedsigner/pull/1009), [#1021](https://github.com/SeedSigner/seedsigner/pull/1021)).
- **Seed Words:** Added complete end-to-end testing for the seed backup verification process and safely omitted warning screens from the back stack ([#958](https://github.com/SeedSigner/seedsigner/pull/958), [#960](https://github.com/SeedSigner/seedsigner/pull/960)).
- **Tools & Entropy:** Wrote tests for 16 previously untested navigation paths in the device's toolset ([#959](https://github.com/SeedSigner/seedsigner/pull/959)).
- **Message Signing:** Added safeguards to prevent application crashes when the message signing process was triggered without proper data ([#999](https://github.com/SeedSigner/seedsigner/pull/999)).

Every single one of these 10 pull requests is fully isolated, successfully tested on the SeedSigner emulator, and ready to be merged.

## Takeaways and Next Steps

The biggest lesson for me was shifting my perspective on testing. When you look at a codebase as a massive, interconnected graph rather than just a collection of individual screens, you catch subtle bugs that manual testing easily misses.

This isn't the end of the road for the AST tool, either. Moving forward, I will be working with my mentor on a long-term basis. We've discussed multiple directions and ideas for how we can leverage this AST work to build even more cool stuff. One of our primary goals is to integrate this tool directly into SeedSigner's CI pipeline, preventing new pull requests from inadvertently introducing these kinds of navigation bugs in the first place.

Building this tool has been incredibly rewarding, and I'm excited to see where we take it next!
