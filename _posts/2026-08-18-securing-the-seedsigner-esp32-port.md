---
layout: post
title: "Securing the SeedSigner ESP32 Port"
author: "Mayank Yadav"
date: 2026-08-18
categories: [Stories, Wallets, Security, Development]
image: ../assets/images/blog_content/2026-08-18-securing-the-seedsigner-esp32-port.jpeg
---

During my Summer of Bitcoin 2026 internship with SeedSigner, I worked on designing and implementing a stateless secure bootloader for the ESP32 port. In this post, I dive into the hardware threat model, the low-level MMU and cache mechanics required to run MicroPython statelessly from volatile PSRAM, anti-phishing flash protections, and the engineering lessons learned bringing stateless Bitcoin signing to physical silicon.

## The Problem: Firmware That Forgets

SeedSigner is a Bitcoin hardware wallet designed around a single idea: *no secrets should persist on the device*. You power it on, enter your seed, sign your transaction, and power it off. When the power goes, the seed goes with it. That's the stateless promise.

But here's the thing - a promise is only as strong as its enforcement. SeedSigner is being ported from Raspberry Pi to ESP32 microcontrollers (the ESP32-P4 and ESP32-S3), and with that port comes an opportunity most people wouldn't think about: the bootloader.

On a Raspberry Pi, the firmware lives on a removable SD card. Pull the card and the device is a blank slate. But on an ESP32, applications are typically baked into onboard SPI flash - persistent, non-volatile memory that survives power cycles. A malicious firmware could silently write your seed into flash, and an attacker could retrieve it later by physically reading the chip. The user would never know.

My Summer of Bitcoin project was to close that attack surface.

I set out to build a **stateless secure bootloader** for the SeedSigner ESP32 port: a system that reads signed firmware from a removable SD card, verifies it cryptographically, and runs it entirely from volatile memory - with onboard flash never written after the device is provisioned. If you pull the power, every byte of runtime state evaporates.

## Why Not Just Use Flash?

The instinct is to ask: why go through all this trouble? Just store the firmware in flash like every other ESP32 project. The answer comes back to the threat model.

A hardware wallet holds private keys, even if transiently. The attack surface isn't just software - it's physical. Someone with five minutes of unsupervised access to your device (the "evil maid" scenario) could:

1. **Swap the SD card** with malicious firmware that looks identical but secretly writes your seed to a hidden flash sector.
2. **Read the flash** later (via SPI probing, UART, or JTAG) and extract the stashed seed.
3. **Broadcast the seed** over Wi-Fi from within the malicious firmware itself.
4. **Execute a Dark Skippy attack** - embed the private key into a PSBT signature the user unknowingly broadcasts.

![Without flash fill protection, empty flash sectors silently accept arbitrary writes from malicious firmware](../assets/images/blog_content/2026-08-18-naive-secret-stashing.png)

So flash is out. It's non-volatile, writable, and probeable. The firmware has to run from volatile memory - memory that does not retain data after power loss.

The ESP32 has several types of volatile memory, but most of them are too small. Internal SRAM is fast and on-chip, but only a few hundred kilobytes - nowhere near enough to hold a full MicroPython runtime. IRAM (Instruction RAM) is a subset of that same internal SRAM, reserved for timing-critical code and interrupt handlers. DRAM holds variables and heap. None of these can fit a multi-megabyte application.

Then there's **PSRAM** - Pseudo-Static RAM. It's an external volatile memory chip (up to 32 MB on the P4, 8 MB on the S3) that sits on the SPI bus alongside flash. The "pseudo-static" means it's built on capacitors like DRAM, so it's genuinely volatile - the information doesn't vanish instantaneously, but within seconds as the capacitors discharge after power loss. That's exactly what we need: large enough to hold the entire firmware, volatile enough that seeds can't persist.

![ESP32-P4 memory organization: flash, PSRAM, and internal SRAM regions](../assets/images/blog_content/2026-08-18-memory-organization.png)

The plan becomes: read the signed firmware from the SD card, load it into PSRAM, verify the signature, and execute it directly from there. Pull the power, and the capacitors drain — no persistent storage holds the runtime state.

The catch is that ESP-IDF, Espressif's development framework, was never designed to run applications from PSRAM. There's no API for it. No documentation. Normally, the CPU fetches code from flash through the MMU (Memory Management Unit) - the MMU translates virtual addresses to physical flash addresses, and the instruction cache serves the bytes transparently. The CPU never knows where the bytes actually live.

![How a normal ESP32-P4 application executes: flash code via MMU, RAM data via direct copy](../assets/images/blog_content/2026-08-18-normal-execution.png)

My bootloader hijacks this mechanism. Instead of mapping the flash cache window to flash, it **reprograms the MMU entries to point at PSRAM physical pages**. The CPU still fetches from the same virtual addresses, the instruction cache still serves the bytes - but underneath, the data comes from volatile PSRAM instead of persistent flash. The firmware is tricked into thinking it's running from flash, while the underlying physics ensure that data decays rapidly after power loss.

This required understanding the MMU at the register level - no ESP-IDF API exists for what I needed. On the P4, I had to program `SPI_MEM_C` and `SPI_MEM_S` register banks directly, computing entry IDs from virtual addresses and MMU values from physical addresses. On the S3, it was a single shared I/D MMU table at `DR_REG_MMU_TABLE` (`0x600C5000`) with 512 entries of 64 KB each. Two completely different hardware models, same conceptual trick.

## The Architecture: Two Layers of Trust

The bootloader uses a dual-layer Root of Trust - two independent cryptographic gates that firmware must pass before it can execute.

![The secure boot chain: eFuse → 2nd-stage bootloader → secure loader → JMP zone → payload](../assets/images/blog_content/2026-08-18-boot-chain-flow.png)

**Layer 1 - Secure Boot V2 (hardware root of trust):** The ESP-IDF second-stage bootloader is signed with an RSA-3072 key whose digest is burned into the chip's one-time-programmable eFuses. On every power cycle, the silicon ROM verifies the bootloader, and the bootloader verifies our loader application. If the signature doesn't match, the chip refuses to boot. An attacker who reflashes the loader with a tampered version is stopped cold - the RSA check runs before any of our code.

**Layer 2 - Specter secp256k1 multisig (software root of trust):** The loader reads a signed firmware bundle from the FAT32 SD card and verifies a secp256k1 multisig signature against vendor public keys compiled into the loader binary. This is the same elliptic curve Bitcoin itself uses for transaction signing. The bundle format comes from the [Specter-DIY](https://github.com/cryptoadvance/specter-bootloader) project - a 256-byte header, the raw ESP32 firmware image, and a signature section:

![The Specter bundle format: header + ESP32 image payload + secp256k1 multisig signatures](../assets/images/blog_content/2026-08-18-specter-bundle.png)

This dual-chain design means two completely independent keys must be compromised to run arbitrary code on the device. Layer 1 protects the loader in flash; Layer 2 protects the firmware on the SD card. They use different cryptographic algorithms (RSA vs. ECDSA), different key management workflows, and different storage (eFuse vs. compiled-in constants).

## The Handoff Problem: Two Programs, One Memory

With the MMU trick solved, the next challenge was getting the loader out of the payload's way.

The loader is a normal ESP-IDF application. MicroPython (the payload) is also a normal ESP-IDF application. Both are compiled with linker scripts that place their code and data starting at `0x4FF00000` in internal SRAM. **They both want to occupy the same physical memory.**

![The memory overlap conflict: loader and payload fight for the same SRAM address range](../assets/images/blog_content/2026-08-18-memory-overlap.png)

This collision - two normal programs fighting over one physical address space - is the central design constraint. The PSRAM-mapped segments are straightforward (the MMU handles those), but the payload's internal SRAM segments have fixed addresses that overlap with the loader's own code and stack. The handoff from the loader to the payload has to be handled very carefully.

### The JMP Zone: Point of No Return

The actual handoff from the loader to the payload happens in what I call the **JMP zone** - a section of code that runs with interrupts disabled, on a dedicated stack, using only bare-metal register writes. Once entered, there is no way back. No `ESP_LOG`, no `malloc`, no FreeRTOS. Just raw UART FIFO writes for debug output and inline assembly.

The JMP zone has to:

1. **Kill everything**: all watchdogs (SWD, LP_WDT, MWDT0, MWDT1, RWDT), all interrupts, the hardware stack guard, all PMP (Physical Memory Protection) constraints.
2. **Evict the data cache** (pre-copy): write a scratch pattern over a large buffer to force dirty cache lines out to SRAM, clearing the way for the payload's data.
3. **Copy SRAM segments**: byte-by-byte from PSRAM to internal SRAM at the payload's expected addresses.
4. **Reprogram the MMU**: point the flash cache window at PSRAM physical pages so the payload's code and read-only data resolve correctly.
5. **Drain the data cache** (post-copy): the SRAM copies went through the write-back D-cache, so the payload bytes are sitting in dirty cache lines, not yet in physical SRAM. Writing the entire 256 KB `evict_buf` with distinct values forces every set and way in the L1 cache, evicting the payload's dirty lines as a side effect.
6. **Jump**: cast the entry address to a `noreturn` function pointer and call it.

The ordering here is load-bearing. If you evict *after* copying, the scratch pattern physically overwrites the payload's `.data` section - every initialized global variable becomes garbage. I learned this the hard way through a bug that took days to track down.

### The Linker Script Trick

To solve the memory overlap, I wrote a custom linker script (`loader_high.ld`) that redefines the `MEMORY` regions, moving the loader's entire footprint up to `0x4FF40000`. This ensures the loader never executes from the payload's address range, eliminating stale instruction cache problems. GNU ld allows a later `MEMORY` declaration to override an earlier one - it emits a warning, but that warning is expected and harmless.

The FreeRTOS main-task stack posed a separate problem: it lands at around `0x4FF04590` - squarely inside the payload's SRAM `.text` range. The JMP zone defines a dedicated 32 KB `jump_stack` in the relocated region and uses a `naked` trampoline to switch the stack pointer before any payload work begins.

## Flash Hardening: The Anti-Phishing Proof

Even with stateless execution from PSRAM, there's a residual attack: the payload firmware runs with full hardware access, including write access to onboard flash. A sufficiently sophisticated malicious firmware could write stolen secrets to flash during runtime, then the attacker retrieves them later.

The anti-phishing system closes this with a tamper-evident seal. On first boot, the loader floods a ~6 MB `random_fill` partition with data from the hardware True Random Number Generator. It hashes the result with SHA-256 and derives **4 BIP-39 words** that are displayed to the user before every boot:

> `abandon also acid airport`

If anything on flash changes - even a single byte - the hash changes, the words change, and the user sees the discrepancy immediately. In the current implementation, the bootloader itself catches the mismatch and halts before any firmware runs.

![With flash fill, any secret stashing requires erasing flash sectors - which changes the anti-phishing words](../assets/images/blog_content/2026-08-18-secret-stashing-with-flash-fill.png)

I also analyzed the brute-force collision attack: can an attacker find different data that hashes to the same 4 words? With 2 words (the early prototype), the collision space was ~2²² - roughly 84 seconds on an ESP32. **The attack succeeded in testing.** Moving to 4 words shifts the space to 2⁴⁴. Naively re-hashing 6 MB per candidate costs ~149 ms per try - that's ~83 million years. Even the optimized incremental attack (rehash only the last block) takes ~262 days on a host machine. And the real endgame is HMAC eFuse binding: mix a hardware-fused secret key into the hash, and the attacker loses the ability to compute candidates entirely.

![The brute-force collision attack: loop over padding bytes until SHA-256 produces matching words - feasible with 2 words, intractable with 4](../assets/images/blog_content/2026-08-18-secret-stashing-brute-force.png)

## Porting Across Architectures: RISC-V and Xtensa

The bootloader runs on two fundamentally different ESP32 chips:

| | **ESP32-P4** | **ESP32-S3** |
|---|---|---|
| **Architecture** | RISC-V | Xtensa |
| **Board** | Waveshare ESP32-P4 WiFi6 Touch LCD 4.3 | ESP32-S3-DEV-KIT-N8R8 / Waveshare S3 Touch LCD 3.5B |
| **PSRAM** | 32 MB (HEX SPI) | 8 MB (Octal) |
| **MMU model** | Separate I/D MMU tables via SPI_MEM_C/S | Shared I/D table at `0x600C5000` (512 × 64 KB) |
| **JMP zone location** | Custom MEMORY region in SRAM | IRAM `0x403A0000–0x403B8000` |
| **Stack switch** | Inline `mv sp, %0` (RISC-V) | `mov a1, %0` (Xtensa, requires `asm volatile` - a plain register assignment is a dead store) |
| **SD card** | On-chip LDO4, SDMMC 4-bit | External wiring, SDSPI / SDMMC 1-bit |

The S3 port uncovered architecture-specific traps. On Xtensa, `.jmp_zone.literal` sections must precede `.jmp_zone.text` - the `l32r` instruction loads from a literal pool *before* the instruction, so placing literals after use causes a fatal assembler error. The S3's IRAM/DRAM are aliased at a `0x6F0000` offset, meaning code at `0x40370000` and data at `0x3FC88000` physically share the same SRAM - which constrains how much combined code and data the JMP zone can use.

For the S3's SD card, I had to solder external wiring since there's no on-chip SD LDO:

![ESP32-S3 dev kit wired to an external HW-125 SD card module on a breadboard](../assets/images/blog_content/2026-08-18-circuit-esp32s3.jpg)

The bootloader supports both SDSPI (SPI bus) and SDMMC (native SD protocol) with automatic fallback - critical for compatibility with the standard buffered HW-125 Arduino SD adapters that most hobbyists use.


## Putting It All Together

Three public repositories represent the summer's work:

1. **[seedsigner-esp32-bootloader](https://github.com/wolgwang1729/seedsigner-esp32-bootloader)** - The standalone, unified bootloader codebase. Supports both ESP32-P4 and ESP32-S3 from a single modular C codebase (`main.c`, `storage.c`, `esp_image.c`, `jump.c`, `anti_phish.c`), with build-time vendor key profiles, automated GitHub Actions CI, and hardware pytest test suites.

2. **[SoB-SeedSigner-Bootloader-Journey](https://github.com/wolgwang1729/SoB-SeedSigner-Bootloader-Journey)** - The comprehensive 17-phase R&D monorepo documenting the entire development journey. Every hardware error, silicon register quirk, cache coherency failure, and the exact root causes behind each line of code - so future developers understand why things are the way they are.

3. **[seedsigner-micropython-builder](https://github.com/wolgwang1729/seedsigner-micropython-builder)** - Fork of the SeedSigner MicroPython builder with the `stateless_shim` boot component for both architectures.

Both the ESP32-P4 and ESP32-S3 bootloaders are **hardware-verified end-to-end**: the loader boots, reads from the SD card, verifies the Specter signature, displays anti-phishing words, and hands off to MicroPython running statelessly from PSRAM - confirmed on physical silicon with interactive REPL input.

#### ESP32-P4 Hardware Demo
<div class="embed-responsive embed-responsive-16by9 my-3">
  <iframe class="embed-responsive-item" src="https://www.youtube.com/embed/jkk55UFmpCI" title="ESP32-P4 Stateless Bootloader Demo" allowfullscreen></iframe>
</div>

#### ESP32-S3 Hardware Demo
<div class="embed-responsive embed-responsive-16by9 my-3">
  <iframe class="embed-responsive-item" src="https://www.youtube.com/embed/al7gvf-3Q34" title="ESP32-S3 Stateless Bootloader Demo" allowfullscreen></iframe>
</div>

## The Road Ahead

Security work is never finished. The attack scenarios are limitless, and several hardening steps are deliberately deferred to production:

- **eFuse provisioning**: The bootloader currently uses virtual eFuses (`CONFIG_EFUSE_VIRTUAL=y`) - software-emulated fuses that don't burn real silicon. This is intentional for development: physical eFuse burns are *irreversible*, and the ESP-IDF eFuse APIs have seen breaking changes across recent ESP32-P4 silicon revisions. Burning with unstable APIs risks permanently bricking development hardware. The virtual eFuse emulation faithfully replicates the security chain while keeping the hardware reusable.

- **HMAC eFuse key binding**: The anti-phishing proof currently uses a plain SHA-256 hash. The production plan specifies `HMAC(eFuse_key, SHA-256(flash_data))` - mixing a hardware-fused secret into the digest so collision attacks become computationally intractable. This requires a physical eFuse burn.

- **Anti-rollback counters**: The loader enforces a version floor (`pl_ver >= 1`), but true anti-rollback needs eFuse-burned version counters that increment monotonically with each release.

- **Flash encryption**: XTS-AES-128 encryption of the immutable loader artifacts in flash. Currently blocked on ESP-IDF >= 6.1 support for `CONFIG_SPIRAM_ENC_EXEMPT`.

- **Upstream merge**: The `stateless_shim` overlay needs to be merged upstream into the SeedSigner MicroPython builder, and the signed language-pack verification infrastructure needs to be finalized.

All six of the ESP32's eFuse key blocks (KEY0-KEY5) are allocated by the production plan. Zero spare. I've recommended keeping one Secure Boot V2 rotation slot in reserve rather than filling all three.

Beyond the hardening checklist, I want people to try to break it. Recent industry wake-up calls - such as the Coinkite entropy vulnerability - show just how unforgiving embedded firmware bugs can be. The 12 attack scenarios I analyzed are the ones I thought of; the critical ones are the ones I didn't. Every bug caught and patched before this ships to production is a user seed protected.

The eFuse HAL and anti-tamper APIs on the ESP32-P4 have been a moving target, and once they stabilize (likely ESP-IDF v6.x), the virtual eFuse training wheels come off and the full production hardening plan can be executed. That also means ordering dev boards specifically for destructive testing - boards that will be permanently locked down and potentially bricked in the process of validating every eFuse burn path.

## Final Thoughts

This project took me from writing application-level Python to programming MMU registers by hand, from reading datasheets to soldering SD card wires onto dev boards.

**The gap between emulation and silicon is enormous.** The bootloader worked perfectly in QEMU for weeks. The moment I moved to real hardware, everything broke - cache controllers behaved differently, MMU hardware routing had undocumented quirks, and PSRAM timing constraints that emulators simply don't model caused silent corruption. If I could start over, I'd skip emulators entirely and start on physical chips from day one.

**Memory is not what you think it is.** On an ESP32, the same physical SRAM byte is reachable from two different addresses depending on whether you access it as code (instruction bus) or data (data bus). The CPU sees virtual addresses, the cache sees physical pages, and the MMU sits between them translating - and every layer can lie to the one above it. Understanding this stack from silicon to software was the single most valuable technical skill I gained.

**Ordering is load-bearing.** The evict-before-copy ordering bug took days to find because the symptoms (corrupted `.data` globals) had no obvious connection to the cause (cache eviction sequence). In bare-metal code with no operating system abstractions to lean on, the *order* you do things in is as important as *what* you do.

**The real security boundary is key management, not code logic.** Every cryptographic check in the bootloader works correctly. The security analysis proved it with 16 hardware tests. But the entire chain rests on the secrecy of two private keys. No code change can fix a leaked key. The hardest part of shipping this to production has nothing to do with firmware - it's operational security around key generation, storage, and rotation.

I'll be continuing to contribute to SeedSigner and the broader Bitcoin hardware ecosystem. The bootloader is open-source, the documentation captures every decision and failure mode, and the engineering journey is fully public.

If you're an embedded developer, security researcher, or Bitcoin builder, here's how you can get involved:
- **Try flashing the bootloader**: Check out the setup guides and test the stateless loader on your ESP32-P4 or ESP32-S3 hardware.
- **Audit and break the security model**: Review our threat model, poke at the virtual eFuse setup, and open an issue if you discover edge cases or potential vulnerabilities.
- **Contribute**: Feel free to submit PRs for new board targets, optimizations, or MicroPython shims on GitHub!

---

*Repositories:*
- *Bootloader: [seedsigner-esp32-bootloader](https://github.com/wolgwang1729/seedsigner-esp32-bootloader)*
- *Journey: [SoB-SeedSigner-Bootloader-Journey](https://github.com/wolgwang1729/SoB-SeedSigner-Bootloader-Journey)*
- *MicroPython builder: [seedsigner-micropython-builder](https://github.com/wolgwang1729/seedsigner-micropython-builder)*
