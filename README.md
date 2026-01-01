# NiXX-Cores™

**Emulation cores for the NiXX™ architecture.**

This repository contains the core emulation libraries for NiXX™ systems. These cores power [NiXX-Studio™](https://github.com/NiXX-Team/NiXX-Studio) and can be used standalone or integrated into other frontends.

---

## Available Cores

| Core | Status | Description |
|------|--------|-------------|
| [NiXX-32™](NiXX-32/) | Pre-Release | Dual-CPU arcade system (M68000 + Z80) |

---

## NiXX-32™

This core supports both hardware variants:

- **NiXX-32™** (1989) - Base hardware with dual-CPU architecture
- **NiXX-32+™** (1992) - Enhanced variant with larger sprites, more colors, and expanded memory

### Key Features

- **Dual-CPU Architecture** - M68000 (16-20 MHz) + Z80 (8-10 MHz)
- **Graphics** - 384x224 resolution, up to 8,192 colors, 3-4 layer parallax, hardware sprites
- **Audio** - YM2151 FM synthesis + NiXX-B DSP + Q-Sound spatial audio
- **Memory** - Up to 4 MB ROM, 2 MB system RAM, 1 MB VRAM

See [NiXX-32/](NiXX-32/) for full specifications of both variants.

---

## Installation

Download the latest release for your platform from the [Releases](https://github.com/NiXX-Team/NiXX-Cores/releases) page.

### For NiXX-Studio™ Users

Place the core files in your NiXX-Studio™ installation's `cores/` directory.

### Library Integration

NiXX32Core.dll is the emulation core library for developers building custom frontends.

---

## Supported Platforms

- Windows 10/11 (64-bit)

Linux and macOS support planned for future releases.

---

## Building from Source

The core source code is not publicly available at this time. Binary releases are provided for supported platforms.

---

## License

NiXX-Cores™ is proprietary software. See [LICENSE.md](LICENSE.md) for terms.

**© 2025 Starsick™. All Rights Reserved.**

---

## See Also

- [NiXX-Studio™](https://github.com/NiXX-Team/NiXX-Studio) - Official IDE for NiXX™ development
