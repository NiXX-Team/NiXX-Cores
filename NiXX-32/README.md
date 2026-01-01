# NiXX-32™

**A dual-CPU arcade system in the NiXX™ architecture.**

NiXX-32™ combines notable aspects of late-80s/early-90s hardware into a cohesive, developer-friendly environment.

This core supports both hardware variants:
- **NiXX-32™** (1989) - Base hardware
- **NiXX-32+™** (1992) - Enhanced variant

---

## Hardware Specifications

### Processors

| CPU | NiXX-32™ | NiXX-32+™ |
|-----|----------|-----------|
| Main | Motorola 68000 @ 16 MHz | Motorola 68000 @ 20 MHz |
| Sound | Zilog Z80 @ 8 MHz | Zilog Z80 @ 10 MHz |

### Graphics

| Feature | NiXX-32™ | NiXX-32+™ |
|---------|----------|-----------|
| Resolution | 384x224 | 384x224 |
| Colors On-Screen | 4,096 | 8,192 |
| Palette | 32,768 colors | 32,768 colors |
| Background Layers | 3 | 4 |
| Sprites | 96 simultaneous | 128 simultaneous |
| Max Sprite Size | 32x32 | 64x64 |
| Effects Chip | Basic particles (smoke, sparks) | Enhanced particles with transparency |

### Audio

| Feature | NiXX-32™ | NiXX-32+™ |
|---------|----------|-----------|
| FM Synthesis | YM2151 (8 voices) | YM2151 (12 voices) |
| PCM | NiXX-B DSP, 8-bit @ 11 kHz | NiXX-B DSP, 8-bit @ 22 kHz |
| Sound ROM | 128 KB | 256 KB |
| Spatial Audio | Q-Sound DSP | Q-Sound DSP |

### Memory

| Region | NiXX-32™ | NiXX-32+™ |
|--------|----------|-----------|
| System RAM | 1 MB | 2 MB |
| Video RAM | 512 KB | 1 MB |
| Game ROM | Up to 2 MB | Up to 4 MB |
| Sound ROM | 128 KB | 256 KB |

### Input

- 2 player support (NiXX-32+™ has dedicated linking circuitry)
- 8-direction digital joystick per player
- 6 action buttons per player (A, B, C, X, Y, Z)
- Start and Select buttons
- Directly mapped to keyboard/gamepad

---

## File Formats

### ROM Format (.n32)

NiXX-32™ ROMs use the `.n32` extension and contain:

- Header (256 bytes) - metadata, checksums, target variant
- Code segment - 68000 executable code
- Graphics segment - tile and sprite data
- Audio segment - FM patches, PCM samples
- Data segment - tilemaps, scripts, other assets

### Save Format (.sav)

Battery-backed SRAM saves use 8 KB maximum.

---

## Development

ROMs for NiXX-32™ can be developed using:

- **NiXX-Studio™** - Full IDE with visual editors
- **Lua scripting** - High-level ROM logic
- **C/C++** - Direct hardware access
- **Assembly** - 68000/Z80 for performance-critical code

---

## Usage

NiXX32Core.dll is the emulation core library. It is used by:
- **NiXX-Studio™** - The official IDE for NiXX-32™ ROM development
- **Custom frontends** - Integrate the core into your own applications

### Integration

```cpp
#include <NiXX32/NiXX32.h>

int main() {
    // Initialize the platform
    if (!NiXX32::Platform::initialize()) {
        return -1;
    }

    // Create system configuration (choose variant)
    auto config = NiXX32::SystemConfig::create_default(
        NiXX32::HardwareVariant::NIXX32_PLUS_1992  // or NIXX32_1989
    );

    // Create and run the system
    auto system = NiXX32::System::create_instance(config);
    system->loadROM("game.n32");
    system->run();

    NiXX32::Platform::shutdown();
    return 0;
}
```

### Keyboard Controls (Default)

| Action | Player 1 | Player 2 |
|--------|----------|----------|
| Up | W | Arrow Up |
| Down | S | Arrow Down |
| Left | A | Arrow Left |
| Right | D | Arrow Right |
| Button A | J | Numpad 1 |
| Button B | K | Numpad 2 |
| Button C | L | Numpad 3 |
| Button X | U | Numpad 4 |
| Button Y | I | Numpad 5 |
| Button Z | O | Numpad 6 |
| Start | Enter | Numpad Enter |
| Select | Backspace | Numpad 0 |

---

## Requirements

- **OS:** Windows 10/11 (64-bit)
- **RAM:** 256 MB minimum
- **Disk:** 50 MB
- **Dependencies:** SDL2 runtime

---

## License

Proprietary software. See [../LICENSE.md](../LICENSE.md) for terms.

**© 2025 Starsick™. All Rights Reserved.**
