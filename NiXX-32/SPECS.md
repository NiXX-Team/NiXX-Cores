## Overview

| **Arcade Board**         | **NiXX-32™**                                                               | **NiXX-32™+ Changes**                                                      |
| ------------------------ | -------------------------------------------------------------------------- | -------------------------------------------------------------------------- |
| **CPU**                  | Motorola 68000 @ 10 MHz                                                    | Motorola 68000 @ 16 MHz                                                    |
| **Co-Processor**         | Zilog Z80H @ 8 MHz (audio processing)                                      | Same                                                                       |
| **Hardware Math**        | Fixed-point arithmetic only (no FPU)                                       | Fixed-point + hardware multiply assist registers (16x16 -> 32)             |
| **System Memory**        | 1 MB                                                                       | 2 MB                                                                       |
| **Video Memory**         | 512 KB (standard VRAM)                                                     | 1 MB (dual-port VRAM)                                                      |
| **Resolution**           | 384x224 pixels                                                             | 384x224 pixels                                                             |
| **Color Depth**          | 15-bit RGB555 (32,768 color space), 1,024 addressable per frame            | 16-bit RGB565 (65,536 color space), 4,096 addressable per frame            |
| **Sprite Handling**      | 96 simultaneous, 16-20 per scanline                                        | 128 simultaneous, 24-32 per scanline                                       |
| **Maximum Sprite Size**  | 32x32 pixels                                                               | 64x64 pixels                                                               |
| **Sprite Blending**      | 1-bit per sprite (50/50 blend)                                             | 2-bit per sprite (4 alpha levels)                                          |
| **Background Layers**    | 4 scroll layers + text plane                                               | 4 scroll layers + text plane + bitmap layer, line-select, window clipping  |
| **Video Output Effects** | Per-RGB brightness, mosaic, scanline darkening, shadow & highlight         | Same + dynamic priorities, per-color priority                              |
| **DMA**                  | Automatic sprite DMA + 1 general-purpose channel, 16-bit bus               | Automatic sprite DMA + 2 general-purpose channels, 32-bit bus              |
| **FM Synthesis**         | 1× Yamaha YM2151 (8 voices)                                                | 2× Yamaha YM2151 (16 voices)                                               |
| **PCM Sound**            | NiXX-B DSP: 8 channels, 8-bit, 11 kHz                                      | QSound replaces NiXX-B: 16 PCM + 3 ADPCM, 16-bit                          |
| **Audio Output**         | Stereo                                                                     | Stereo (with QSound spatial positioning)                                   |
| **Base Sample ROM**      | NXS-2000                                                                   | NXS-2001                                                                   |
| **Sound ROM**            | 128 KB                                                                     | 256 KB                                                                     |
| **Game ROM Capacity**    | 2 MB                                                                       | 4 MB                                                                       |
| **Multiplayer Support**  | None built-in                                                              | Dedicated linking circuitry                                                |
| **Anti-Piracy**          | Battery-backed encryption (“suicide” protection)                           | Enhanced encryption with boot verification                                 |
| **Based On**             | Sega Super Scaler heritage, CPS-1 era general-purpose architecture         | **Enhancement for sequels**                                                |
| **Game(s)**              | *Telefunk*                                                                 | *Telefunk II*, *Telefunk III*                                              |
| **Release Year**         | 1989                                                                       | 1992                                                                       |

---

## CPU

### Motorola 68000

| Feature          | NiXX-32                  | NiXX-32+                 |
|------------------|--------------------------|---------------------------|
| Clock            | 10 MHz (MC68000-10)      | 16 MHz (MC68000FN16)     |
| Cycles Per Frame | ~166,667 (@ 60 Hz)       | ~266,667 (@ 60 Hz)       |

The base board uses the industry-standard 10MHz speed grade — the same clock as the CPS-1, System 16, and most contemporary arcade boards. This is the most cost-effective part, requiring no premium memory chips or tight bus timing. The Plus board upgrades to the 16MHz premium speed grade, providing 60% more CPU cycles per frame for more complex game logic, AI, and per-scanline effects.

- **Architecture:** 32-bit internal registers, 16-bit external data bus
- **Address Bus:** 24-bit (16 MB addressable space)
- **Byte Order:** Big-endian (Motorola byte order) — all data on the bus is big-endian
- **Arithmetic:** Fixed-point only. No floating-point unit (FPU) or coprocessor on either board. All game math uses integer and fixed-point operations (16.16 format standard, 8.24 and 24.8 also supported).
- **Prefetch:** Enabled. The 68000 prefetches the next instruction word during execution, matching real hardware behavior.
- **Address Errors:** Enabled. Word/long-word access on odd addresses triggers an address error exception, matching real hardware behavior.

### Master Clock

All system clocks are derived from on-board crystal oscillators.

**NiXX-32 (base board):**

| Crystal | Frequency | Derived Clocks |
|---------|-----------|----------------|
| Main oscillator | 20 MHz | ÷2 = 10 MHz (M68000) |
| Audio oscillator | 8 MHz | Direct = 8 MHz (Z80) |
| NTSC colorburst | 3.579545 MHz | Direct = 3.579545 MHz (YM2151) |

The base board uses three crystals. The M68000 and Z80 clocks don't share a clean integer divisor at these frequencies, so separate oscillators are used — standard practice for cost-conscious board designs.

**NiXX-32+ (Plus board):**

| Crystal | Frequency | Derived Clocks |
|---------|-----------|----------------|
| Main oscillator | 32 MHz | ÷2 = 16 MHz (M68000), ÷4 = 8 MHz (Z80) |
| NTSC colorburst | 3.579545 MHz | Direct = 3.579545 MHz (YM2151, both chips) |

The Plus board consolidates to two crystals. The 32 MHz master divides cleanly to both the M68000 (÷2) and Z80 (÷4) clocks, eliminating the separate audio oscillator. This is a minor cost and board space improvement enabled by the higher master frequency.

### Hardware Multiply Assist (NiXX-32+ only)

The Plus board adds a hardware multiply coprocessor in the custom silicon. The CPU writes two 16-bit operands to a register pair and reads the 32-bit result back. This is faster than the 68000's native MULS instruction and accelerates scaling calculations, perspective math, and fixed-point multiplication. Comparable to the Neo Geo's hardware multiply registers.

### Interrupt Priority Levels

The NiXX-32 interrupt architecture prioritizes audio fidelity. The autonomous graphics hardware allows VBlank to sit below audio in the priority hierarchy without impacting frame rate, guaranteeing glitch-free FM synthesis and spatial audio output.

| IPL Level | Source              | Description                                         |
|-----------|---------------------|-----------------------------------------------------|
| 7 (NMI)   | Watchdog            | Non-maskable. Hardware reset if CPU locks up.       |
| 6          | Audio               | YM2151 timer interrupts. Guarantees clean audio.    |
| 5          | HBlank              | Per-scanline raster effects (palette swaps, wavy distortion, mid-screen changes). |
| 4          | VBlank              | Frame sync. Safe at this level due to autonomous graphics hardware. CPU updates sprite table and scroll registers during 38-scanline blanking window. |
| 3          | Timer               | General-purpose programmable timer for game logic.  |
| 2          | External / Link     | External IRQ. Used by Plus board linking circuitry for multiplayer. Unused on base board. |
| 1          | Software            | Lowest priority. Game-triggered interrupts for deferred work. |

### Memory Map

| Address Range         | Size   | Region              | NiXX-32+ Changes      |
|-----------------------|--------|----------------------|------------------------|
| `0x000000 - 0x1FFFFF` | 2 MB   | Game ROM             | Expanded to `0x3FFFFF` (4 MB) |
| `0x400000 - 0x4FFFFF` | 1 MB   | System RAM           | Expanded to `0x5FFFFF` (2 MB) |
| `0x600000 - 0x67FFFF` | 512 KB | Video RAM            | Expanded to `0x6FFFFF` (1 MB) |
| `0x700000 - 0x700FFF` | 4 KB   | Shared Memory (CPU/Z80) | Expanded to `0x701FFF` (8 KB) |
| `0x800000 - 0x81FFFF` | 128 KB | Sound ROM            | Expanded to `0x83FFFF` (256 KB) |
| `0xFF0000 - 0xFFFFFF` | 64 KB  | Hardware Registers   | Same                   |

### DMA Controller

#### Automatic Sprite DMA

During VBlank, the sprite controller automatically initiates a DMA transfer to copy the sprite attribute table from CPU RAM to its internal buffer. The CPU is bus-blocked during this transfer (~100 microseconds). This is hardware-initiated and requires no software setup — it happens every frame. Game code writes sprite positions and attributes to RAM during the active display period, and the hardware copies the table during blanking.

#### General-Purpose DMA

| Feature          | NiXX-32               | NiXX-32+              |
|------------------|-----------------------|------------------------|
| Channels         | 1                     | 2                      |
| Bus Width        | 16-bit                | 32-bit                 |
| Initiation       | CPU-triggered         | CPU-triggered          |

The CPU writes source address, destination address, and transfer length to DMA registers, then triggers the transfer. Useful for bulk VRAM updates (tilemap loading, palette swaps, etc.). On the Plus board, two independent channels allow simultaneous transfers (e.g., tilemap + palette), and the 32-bit internal bus roughly halves transfer time.

### VRAM Architecture

| Feature          | NiXX-32                          | NiXX-32+                         |
|------------------|----------------------------------|-----------------------------------|
| Type             | Standard VRAM                    | Dual-port VRAM                    |
| Capacity         | 512 KB                           | 1 MB                              |
| Bus Contention   | CPU and graphics hardware share access | Simultaneous read/write — CPU can write while graphics hardware reads |

#### VRAM Internal Layout

| Region          | NiXX-32       | NiXX-32+      |
|-----------------|---------------|----------------|
| Sprite RAM      | 16 KB         | 32 KB          |
| Palette RAM     | 16 KB         | 32 KB          |
| Tile RAM        | 256 KB        | 512 KB         |
| Background RAM  | 128 KB        | 256 KB         |
| **Mapped Total**| **416 KB**    | **832 KB**     |
| Reserved        | 96 KB         | 168 KB         |

Reserved space is unused VRAM resulting from standard chip sizes. Reads return open bus (`0xFF`), writes are ignored. This is typical for arcade hardware — VRAM chips come in power-of-2 sizes and the address decoder maps fixed regions within them. The unused space also provides guard regions between mapped areas.

### Memory Access Wait States

| Region             | Wait States | Notes                                    |
|--------------------|-------------|------------------------------------------|
| Game ROM           | 0           | Fast ROM access                          |
| System RAM         | 0           | Fast RAM access                          |
| Video RAM          | 1           | Slightly slower due to shared bus (base board) |
| Sound ROM          | 0           | Fast ROM access                          |
| Hardware Registers | 2           | I/O register access is slowest           |

Wait states affect M68000 bus cycle timing and real game performance. VRAM is one cycle slower than RAM, and hardware register access costs two extra cycles per read/write. The Plus board's dual-port VRAM eliminates contention wait states during active display, but the base wait state for VRAM access remains.

### Unmapped Address Behavior

Reads to unmapped address regions return `0xFF` (open bus). Writes are silently ignored. No bus error is generated. This matches the behavior of the reference hardware (CPS-2, System 32).

### Bus Arbitration (Shared Memory)

The M68000 and Z80 share a 4 KB memory region (8 KB on the Plus board) for inter-processor communication. Access is managed by request/grant bus arbitration, matching the approach used by the CPS-2 and other M68000 + Z80 arcade boards.

- **Bus Master:** M68000 (always has priority)
- **Bus Slave:** Z80 (held off during M68000 shared memory access)
- **Protocol:** When the M68000 accesses the shared region, the Z80 is automatically stalled until the access completes. When the M68000 is not accessing shared memory, the Z80 has free access. No software-level locking is required — arbitration is handled entirely by the bus controller hardware.
- **Latency:** Z80 stalls are typically 2-4 clock cycles (one M68000 bus cycle). This is invisible to game code and does not affect audio timing in practice.

### Sound ROM Banking

The Z80's 16-bit address bus limits it to 64 KB of addressable memory. Sound ROM is accessed through a 32 KB bank window at Z80 address `0x0000-0x7FFF`. The active bank is selected by writing to hardware register `0xFF3004` (SOUND_ROM_BANK_CONTROL).

| Feature       | NiXX-32         | NiXX-32+        |
|---------------|-----------------|------------------|
| Sound ROM     | 128 KB          | 256 KB           |
| Bank Window   | 32 KB           | 32 KB            |
| Total Banks   | 4               | 8                |
| Bank Register | `0xFF3004`      | `0xFF3004`       |

---

## Display

### Resolution and Timing (both boards)

- **Pixel Resolution:** 384×224 (fixed, non-configurable)
- **Display Aspect Ratio:** 4:3 (standard arcade CRT)
- **Pixel Aspect Ratio:** Non-square — pixels are wider than tall when displayed on a 4:3 monitor. This is standard for arcade hardware of the era.
- **Refresh Rate:** ~59.94 Hz (NTSC standard)
- **Total Scanlines:** 262 per frame (224 active + 38 VBlank)
- **VBlank Window:** 38 scanlines — CPU time for sprite table updates, scroll register writes, palette changes, and DMA transfers

### Color Depth

#### Color Format

| Feature              | NiXX-32                  | NiXX-32+                          |
|----------------------|--------------------------|------------------------------------|
| Format               | 15-bit RGB555            | 16-bit RGB565 (selectable to RGB555) |
| Color Space          | 32,768 possible colors   | 65,536 possible colors             |
| Bits Per Channel     | 5-5-5 (R-G-B)           | 5-6-5 (R-G-B, extra bit on green)  |
| Color Mode Register  | Fixed                    | `0xFF0002` bit 0 (0 = 15-bit, 1 = 16-bit) |

#### Palette Structure

| Feature                     | NiXX-32                | NiXX-32+              |
|-----------------------------|------------------------|-----------------------|
| Palette RAM                 | 16 KB                  | 32 KB                 |
| Total Palette Entries       | 8,192                  | 16,384                |
| Palette Banks               | 4                      | 8                     |
| Colors Per Bank             | 256                    | 512                   |
| **Addressable Per Frame**   | **1,024**              | **4,096**             |

Sprites and background tiles select a palette bank via a 4-bit field in their attribute data (bits 12-15). The hardware limits active banks to 4 (base) or 8 (Plus), though the attribute field supports up to 16 — the upper values are reserved.

#### Palette Entry Bit Layout

Each palette entry is 16 bits (2 bytes). The interpretation depends on the color mode and whether per-color priority mode is active.

**Normal Mode — RGB555 (NiXX-32 base, always):**

| Bit | 15 | 14-10 | 9-5 | 4-0 |
|-----|-----|-------|-----|-----|
| Field | Unused | Red (5-bit) | Green (5-bit) | Blue (5-bit) |

**Normal Mode — RGB565 (NiXX-32+ with color mode bit set):**

| Bit | 15-11 | 10-5 | 4-0 |
|-----|-------|------|-----|
| Field | Red (5-bit) | Green (6-bit) | Blue (5-bit) |

**Extended Mode — Per-Color Priority (NiXX-32+ only, when enabled):**

| Bit | 15 | 14 | 13-10 | 9-5 | 4-0 |
|-----|-----|-----|-------|-----|-----|
| Field | Priority override | Shadow/Highlight select | Red (4-bit) | Green (5-bit) | Blue (5-bit) |

- **Bit 15:** When set, this color is composited at a different priority than its parent sprite/tile
- **Bit 14:** 0 = shadow (darkens pixels behind), 1 = highlight (brightens pixels behind). Only meaningful when bit 15 is set.
- **Red channel reduced to 4 bits** (16 levels instead of 32) — this is the cost of encoding priority data in the palette entry

Per-color priority mode is enabled per palette bank via the video registers. Banks that don't need per-color priority retain full RGB555/565 color precision. Games typically only enable this on specific banks used for sprites or tiles that need the depth trick, keeping full color quality elsewhere.

#### Theoretical On-Screen Maximum

The addressable-per-frame count (1,024 / 4,096) represents the colors available within a single palette bank configuration. In practice, each background layer and the sprite layer can reference different palette banks independently. If all 3 layers + sprites use different banks on the base board, the theoretical visible unique colors in a single frame is up to ~4,096. On the Plus board with 4 layers + sprites using different banks, the theoretical maximum approaches ~8,192.

These theoretical maximums depend on art design and palette assignment — most games will use a subset of the available banks.

#### Palette Double-Buffering

Palette RAM is significantly larger than the active bank structure requires (8,192 entries vs 1,024 addressable on base; 16,384 vs 4,096 on Plus). The excess entries are available for **palette double-buffering** — the CPU can prepare a second set of palettes in unused RAM and swap bank pointers during VBlank for effects like:

- Fade-in / fade-out transitions
- Day/night palette cycling
- Flash effects (damage, power-ups)
- Smooth color transitions between game areas

The PaletteManager supports color cycling and fade effects using this extra capacity.

### Sprites

#### Sprite Limits

| Feature                  | NiXX-32            | NiXX-32+            |
|--------------------------|--------------------|----------------------|
| Total Sprites            | 96                 | 128                  |
| Per-Scanline Limit       | 16-20              | 24-32                |
| Maximum Sprite Size      | 32×32 pixels       | 64×64 pixels         |
| Overflow Behavior        | Hard cutoff by priority | Hard cutoff by priority |

The per-scanline limit is a **cycle budget**, not a fixed count. The sprite hardware has a finite amount of processing time per horizontal line. Smaller, unscaled sprites cost fewer cycles — a line of 8×8 sprites can fit more than a line of 32×32 scaled sprites. The range reflects this: 16 sprites at maximum complexity, up to 20 at lower complexity (base); 24-32 on the Plus board due to faster custom sprite silicon.

When the per-scanline budget is exceeded, the lowest-priority sprites on that line are dropped (not rendered). There is no flickering — the cutoff is deterministic and priority-based. Game developers are responsible for designing around the limit by avoiding excessive horizontal sprite clustering.

#### Available Sprite Sizes

| Size Code | Dimensions | NiXX-32 | NiXX-32+ |
|-----------|------------|---------|----------|
| 0         | 8×8        | Yes     | Yes      |
| 1         | 16×16      | Yes     | Yes      |
| 2         | 32×32      | Yes     | Yes      |
| 3         | 64×64      | No      | Yes      |

Sizes are fixed options (2-bit field in attributes), not arbitrary dimensions.

#### Sprite Attribute Table

Each sprite uses **16 bytes** in the sprite attribute table in VRAM. The table is automatically DMA'd from CPU RAM to the sprite controller during VBlank.

| Offset | Size    | Field         | Description                                            |
|--------|---------|---------------|--------------------------------------------------------|
| +0     | 2 bytes | X Position    | 10-bit signed horizontal position                      |
| +2     | 2 bytes | Y Position    | 10-bit signed vertical position                        |
| +4     | 2 bytes | Tile Index    | 10-bit tile reference (0-1023)                         |
| +6     | 2 bytes | Attributes    | Packed bitfield (see below)                            |
| +8     | 2 bytes | Transform     | Bit 15 = transform enabled; remaining bits reserved    |
| +10    | 2 bytes | Scale X       | 8.8 fixed-point (0x0100 = 1.0×)                       |
| +12    | 2 bytes | Rotation      | 9-bit angle (0-511 mapped to 0-359°)                  |
| +14    | 2 bytes | Scale Y       | 8.8 fixed-point (0x0100 = 1.0×)                       |

**Note:** The sprite attribute table currently uses 16 bytes per sprite with the fields above. Hot-spot positioning (see below) requires 2 additional bytes, expanding the descriptor to **18 bytes** per sprite. This fits within the existing Sprite RAM allocation (32 KB on Plus = 1,774 sprites at 18 bytes, well above the 128 sprite limit).

#### Attribute Bitfield (offset +6)

| Bits  | Field         | Description                                          |
|-------|---------------|------------------------------------------------------|
| 15-14 | Alpha         | Blend level. Base board: bit 15 only (1-bit: 0 = opaque, 1 = 50/50). Plus board: 2-bit (4 levels: 0 = opaque, 1 = 75%, 2 = 50%, 3 = 25%). |
| 13-10 | Palette       | Sub-palette select (0-15). Selects which 16-color group within the active palette bank. 4bpp tiles use 16 colors per sub-palette, so 4 bits are required to address all 16 sub-palettes within a 256-color bank. |
| 9-6   | Priority      | 4-bit priority (16 levels, 0 = lowest, 15 = highest) |
| 5-4   | Size          | Sprite size code (0-3, see table above)              |
| 3     | Flip X        | Horizontal flip                                      |
| 2     | Flip Y        | Vertical flip                                        |
| 1     | Shadow        | Shadow/highlight mode enable (see Video Output Effects) |
| 0     | Enable        | 1 = sprite active, 0 = sprite disabled               |

#### Hot-Spot Positioning (both boards)

Each sprite has a configurable origin point (hot-spot) that determines the anchor for positioning, rotation, and scaling. Stored as two signed 8-bit offsets relative to the sprite's top-left corner.

- **Hot-spot X:** Signed offset (−128 to +127) from left edge
- **Hot-spot Y:** Signed offset (−128 to +127) from top edge
- **Default:** (0, 0) = top-left corner (standard behavior)
- **Common usage:** Center of sprite for rotation around midpoint; bottom-center for character feet positioning

Setting the hot-spot to the center of a 32×32 sprite (16, 16) means the X/Y position in the attribute table refers to the sprite's center rather than its top-left corner. This simplifies rotation (sprites rotate around the hot-spot) and positioning (characters can be placed by their feet rather than their head).

#### Scaling and Rotation (both boards)

Per-sprite hardware scaling and rotation, implemented in the custom graphics silicon. Inspired by Sega's Super Scaler technology (Hang-On 1985, Out Run 1986). The CPU writes scale and rotation values to the sprite attribute table; the hardware applies a 2×2 affine transform during rendering. Rotation occurs around the sprite's hot-spot.

- **Scale format:** 8.8 fixed-point per axis. `0x0100` = 1.0× (no scaling), `0x0080` = 0.5× (half size), `0x0200` = 2.0× (double size). Independent X and Y scaling allows non-uniform stretch.
- **Rotation:** 9-bit value (0-511) mapped to 0-359 degrees. Hardware rotation around the sprite's hot-spot.
- **Transform enable:** Bit 15 of the transform word (offset +8) must be set for scaling/rotation to apply. When disabled, the sprite renders at native size with no transform overhead — this saves per-scanline processing cycles.

#### Sprite Alpha Blending

| Feature              | NiXX-32                          | NiXX-32+                          |
|----------------------|----------------------------------|------------------------------------|
| Alpha bits           | 1 (bit 15 of attributes)        | 2 (bits 15-14 of attributes)      |
| Blend levels         | 2 (opaque or 50/50)             | 4 (opaque, 75%, 50%, 25%)         |
| Blend method         | Fixed 50% mix with background   | Variable mix per level             |

On the base board, bit 15 acts as a simple on/off toggle for 50/50 blending. On the Plus board, the 2-bit field provides 4 blend levels — sufficient for ghost effects, glass, water surfaces, and fade effects. 4 alpha levels is realistic for early-90s custom silicon.

Color index 0 remains fully transparent (invisible) on both boards regardless of the alpha setting. This is standard palette-based transparency and is separate from the alpha blending feature.

#### Sprite Priority and Layering

Sprites have 16 priority levels (0-15). Higher numeric priority renders on top. Sprites can be interleaved between background layers, not just drawn above all backgrounds:

| Priority Range | Layer Position          |
|----------------|------------------------|
| 0-3            | Behind all backgrounds  |
| 4-7            | Between BG layers 0-1   |
| 8-11           | Between BG layers 1-2   |
| 12-15          | In front of all backgrounds |

This allows effects like a character walking behind a foreground building (sprite at priority 2) while projectiles fly in front of everything (sprite at priority 14).

### Background Layers

#### Layer Count and Capabilities

| Feature                  | NiXX-32                            | NiXX-32+                              |
|--------------------------|------------------------------------|----------------------------------------|
| Scroll Layers            | 4                                  | 4                                      |
| Text Plane               | Yes (fixed HUD layer)              | Yes (fixed HUD layer)                  |
| Bitmap Layer             | No                                 | Yes (raw framebuffer)                  |
| Tilemap Size             | 64×64 tiles (512×512 pixels)       | 128×64 tiles (1024×512 pixels)         |
| Tile Size                | 8×8 pixels (fixed)                 | 8×8 pixels (fixed)                     |
| Tile Color Depth         | 4 bpp (16 colors per tile)         | 4 bpp (16 colors per tile)             |
| Per-Scanline Affine      | Yes                                | Yes                                    |
| Line-Select              | No                                 | Yes (per-scanline row picking)         |
| Window Clipping          | No                                 | Yes (2 rectangular regions per layer)  |

#### Tile Format

Each tilemap entry is 16 bits (2 bytes):

| Bits  | Field      | Description                          |
|-------|------------|--------------------------------------|
| 9-0   | Tile Index | Tile reference (0-1023)              |
| 10    | Flip Y     | Vertical flip                        |
| 11    | Flip X     | Horizontal flip                      |
| 15-12 | Palette    | Palette bank select (0-3 base, 0-7 Plus) |

Tiles are 8×8 pixels at 4 bits per pixel (16 colors per tile from the selected palette bank). Each 8×8 tile occupies 32 bytes in Tile RAM.

#### Scroll Modes (both boards)

**Basic Scrolling:** Each layer has independent `scrollX` and `scrollY` registers. The hardware handles tile fetching and wrapping automatically. Wrapping is configurable per-axis per-layer — when enabled, the tilemap repeats seamlessly.

**Per-Scanline Affine (Mode 7 / Super Scaler):** Each layer supports a full 2×2 affine transformation matrix (`pa`, `pb`, `pc`, `pd`) with texture reference points (`refX`, `refY`), configurable per-scanline. This enables:

- **Perspective ground planes** — Out Run-style road rendering. Each scanline uses a different scale/offset to create the illusion of depth. The hardware includes a `setupPerspectiveGround()` helper for this common case.
- **Mode 7-style rotation** — Full ground plane rotation and scaling (F-Zero / Mario Kart style).
- **Lateral curve shifting** — Per-scanline horizontal offset creates road curves without rotation.

Matrix components use **16.16 fixed-point** math. The CPU pre-computes per-scanline parameters and writes them before the frame; the hardware reads them autonomously during rendering.

#### Line-Select (NiXX-32+ only)

Per-scanline row picking — instead of reading tilemap rows sequentially, each scanline specifies which tilemap row to display via a lookup table. This enables vertical distortion effects:

- **Hills and elevation changes** — skipping or repeating tilemap rows creates the illusion of hills rising and falling (as used in Out Run-style games).
- **Vertical compression/stretching** — squeezing or spreading rows for perspective effects.
- **Parallax within a single layer** — different vertical scroll speeds at different screen positions.

One table per layer, 224 entries (one per visible scanline). Each entry is a 9-bit row index (0-511, covering the full tilemap height). Tables are stored in BG RAM.

#### Window Clipping (NiXX-32+ only)

2 rectangular clip regions per layer. Each region is defined by left, right, top, and bottom coordinates. Within a clip region, the layer can be either **shown** or **hidden**, enabling:

- **HUD masking** — prevent a scroll layer from rendering behind the score/timer area.
- **Split-screen** — show different scroll positions in different screen regions (multiplayer).
- **Reveal effects** — progressively expose or hide layer content.

#### Text Plane (both boards)

A lightweight non-scrolling tilemap layer dedicated to HUD elements (scores, timers, health bars, text). Always rendered on top of all other layers and sprites.

- **Size:** 48×28 tiles (384×224 pixels, full screen coverage)
- **Tile format:** Same as scroll layers (8×8, 4bpp, 16-bit tilemap entries)
- **No scrolling, no affine** — fixed position, zero processing overhead
- **Tilemap size:** ~2.7 KB in BG RAM

Comparable to the Neo Geo's "fix layer." Eliminates the need to sacrifice a scroll layer for HUD elements.

#### Bitmap Layer (NiXX-32+ only)

A raw framebuffer layer with no tile constraints. Each pixel is individually addressable. Double-buffered — the CPU writes to the back buffer while the hardware displays the front buffer, then buffers are swapped during VBlank.

- **Resolution:** 384×224 pixels
- **Color depth:** 8 bpp (256 colors from a dedicated palette bank)
- **VRAM usage:** ~168 KB (two 84 KB framebuffers for double-buffering)
- **Priority:** Configurable — can sit behind tile layers (pre-rendered backgrounds) or in front (overlay effects)
- **No scrolling, no affine** — fixed full-screen framebuffer
- **Buffer swap:** Hardware register triggers front/back buffer swap during VBlank

Enables effects that tile-based layers cannot achieve: pre-rendered artwork, irregular shapes, full-screen fades, software-rendered graphics. The most VRAM-intensive feature on the Plus board.

#### Layer Compositing Order

Layers are composited from back to front. Sprites are interleaved via their 16-level priority system.

| Priority (back to front) | Layer                                          |
|--------------------------|------------------------------------------------|
| Lowest                   | Bitmap layer (Plus only, priority configurable) |
| ↑                        | Background layers 0-3 (configurable order via per-layer 8-bit priority) |
| ↑                        | Sprites (interleaved between BG layers via sprite priority 0-15) |
| Highest                  | Text plane (always on top)                      |

The bitmap layer's priority is configurable — it can be moved above or below the tile scroll layers depending on the desired effect.

#### Background RAM Budget

**NiXX-32 (128 KB available):**

| Usage                        | Size     |
|------------------------------|----------|
| 4 scroll layer tilemaps × 8 KB | 32 KB  |
| Text plane tilemap           | ~2.7 KB  |
| **Total used**               | **~35 KB** |
| Reserved (chip granularity)  | ~93 KB   |

**NiXX-32+ (256 KB available):**

| Usage                              | Size     |
|------------------------------------|----------|
| 4 scroll layer tilemaps × 16 KB        | 64 KB    |
| Line-select tables (4 × ~450 bytes)    | ~2 KB    |
| Text plane tilemap                      | ~2.7 KB  |
| Bitmap framebuffer (double-buffered)    | ~168 KB  |
| **Total used**                          | **~237 KB** |
| Reserved                                | ~19 KB   |

### Video Output Effects

Visual effects on the NiXX-32 are achieved through a combination of hardware registers in the video output stage, color math in the video mixer, and CPU-driven palette manipulation — not a dedicated effects chip. This matches how real arcade hardware of the era handled visual effects.

#### Hardware Effect Registers (both boards)

| Register          | Address    | Bits | Effect                                    |
|-------------------|------------|------|-------------------------------------------|
| Brightness R      | `0xFF0004` | 8    | Red channel brightness (0 = off, 128 = normal, 255 = max) |
| Brightness G      | `0xFF0005` | 8    | Green channel brightness                   |
| Brightness B      | `0xFF0006` | 8    | Blue channel brightness                    |
| Mosaic            | `0xFF0008` | 8    | Pixel block size (bits 7-4 = vertical, bits 3-0 = horizontal). 0 = off, up to 15 = 16×16. |
| Scanline Mode     | `0xFF000A` | 8    | Alternating scanline darkening intensity (0 = off, 255 = maximum) |
| Shadow/Highlight  | `0xFF000C` | 8    | Shadow/highlight mode control (see below)  |

These registers are in the video DAC output path and apply after all layer compositing is complete.

#### Per-Component Color Brightness (both boards)

Three independent 8-bit brightness registers provide per-channel RGB control. Unlike the Sega System 32's 7 levels, the NiXX-32 provides a full 8-bit range (256 levels) per component, enabling:

- **Color tinting** — reduce blue/green for warm sunset tone, reduce red/green for underwater blue tint
- **Damage flash** — spike the red channel briefly
- **Fade to any color** — not just black/white, but fade to red, blue, sepia, etc.
- **Day/night transitions** — gradually shift color temperature across all three channels

Setting all three channels to 128 produces normal output. Setting all to 0 fades to black. Independent control per channel enables tinting effects that a single global brightness register cannot achieve.

#### Shadow & Highlight Mode (both boards)

When a sprite or tile has the shadow bit set (bit 1 of sprite attributes, or via a reserved palette index for tiles), the video mixer modifies the pixels *behind* that element rather than drawing the element's own pixels:

- **Shadow:** Pixels behind the shadow element are darkened by a fixed amount (approximately 50% brightness reduction). Used for character drop shadows, darkened areas under platforms, or shadow casting.
- **Highlight:** Pixels behind the highlight element are brightened. Used for spotlight effects, glowing objects, or specular highlights on water.

The shadow/highlight register (`0xFF000C`) controls the mode:
- Bit 0: Enable shadow/highlight processing
- Bits 3-1: Shadow intensity (0 = subtle, 7 = maximum darkening)
- Bits 6-4: Highlight intensity (0 = subtle, 7 = maximum brightening)

This is a color math operation in the video mixer — no additional VRAM or sprites are consumed. Comparable to the Sega Genesis/System 32 shadow & highlight mode.

#### Dynamic Priorities (NiXX-32+ only)

The Plus board's video mixer supports changing layer and sprite priority mid-frame via HBlank interrupt writes to priority registers. This allows:

- **Split-screen priority changes** — different layer ordering in the top and bottom halves of the screen
- **Per-scanline sprite-to-background ordering** — a sprite can be behind a layer on one scanline and in front on another
- **Depth-based compositing** — simulate z-ordering changes across the screen height (useful for pseudo-3D scenes)

On the base board, priorities are fixed for the entire frame and can only be updated during VBlank.

#### Per-Color Priority (NiXX-32+ only)

Individual palette entries on the Plus board can have a priority override bit. When set, pixels using that color index are composited at a different priority than their parent sprite or tile. This enables:

- **Transparent cockpit windows** — the glass color passes behind a background layer while the cockpit frame stays in front
- **Selective masking** — specific colors within a sprite can hide behind scenery while the rest of the sprite stays visible
- **Depth tricks** — parts of a single tile appearing at different depths without splitting into multiple sprites

Controlled via the upper bits of the palette entry in Palette RAM (Plus board interprets the 16-bit color entry differently when per-color priority mode is enabled).

#### Palette-Based Effects (CPU-driven, both boards)

The following effects are achieved through palette RAM manipulation during VBlank. No additional hardware is required — the CPU writes modified palette entries to Palette RAM:

- **Fade to black / white** — progressively darken or brighten all palette entries over several frames
- **Screen flash** — write a bright palette set, then restore the original over 2-3 frames (damage hit, explosion)
- **Color cycling** — rotate a range of palette entries each frame to animate water, fire, lights, neon signs without changing tile data
- **Palette swap** — switch palette bank pointers instantly for alternate color schemes (day/night, power-up states, character recolors)
- **Gradient effects** — write computed palette ramps for sky gradients, underwater tinting

Palette double-buffering (using the excess Palette RAM beyond the active bank structure) enables smooth transitions by preparing the next palette set while the current one is displayed.

#### Effects via Existing Graphics Systems

- **Particles** — implemented using regular sprites. The game allocates sprite slots for smoke, sparks, debris, and manages their position/lifetime in software. This is standard for arcade hardware.
- **Wave distortion** — per-scanline horizontal scroll offsets via the affine system. Creates heat shimmer, underwater waviness, or screen shake.
- **Raster effects** — per-scanline palette changes or scroll position changes via HBlank interrupts. Color gradient skies, split-screen scroll speeds.

---

## Co-Processor

### Zilog Z80H @ 8 MHz (both boards)

Dedicated audio coprocessor. Handles all sound driver execution — no game logic runs on the Z80. The Z80H is the 8 MHz speed-binned variant of the Zilog NMOS Z80, behaviorally identical to the original but rated for higher clock speeds.

- **Architecture:** 8-bit CPU, 16-bit address bus (64 KB addressable)
- **Role:** Audio processing only — drives YM2151 FM synthesis, PCM sample playback, and QSound spatial audio (Plus board only)
- **Interrupt Mode:** IM 2 (vectored interrupts). Each interrupt source (YM2151 timers, PCM buffer, main CPU sync) has its own vector in a jump table pointed to by the Z80's I register. This allows direct dispatch to per-source handlers without polling status registers.

### Communication with M68000

The Z80 and M68000 communicate exclusively through shared RAM. The M68000 writes command bytes and parameters to the shared memory region; the Z80 reads and processes them. Synchronization is managed via I/O port `0x08` (audio sync trigger).

- **Shared Memory (Z80 view):** `0xD000-0xDFFF` (4 KB)
- **Shared Memory (M68000 view):** `0x700000-0x700FFF` (base) / `0x700000-0x701FFF` (Plus, 8 KB)

### Z80 Memory Map (64 KB)

| Address Range     | Size  | NiXX-32                | NiXX-32+                      |
|-------------------|-------|------------------------|-------------------------------|
| `0x0000-0x7FFF`   | 32 KB | Sound ROM              | Sound ROM (banked)            |
| `0x8000-0x9FFF`   | 8 KB  | Audio RAM              | 16 KB Audio RAM (to `0xBFFF`) |
| `0xA000-0xBFFF`   | 8 KB  | Unmapped               | (part of Audio RAM)           |
| `0xC000-0xCFFF`   | 4 KB  | Audio Hardware Registers | Audio Hardware Registers    |
| `0xD000-0xDFFF`   | 4 KB  | Shared Memory (CPU/Z80) | Shared Memory (CPU/Z80)     |
| `0xE000-0xFFFF`   | 8 KB  | Unmapped               | Unmapped                      |

### Z80 I/O Port Map

#### YM2151 FM Synthesis (both boards)

| Port   | Direction | Function         |
|--------|-----------|------------------|
| `0x00` | Write     | Register select  |
| `0x01` | Write     | Data write       |
| `0x02` | Read      | Status read      |

#### NiXX-B DSP PCM (NiXX-32 only)

| Port   | Direction | Function             |
|--------|-----------|----------------------|
| `0x04` | Write     | Sample data (low)    |
| `0x05` | Write     | Sample data (high)   |
| `0x06` | R/W       | Playback control     |

#### System Control (both boards)

| Port   | Direction | Function              |
|--------|-----------|------------------------|
| `0x08` | Write     | Audio sync trigger     |
| `0x0A` | R/W       | Interrupt control      |
| `0x0C` | Write     | Timing divider         |

#### QSound (NiXX-32+ only, replaces NiXX-B DSP)

| Port   | Direction | Function          |
|--------|-----------|-------------------|
| `0x10` | Write     | Address low byte  |
| `0x11` | Write     | Address high byte |
| `0x12` | R/W       | Data register     |
| `0x13` | Write     | Control register  |

#### Second YM2151 (NiXX-32+ only)

| Port   | Direction | Function         |
|--------|-----------|------------------|
| `0x20` | Write     | Register select  |
| `0x21` | Write     | Data write       |
| `0x22` | Read      | Status read      |

### Z80 Interrupt Sources

| Source          | Bit Mask | Description                              |
|-----------------|----------|------------------------------------------|
| Audio Timer     | 1        | Periodic audio timing interrupt          |
| YM2151 Timer A  | 2        | YM2151 Timer A overflow                  |
| YM2151 Timer B  | 4        | YM2151 Timer B overflow                  |
| PCM Buffer      | 8        | PCM buffer empty — refill needed         |
| Main CPU Sync   | 16       | M68000 sync request (command available)  |
| External IRQ    | 32       | External interrupt                       |

### Audio Timing

| Parameter            | NiXX-32                    | NiXX-32+                   |
|----------------------|----------------------------|-----------------------------|
| YM2151 Chips         | 1                          | 2                           |
| YM2151 Clock         | 3.579545 MHz               | 3.579545 MHz (each)         |
| FM Voices            | 8                          | 16                          |
| PCM Engine           | NiXX-B DSP                 | QSound (DL-1425)            |
| PCM Channels         | 8                          | 16 PCM + 3 ADPCM           |
| PCM Bit Depth        | 8-bit                      | 16-bit                      |
| PCM Sample Rate      | 11,025 Hz                  | 24,000 Hz (QSound native)   |
| Audio Output Rate    | 44,100 Hz                  | 44,100 Hz                   |
| Audio Output         | Stereo                     | Stereo                      |
| Audio Buffer Size    | 512 samples                | 512 samples                 |
| Target Latency       | ~20 ms                     | ~20 ms                      |
| Z80 Cycles Per Frame | ~133,333 (@ 8 MHz, 60 Hz)  | ~133,333 (@ 8 MHz, 60 Hz)  |

---

## Audio

The NiXX-32 audio architecture combines FM synthesis with sample-based PCM playback, driven entirely by the Z80 coprocessor. The interrupt priority system (audio at IPL 6, above VBlank at IPL 4) guarantees glitch-free audio output — the autonomous graphics hardware allows this without impacting frame rate.

Sound designers can lean heavily on PCM for a Capcom CPS-2-style sound profile, use FM synthesis for a classic late-80s aesthetic, or blend both. The hardware provides both options — the creative direction is up to the developer.

### FM Synthesis

| Feature            | NiXX-32                  | NiXX-32+                  |
|--------------------|--------------------------|----------------------------|
| Chip               | 1× Yamaha YM2151 (OPM)  | 2× Yamaha YM2151 (OPM)    |
| FM Voices          | 8                        | 16 (8 per chip)            |
| Clock              | 3.579545 MHz             | 3.579545 MHz (each)        |
| Operators Per Voice | 4                       | 4                          |
| Algorithms         | 8 (0-7)                  | 8 (0-7)                    |
| Feedback           | Per-channel configurable  | Per-channel configurable   |
| LFO                | Yes                      | Yes                        |
| Noise Generator    | Yes (channel 8)          | Yes (channel 8 on each chip) |
| Timers             | Timer A + Timer B        | Timer A + Timer B (each chip) |
| Output             | Stereo                   | Stereo                     |

The YM2151 generates sound algorithmically from operator parameters — no sample data required. FM voices consume zero Sound ROM space, making them ideal for sustained pads, bass lines, and sound effects where ROM budget is tight.

On the Plus board, the second YM2151 is independently addressable via separate Z80 I/O ports (`0x20-0x22`). Both chips share the same clock. The Z80 sound driver manages voice allocation across both chips transparently.

### PCM Sound — NiXX-B DSP (NiXX-32 only)

The NiXX-B DSP is a custom sample playback chip on the base board. It handles all PCM audio — drums, voice clips, sampled instruments, and sound effects.

| Feature            | Specification            |
|--------------------|--------------------------|
| Channels           | 8 simultaneous           |
| Bit Depth          | 8-bit PCM                |
| Sample Rate        | 11,025 Hz                |
| Output             | Stereo                   |
| Control            | Z80 via I/O ports `0x04-0x06` |

At 8-bit 11kHz, one second of mono audio is ~11 KB. The 128 KB Sound ROM (NXS-2000) provides a base sample library, and additional game-specific samples can be loaded from Game ROM into audio RAM via the Z80. Careful ROM budgeting allows full sample-based music on the base board, though most developers will use FM for melodic content and PCM for percussive and spot-effect sounds.

### PCM Sound — QSound (NiXX-32+ only)

On the Plus board, the QSound chip (Capcom DL-1425) **replaces** the NiXX-B DSP entirely. QSound is both a PCM playback engine and a spatial audio processor — it handles all sample-based audio with built-in 3D positioning.

| Feature            | Specification            |
|--------------------|--------------------------|
| PCM Channels       | 16 simultaneous          |
| ADPCM Channels     | 3 (one-shot, ~4:1 compression) |
| Total Voices       | 19                       |
| PCM Bit Depth      | 16-bit                   |
| Native Sample Rate | 24,000 Hz                |
| Output             | Stereo                   |
| Spatial Audio      | Per-voice 3D positioning, HRTF processing |
| Effects            | Reverb, echo, Doppler, FIR filtering |
| Control            | Z80 via I/O ports `0x10-0x13` |

QSound's 16 PCM channels at 16-bit provide sufficient capacity for full sample-based music (comparable to the CPS-2's audio profile) plus sound effects and voice clips simultaneously. The ADPCM channels compress audio ~4:1, extending effective Sound ROM capacity for one-shot samples.

#### QSound Spatial Audio

Each QSound voice can be independently positioned in a virtual 3D sound field. The Z80 writes position parameters and the QSound chip applies HRTF (head-related transfer function) processing to create the illusion of spatial depth through stereo speakers. Features include:

- **Per-voice 3D positioning** — place sounds anywhere in the stereo field
- **Doppler effect** — automatic pitch shifting based on velocity
- **Distance attenuation** — volume decreases with virtual distance
- **Reverb and echo** — configurable room simulation
- **FIR filtering** — frequency shaping per voice

Spatial audio is optional per voice — channels can be used as straight stereo PCM without spatial processing.

### Sound ROM and Sample Storage

| Feature              | NiXX-32               | NiXX-32+              |
|----------------------|-----------------------|-----------------------|
| Sound ROM            | 128 KB                | 256 KB                |
| Base Sample Library  | NXS-2000              | NXS-2001              |
| Z80 Bank Window      | 32 KB (4 banks)       | 32 KB (8 banks)       |
| Bank Register        | `0xFF3004`            | `0xFF3004`            |

The Sound ROM contains the board's **base sample library** — a curated set of common drum hits, instrument samples, and sound effects shared across all games on that board revision. Game-specific samples beyond the base library are stored in the **Game ROM** (2 MB base / 4 MB Plus) and loaded into audio RAM by the Z80 during game initialization or level transitions.

This two-tier approach (board sample library + game-specific samples) maximizes the effective sample budget:
- **NXS-2000** (base): General-purpose 8-bit sample library optimized for the NiXX-B DSP
- **NXS-2001** (Plus): Expanded 16-bit sample library optimized for QSound, including higher-quality versions of NXS-2000 content plus spatial audio test samples

### Audio Mixing and Output

| Feature              | NiXX-32               | NiXX-32+              |
|----------------------|-----------------------|-----------------------|
| FM Source            | 1× YM2151 (stereo)   | 2× YM2151 (stereo)    |
| PCM Source           | NiXX-B DSP (stereo)  | QSound (stereo)        |
| Mix Method           | Hardware mixer        | Hardware mixer         |
| Output Sample Rate   | 44,100 Hz             | 44,100 Hz              |
| Output Bit Depth     | 16-bit                | 16-bit                 |
| Output Channels      | 2 (stereo)            | 2 (stereo)             |

The hardware mixer combines FM and PCM sources into a final stereo output. Per-source volume levels are configurable via mixer registers, allowing the sound driver to balance FM against PCM dynamically (e.g., ducking music during voice clips).

---

## Input

### Cabinet Input Configuration (both boards)

The NiXX-32 is a general-purpose arcade board — the input system supports a variety of cabinet configurations from racing wheels to standard joystick panels. The board defines the maximum input capability; individual cabinets wire up the controls they need.

#### Per-Player Inputs (active low)

| Input              | Bits | Description                                          |
|--------------------|------|------------------------------------------------------|
| Digital Directional | 4   | Up, Down, Left, Right (8-way joystick or D-pad)     |
| Analog Axis        | 8   | 8-bit analog input (0-255). Used for steering wheels, paddles, or throttle levers. Center = 128. |
| Button 1           | 1   | Primary action (e.g., accelerate)                    |
| Button 2           | 1   | Secondary action (e.g., brake)                       |
| Button 3           | 1   | Tertiary action (e.g., attack)                       |
| Button 4           | 1   | Quaternary action (e.g., alt attack)                 |

2 player inputs supported simultaneously. Each player has the full input set above.

#### System Inputs (active low)

| Input              | Description                                          |
|--------------------|------------------------------------------------------|
| Coin 1             | Player 1 coin slot                                   |
| Coin 2             | Player 2 coin slot                                   |
| Start 1            | Player 1 start button                                |
| Start 2            | Player 2 start button                                |
| Service            | Service credit button (inside cabinet)               |
| Test               | Enter test/diagnostic mode                           |
| Tilt               | Tilt sensor (anti-tampering)                         |

#### DIP Switches

2 banks of 8 switches (16 total). Used for game configuration without software changes:

- Difficulty level
- Number of lives / continues
- Coin value (1 coin = 1 credit, 2 coins = 1 credit, etc.)
- Demo sound on/off
- Cabinet type (upright, cocktail, dedicated)
- Free play enable

DIP switch settings are read-only from the M68000 via input registers at `0xFF2006` and `0xFF2008`.

#### Cabinet Configurations

The same board supports different cabinet wiring:

| Cabinet Type     | Movement Input         | Buttons Used       | Game Genre          |
|------------------|------------------------|--------------------|---------------------|
| Racing (Telefunk) | Analog steering + digital U/D | 4 (accel, brake, attack, alt) | Pseudo-3D racing |
| Joystick (standard) | 8-way digital joystick | 2-4              | Beat-em-up, platformer, shooter |
| Fighting         | 8-way digital joystick | 4                  | Fighting games      |
| Paddle           | Analog axis            | 1-2                | Puzzle, breakout    |

The game reads whichever inputs the cabinet provides. An analog steering wheel maps to the analog axis register; a digital joystick maps to the directional bits. Games should support both where practical for wider cabinet compatibility.

---

## Multiplayer Link (NiXX-32+ only)

The Plus board includes dedicated linking circuitry for cabinet-to-cabinet multiplayer (e.g., Telefunk 2 two-player racing). The base board has no multiplayer support.

- **Connection:** Serial link between two cabinets
- **Protocol:** Shared game state — player positions, game events, synchronization signals
- **Interrupt:** External/Link interrupt (IPL 2) fires when data arrives from the linked cabinet
- **Topology:** Point-to-point (two cabinets only, no daisy-chaining)
- **Use Case:** Split-screen or head-to-head competitive play across two monitors

The link system is not yet implemented in the emulation layer. The IPL 2 interrupt and hardware register space are reserved for future implementation.

---

## Anti-Piracy

### Board-Level Protection (in-lore)

The NiXX-32 boards use era-appropriate hardware protection to prevent ROM piracy and unauthorized cloning.

**NiXX-32 (1989) — Battery-Backed Encryption:**

Encryption keys are stored in battery-backed SRAM on the board. The custom silicon decrypts game code and graphics data at runtime using these keys. If the battery is removed or dies, the keys are lost and the board becomes non-functional ("suicide" protection). This matches the approach used by the Capcom CPS-2 and other boards of the era.

- Game ROM data is stored encrypted on the ROM chips
- Decryption happens in real-time by the custom graphics/CPU silicon
- Each board has a unique key set programmed during manufacturing
- Battery life: approximately 5-8 years under normal conditions

**NiXX-32+ (1992) — Enhanced Encryption with Boot Verification:**

The Plus board improves on the base board's protection with an additional boot verification step. On power-up, the board verifies a cryptographic signature in the ROM header before allowing execution. This prevents modified ROMs from running even if the encryption is bypassed.

- Same battery-backed key system as the base board
- Additional boot-time ROM header verification
- Stronger encryption algorithm (longer key length)
- Tamper-detection circuitry on the board PCB

### Modern Emulation-Layer Security (ROM Signing)

For games developed today using the NiXX-32 platform, ROM signing provides modern content integrity verification separate from the in-lore board protection.

- **Algorithm:** RSA-2048 with PKCS#1 v1.5 padding
- **Hash:** SHA-256 for content integrity
- **ROM Header:** 1024-byte header containing signature, hash, developer ID, game title, timestamp, and hardware version
- **Tooling:** `rom-signer` tool for signing, verification, and key generation
- **Validation Pipeline:** Magic number → version check → CRC32 → SHA-256 → RSA signature → hardware compatibility

ROM signing is an emulation-layer feature — it ensures that ROM files distributed for the emulator have not been tampered with. It is separate from the in-lore battery-backed encryption that the physical boards would have used.

---

## ROM Configuration

### Physical ROM Chips

On the physical PCB, ROM data is stored across separate mask ROM chips:

| ROM | NiXX-32 | NiXX-32+ | Memory Address |
|-----|---------|----------|----------------|
| Game ROM | 2 MB | 4 MB | `0x000000` — flat, no banking |
| Sound ROM | 128 KB | 256 KB | `0x800000` — banked (32 KB window) |
| System BIOS | On-board | On-board | Executes at power-on before game ROM |

**Game ROM** is directly addressable by the M68000's 24-bit address bus — no banking required. The full 2 MB (base) or 4 MB (Plus) is visible as a flat address space.

**Sound ROM** is accessed by the Z80 through a 32 KB bank window (see Sound ROM Banking in the Memory section).

**System BIOS** handles power-on diagnostics, RAM tests, input test mode, and DIP switch configuration. It executes before transferring control to the game ROM. Standard for arcade hardware of the era.

### Region Support

ROMs carry region metadata (USA, Japan, Europe, World) in their header. Region is informational only — there is no hardware-level region locking. Any ROM runs on any board regardless of region tag.

---

## Hardware Register Map (`0xFF0000 - 0xFFFFFF`)

The 64 KB hardware register region is divided into functional blocks. All registers are memory-mapped and accessible by the M68000 via normal read/write operations. Hardware register access incurs 2 wait states per access.

### Register Block Layout

| Address Range           | Size  | Block           | Description                    |
|-------------------------|-------|-----------------|--------------------------------|
| `0xFF0000 - 0xFF0FFF`   | 4 KB  | Video Registers | Graphics control and effects   |
| `0xFF1000 - 0xFF1FFF`   | 4 KB  | Audio Registers | Audio mixer and control        |
| `0xFF2000 - 0xFF2FFF`   | 4 KB  | Input Registers | Player controls and coin input |
| `0xFF3000 - 0xFF3FFF`   | 4 KB  | System Registers| Banking, timers, watchdog      |
| `0xFF4000 - 0xFF4FFF`   | 4 KB  | DMA Registers   | DMA controller configuration   |
| `0xFF5000 - 0xFF5FFF`   | 4 KB  | Sprite Registers| Sprite controller settings     |
| `0xFF6000 - 0xFF6FFF`   | 4 KB  | Background Registers | Layer control and scroll  |
| `0xFF7000 - 0xFF7FFF`   | 4 KB  | Reserved        | Future expansion               |
| `0xFF8000 - 0xFF8FFF`   | 4 KB  | Communication   | Inter-CPU communication bus    |
| `0xFF9000 - 0xFFEFFF`   | 24 KB | Reserved        | Future expansion               |
| `0xFFF000 - 0xFFFFFF`   | 4 KB  | Link Registers  | Multiplayer link (Plus only)   |

### Video Registers (`0xFF0000 - 0xFF0FFF`)

| Address    | Size | R/W | Register            | Description                                         |
|------------|------|-----|---------------------|-----------------------------------------------------|
| `0xFF0000` | 2    | R/W | Display Control     | Bit 0: display enable. Bit 1: interlace mode.       |
| `0xFF0002` | 2    | R/W | Color Mode          | Bit 0: 0 = 15-bit RGB555, 1 = 16-bit RGB565 (Plus only) |
| `0xFF0004` | 1    | R/W | Brightness R        | Red channel brightness (0 = off, 128 = normal, 255 = max) |
| `0xFF0005` | 1    | R/W | Brightness G        | Green channel brightness                            |
| `0xFF0006` | 1    | R/W | Brightness B        | Blue channel brightness                             |
| `0xFF0008` | 1    | R/W | Mosaic              | Bits 7-4: vertical block size, bits 3-0: horizontal |
| `0xFF000A` | 1    | R/W | Scanline Darkening  | Alternating scanline intensity (0 = off, 255 = max) |
| `0xFF000C` | 1    | R/W | Shadow/Highlight    | Bit 0: enable, bits 3-1: shadow intensity, bits 6-4: highlight intensity |
| `0xFF000E` | 2    | R   | VBlank Status       | Bit 0: 1 during VBlank, 0 during active display    |
| `0xFF0010` | 2    | R   | HBlank Status       | Bit 0: 1 during HBlank                              |
| `0xFF0012` | 2    | R   | Current Scanline    | Current scanline number (0-261)                      |

### Audio Registers (`0xFF1000 - 0xFF1FFF`)

| Address    | Size | R/W | Register            | Description                                         |
|------------|------|-----|---------------------|-----------------------------------------------------|
| `0xFF1000` | 2    | R/W | Audio Control       | Bit 0: audio enable, bit 1: FM enable, bit 2: PCM enable |
| `0xFF1002` | 1    | R/W | Master Volume       | Global volume (0-255)                               |
| `0xFF1004` | 1    | R/W | FM Volume           | YM2151 mix level (0-255)                            |
| `0xFF1006` | 1    | R/W | PCM Volume          | NiXX-B DSP / QSound mix level (0-255)               |
| `0xFF1008` | 1    | R/W | FM Balance          | FM left/right balance (0 = left, 128 = center, 255 = right) |
| `0xFF100A` | 1    | R/W | PCM Balance         | PCM left/right balance                              |

### Input Registers (`0xFF2000 - 0xFF2FFF`)

| Address    | Size | R/W | Register            | Description                                         |
|------------|------|-----|---------------------|-----------------------------------------------------|
| `0xFF2000` | 2    | R   | Player 1 Controls   | Directional + buttons (active low)                  |
| `0xFF2002` | 2    | R   | Player 2 Controls   | Directional + buttons (active low)                  |
| `0xFF2004` | 2    | R   | System Inputs       | Coin, start, service, test buttons                  |
| `0xFF2006` | 2    | R   | DIP Switch Bank 1   | Configuration switches (active low)                 |
| `0xFF2008` | 2    | R   | DIP Switch Bank 2   | Configuration switches (active low)                 |

### System Registers (`0xFF3000 - 0xFF3FFF`)

| Address    | Size | R/W | Register               | Description                                      |
|------------|------|-----|------------------------|--------------------------------------------------|
| `0xFF3000` | 2    | R/W | ROM Bank Control       | Game ROM banking (for ROMs > visible window)     |
| `0xFF3004` | 2    | R/W | Sound ROM Bank Control | Z80 sound ROM bank select (4 banks base, 8 Plus) |
| `0xFF3008` | 2    | R/W | Complex Bank Control   | Multi-level banking (Plus only)                  |
| `0xFF3010` | 2    | R/W | Timer Control          | General-purpose timer enable, mode, prescaler    |
| `0xFF3012` | 2    | R/W | Timer Reload           | Timer reload value                               |
| `0xFF3014` | 2    | R   | Timer Counter          | Current timer count (read-only)                  |
| `0xFF3020` | 2    | W   | Watchdog               | Write any value to reset watchdog timer          |
| `0xFF3030` | 2    | R   | Hardware ID            | Board identification (0x3200 = base, 0x3201 = Plus) |
| `0xFF3032` | 2    | R   | Hardware Revision      | PCB revision number                              |

### DMA Registers (`0xFF4000 - 0xFF4FFF`)

#### Channel 0 (both boards)

| Address    | Size | R/W | Register            | Description                                         |
|------------|------|-----|---------------------|-----------------------------------------------------|
| `0xFF4000` | 4    | R/W | DMA0 Source         | Source address (24-bit)                             |
| `0xFF4004` | 4    | R/W | DMA0 Destination    | Destination address (24-bit)                        |
| `0xFF4008` | 2    | R/W | DMA0 Length         | Transfer length in words                            |
| `0xFF400A` | 2    | R/W | DMA0 Control        | Bit 0: start, bit 1: busy (read), bit 2: interrupt on complete |
| `0xFF400C` | 2    | R   | DMA0 Status         | Bit 0: active, bit 1: complete, bit 2: error       |

#### Channel 1 (NiXX-32+ only)

| Address    | Size | R/W | Register            | Description                                         |
|------------|------|-----|---------------------|-----------------------------------------------------|
| `0xFF4010` | 4    | R/W | DMA1 Source         | Source address (24-bit)                             |
| `0xFF4014` | 4    | R/W | DMA1 Destination    | Destination address (24-bit)                        |
| `0xFF4018` | 2    | R/W | DMA1 Length         | Transfer length in words                            |
| `0xFF401A` | 2    | R/W | DMA1 Control        | Same as DMA0 Control                                |
| `0xFF401C` | 2    | R   | DMA1 Status         | Same as DMA0 Status                                 |

### Sprite Registers (`0xFF5000 - 0xFF5FFF`)

| Address    | Size | R/W | Register               | Description                                      |
|------------|------|-----|------------------------|--------------------------------------------------|
| `0xFF5000` | 2    | R/W | Sprite Control         | Bit 0: sprite enable, bit 1: auto-DMA enable    |
| `0xFF5002` | 4    | R/W | Sprite Table Address   | Source address for sprite DMA (in main RAM)      |
| `0xFF5006` | 2    | R   | Sprite DMA Status      | Bit 0: DMA active, bit 1: DMA complete          |
| `0xFF5008` | 2    | R   | Sprite Overflow Status  | Bit 0: overflow occurred, bits 15-8: scanline   |

### Background Registers (`0xFF6000 - 0xFF6FFF`)

#### Per-Layer Registers (repeated for layers 0-3, offset +0x40 per layer)

Base addresses: Layer 0 = `0xFF6000`, Layer 1 = `0xFF6040`, Layer 2 = `0xFF6080`, Layer 3 = `0xFF60C0`

| Offset | Size | R/W | Register            | Description                                         |
|--------|------|-----|---------------------|-----------------------------------------------------|
| +0x00  | 2    | R/W | Layer Control       | Bit 0: enable, bit 1: affine enable, bit 2: wrap X, bit 3: wrap Y |
| +0x02  | 1    | R/W | Layer Priority      | 8-bit priority (0-255, higher = in front)           |
| +0x04  | 2    | R/W | Scroll X            | Horizontal scroll position                          |
| +0x06  | 2    | R/W | Scroll Y            | Vertical scroll position                            |
| +0x08  | 4    | R/W | Tilemap Base        | Base address of tilemap data in BG RAM              |
| +0x0C  | 1    | R/W | Palette             | Palette bank for this layer                         |

#### Global Background Registers

| Address    | Size | R/W | Register               | Description                                      |
|------------|------|-----|------------------------|--------------------------------------------------|
| `0xFF6200` | 2    | R/W | Text Plane Control     | Bit 0: enable text plane                         |
| `0xFF6204` | 4    | R/W | Text Plane Tilemap Base| Base address of text plane tilemap in BG RAM     |
| `0xFF6210` | 2    | R/W | Bitmap Control         | Bit 0: enable (Plus only), bit 1: buffer select  |
| `0xFF6212` | 1    | R/W | Bitmap Priority        | Bitmap layer priority value                      |
| `0xFF6214` | 4    | R/W | Bitmap Buffer 0 Base   | Front buffer address in BG RAM (Plus only)       |
| `0xFF6218` | 4    | R/W | Bitmap Buffer 1 Base   | Back buffer address in BG RAM (Plus only)        |
| `0xFF6220` | 2    | R/W | Window Clip 0 Left     | Window region 0 left edge (Plus only)            |
| `0xFF6222` | 2    | R/W | Window Clip 0 Right    | Window region 0 right edge                       |
| `0xFF6224` | 2    | R/W | Window Clip 0 Top      | Window region 0 top edge                         |
| `0xFF6226` | 2    | R/W | Window Clip 0 Bottom   | Window region 0 bottom edge                      |
| `0xFF6228` | 2    | R/W | Window Clip 0 Mode     | Bit 0: enable, bit 1: 0 = show inside / 1 = hide inside |
| `0xFF6230` | 10   | R/W | Window Clip 1          | Same layout as Window Clip 0 (Plus only)         |

### Communication Registers (`0xFF8000 - 0xFF8FFF`)

| Address    | Size | R/W | Register               | Description                                      |
|------------|------|-----|------------------------|--------------------------------------------------|
| `0xFF8000` | 256  | R/W | CPU 0 Comm Block       | M68000 communication mailbox                     |
| `0xFF8100` | 256  | R/W | CPU 1 Comm Block       | Z80 communication mailbox                        |

Used by the inter-CPU communication bus for interrupt signaling and sync events beyond the shared memory region.

### Link Registers (`0xFFF000 - 0xFFFFFF`, NiXX-32+ only)

| Address    | Size | R/W | Register               | Description                                      |
|------------|------|-----|------------------------|--------------------------------------------------|
| `0xFFF000` | 2    | R/W | Link Control           | Bit 0: link enable, bit 1: master/slave          |
| `0xFFF002` | 2    | R   | Link Status            | Bit 0: connected, bit 1: data ready, bit 2: error |
| `0xFFF004` | 2    | R/W | Link TX Data           | Transmit data register                           |
| `0xFFF006` | 2    | R   | Link RX Data           | Receive data register                            |
| `0xFFF008` | 2    | R/W | Link Interrupt Control | Bit 0: enable IRQ on data received               |

Reserved on the base board. Reads return `0xFF`, writes are ignored.

