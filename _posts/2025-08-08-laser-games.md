---
layout: post
---

# ⚡️ VEC3x – Laser-Powered Vector Gaming Console

**VEC3x** is a custom-built game console I wrote, that brings vector-style arcade gaming into the physical world using **laser projection technology**. Originally designed as an MVP for an XR platform, the system has evolved into a standalone console with real hardware, tools for developers, and support for public outdoor gaming events.

{% include youtubeplayer.html id="ltKPo437-PQ" %}

## 🚀 Project Overview

VEC3x is a **laser-based vector game console** that:

- Plays custom-designed vector-style games
- Emulates 20+ classic **Vectrex** titles
- Projects graphics onto **walls, water, sails**, or any large surface
- Supports **up to 4 players** via smartphone that act as controllers
- Provides a **full development environment** for creating new laser-based games


## 🛠️ System Architecture

### 1. Laser Display System
- Uses a **professional ILDA-compatible laser projector**
- Real-time vector rendering via USB DAC and custom ILDA interface
- Supports both **indoor and outdoor** use
- Projects on buildings, sails, water surfaces, fog, etc.

### 2. Linux-Based Handheld Console
- Runs a custom **vector game engine**
- Outputs optimized point data for laser projection
- Compact, with Wi-Fi and USB connectivity

![Editor](/images/laser.gif)

### 3. Smartphone Multiplayer Interface
- Up to **4 smartphones** connect via Wi-Fi or Bluetooth

![Smartphone controller](/images/smartphone-controller.png)

## 🎮 Game Engine and SDK

The VEC3x engine is built from the ground up for **laser vector output**:

- Optimized vector path ordering (minimizes blanking and mirror jitter)
- Intensity modulation for glow and fade effects
- Frame generation: ~60 vector paths @ 30 FPS
- Real-time USB DAC output and ILDA streaming

### Development Tools

- Desktop **game simulator**
- Live code reload over LAN
- C/C++ SDK with full access to hardware

### Included Games

- Laser Pong (with water projection mode)
- Laser Chess (player vs. computer or player)
- Vector Breakout (multiplayer)
- Star Castle-style shooter
- Vectrex emulator with support for 20+ original games


## ⚙️ Hardware Details

| Component       | Specs                                                                 |
|-----------------|-----------------------------------------------------------------------|
| Laser Projector | ILDA-compatible, Class 3B (with safety features)                      |
| Console         | Custom Raspberry computer.                                            |
| Inputs          | 4x smartphone controllers, GPIO, USB, optional sensors                |
| Outputs         | Laser projection, USB DAC, DMX, audio                                 |


## 🌍 Outdoor Gaming in Barcelona

A public **outdoor installation** is in preparation for Barcelona:

- Games projected onto building walls, sails, and public spaces
- Connect via QR code and play using your smartphone
- Real-world multiplayer experience — no screens required


![Barcelona outdoor installation](/images/Barcelona2.png)

## 📦 Roadmap

- [x] Handheld console prototype
- [x] Laser DAC communication layer
- [x] Vector game engine
- [x] Vectrex emulation
- [ ] Open SDK & tooling
- [ ] Public beta testing event
- [ ] Open-source parts of the system


## 👾 Who Is This For?

- Retro gaming fans
- Creative coders and game developers
- Laser and AV installation artists
- Indie devs building physical-digital experiences

---

> **VEC3x** – Bringing vector games back from the 80s, and onto the city walls of the future.
