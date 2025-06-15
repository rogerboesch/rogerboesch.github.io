---
layout: post
---

# 👾 Chess on the Vectrex: Multiplatform C Development with Raspberry Pi Pico, and Raspberry Pi

Bringing modern code to retro hardware is a passion of mine—and few retro platforms are as uniquely charming as the **Vectrex**, the only home console with a built-in vector display. For my current project, I combined my love of C programming, embedded systems, and retro gaming to create a **multiplatform chess program for the Vectrex**, using **PiTrex** and both **Raspberry Pi Pico** and **Raspberry Pi** boards.

![Baremetal](/images/baremetal.png)

## PiTrex + Vectrex = A Modern Playground for Vector Dreams

The **PiTrex** project is an open hardware interface that allows modern Raspberry Pi boards to take full control of the Vectrex vector display. It plugs into the cartridge port and bypasses the original 6809 CPU, giving you direct access to the analog vector hardware using modern microcontrollers.

My chess program runs on **all platforms**:
- 🔧 **Raspberry Pi Pico (RP2040)** in **bare-metal C**—no OS, full control, super minimal.
- 🖥️ **Raspberry Pi Zero/3/4** under **Linux**.
- 💻 **macOS, Windows** using GLEW.

All targets share the **same C codebase** and use my custom-written **vector graphics library** built specifically for PiTrex development.


## Multiplatform from the Start

From day one, I designed the project to be **platform-independent**. Whether you’re flashing code to a Pico or compiling it on a Linux-based Pi, the logic, rendering, and engine code are identical.

Thanks to a few well-placed platform abstraction, the project supports:

- **Bare-metal builds** for the RP2040 (Raspberry Pi Pico)  
- **Standard Linux builds** for Raspberry Pi boards (Pi Zero, Pi 3/4)  
- **The same rendering pipeline** for both, using my vector library

This setup makes development fast and flexible: I test logic-heavy components like the chess engine or input handling on the Pi 4, and later deploy to the Pico for performance and timing tests.

---

## My Custom Vector Library

To drive the Vectrex display via PiTrex, I wrote a **custom vector graphics library** in C that works across all supported platforms.

It includes:
- A scalable **vector font system** (used for chess pieces and UI)
- Basic primitives (lines, circles, boxes)
- Utility functions for drawing the chess board, moves, highlights
- Frame sync and draw control abstractions for both Linux and bare-metal

The library abstracts the PiTrex vector hardware enough to make cross-platform rendering seamless—whether you're running on a full Linux Pi or a stripped-down RP2040 firmware.

---

## The Chess Engine

At the core of it all is a **handwritten chess engine** in pure C:

- Legal move generation  
- Turn-based logic and state handling  
- A basic "AI" player  
- Vector-based UI rendering

The goal was not just to show moves, but to create a playable, elegant chess experience on real hardware with vector graphics—a blend of game logic and visual design optimized for minimal systems.

---

## Development Workflow

My typical workflow looks like this:

1. Code and test logic-heavy parts (like engine and UI) on a **Raspberry Pi 4** under Linux.  
2. Cross-compile the same codebase for **Raspberry Pi Pico**, using the [Pico SDK](https://github.com/raspberrypi/pico-sdk).  
3. Flash to the Pico and test on real Vectrex hardware using **PiTrex**.  
4. Iterate!

This gives me the best of both worlds: **modern dev speed with real hardware authenticity**.

---

## See It in Action

The result? A fully playable chess game on the original Vectrex screen, powered by either a Raspberry Pi Pico (running bare-metal) or a Raspberry Pi board running Linux.

✅ Multiplatform C codebase  
✅ Custom vector drawing library  
✅ Minimal UI and gameplay loop  
✅ Bare-metal and Linux-compatible  
✅ Beautiful vector rendering on Vectrex CRT

🔗 **Vectrex chess**: [github.com/rogerboesch/pitrex-chess](https://github.com/rogerboesch/pitrex-chess)
🔗 **Pixel Engine**: [github.com/rogerboesch/rbxe](https://github.com/rogerboesch/rbxe)

---

## What's Next?

Coming soon:
- Improved AI opponent  
- Move history, undo, and checkmate detection  
- Alternate themes and overlays  
- Possibly a complete Vectrex-compatible cartridge or prebuilt PiTrex image

---

## Final Thoughts

This project brings together everything I enjoy:
- **Bare-metal embedded C programming**  
- **Multiplatform code architecture**  
- **Retro hardware and vector graphics**  
- And of course: **Chess.**

If you're into classic gaming, embedded development, or just love writing efficient C code that talks directly to hardware, I encourage you to check out the repo and try building it yourself.

Let’s keep the Vectrex glowing—across platforms.

🔗 **GitHub Repo**: [github.com/rogerboesch/pitrex-chess](https://github.com/rogerboesch/pitrex-chess)
