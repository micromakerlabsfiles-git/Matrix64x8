# OSPREY-64x8: Vehicle Rooftop Visual Matrix & Companion Pro Studio

**OSPREY-64x8** is a standalone, high-performance, production-ready visual display firmware and single-file companion Web Serial Studio for an ESP32 / ESP32-C3 / ESP32-S3 driving an 8-panel cascade of 8×8 WS2812B RGB LED matrices (512 LEDs total).

Engineered as an automotive rooftop "top light" sign for vehicles, fleets, and public announcements, it displays dynamic messages (e.g. `WELCOME ABOARD`), rotates announcements, runs **140 distinct built-in animations**, features **50 curated color palettes**, provides **full WLED-style matrix geometry configuration**, includes an **interactive Custom Effect Builder**, a **Dynamic Multi-Stage Boot Sequence Studio**, a **dedicated Tactical Strobe Lights Studio**, and an integrated **ESP Web Flasher** for direct in-browser firmware updates.

---

> [!IMPORTANT]
> **CRITICAL REGULATORY & SAFETY DISCLAIMER:**
> This system is designed exclusively for **entertainment, vehicle-identification, and visual-FX display purposes**.
> The turn-indicator, hazard, and strobe light patterns on the LED matrix are **novelty/informational only** — they are **NOT** a certified replacement for the vehicle's legally mandated DOT/ECE indicator and hazard lights.
> This firmware is strictly scoped to display logic. **DO NOT** wire this system directly into or tamper with the vehicle's certified turn-signal wiring, CAN-bus safety lines, or exterior lighting relays.

---

## 1. Hardware Architecture & GPIO Pinout

### Official Hardware GPIO Pin Mapping (Config.h)

| Function | GPIO | Electrical Spec | Operational Behavior |
| :--- | :---: | :--- | :--- |
| **WS2812B LED Data Out** | **2** | Digital Out (FastLED RMT) | Feeds DIN of Panel 0; cascades in series through Panel 7 (512 LEDs total). |
| **Mode Button (PBS-01)** | **6** | `INPUT_PULLUP`, Active-LOW | Mechanical momentary push button with 25ms debounce and multi-click/hold gesture decoding. |
| **Right Turn Indicator** | **3** | `INPUT_PULLUP` / Active-HIGH | Level-sensitive switch input (animates chevrons while active). Configurable Active-HIGH for Op-Amp or Active-LOW. |
| **Left Turn Indicator** | **4** | `INPUT_PULLUP` / Active-HIGH | Level-sensitive switch input (animates chevrons while active). Configurable Active-HIGH for Op-Amp or Active-LOW. |
| **Hazard Switch** | **5** | `INPUT_PULLUP` / Active-HIGH | Dedicated level-sensitive hazard switch (triggers full center-out hazard pattern). Can be toggled or disabled in software. |

> [!NOTE]
> **Fixed Hardware Pinouts:** GPIO pin assignments are strictly configured at the board level in `include/Config.h`. To maintain electrical integrity, pins are fixed and protected from accidental reconfiguration.

---

## 2. Power & Automotive Enclosure Design

1. **Automotive 12V &rarr; 5V Buck Converter**: Step vehicle 12V auxiliary power down to **5.0V DC &plusmn;0.1V** rated for at least **3.5A continuous**.
2. **Circuit Protection**:
   - Install an **inline 3A or 5A fast-blow fuse** on the 12V positive feed.
   - Include a reverse-polarity protection Schottky diode or P-channel MOSFET.
   - Place a **1000µF 6.3V+ capacitor** across 5V and GND near the first LED matrix.
3. **Hardware Current Limiting (WLED Style)**:
   - Configurable from 500mA to 5000mA (default 2500mA @ 5V = 12.5W max).
   - FastLED dynamically caps current without brownouts.

---

## 3. Physical Controls & Mechanical Gestures (PBS-01 on GPIO 6)

The multi-function mode button decodes multi-tap and hold gestures:

| Gesture | Action | Operational Behavior |
| :--- | :--- | :--- |
| **Single Click** | **Change Animation (Next)** | Immediately advances to the **NEXT animation** (0 &rarr; 139) and resets the cycle timer so the new mode plays for its full duration. |
| **Double Click / Tap** | **Favorite FX Toggle** | Instantly switches to the configured Favorite Animation mode. |
| **Triple Click** | **Cycle Brightness** | Steps through presets: `15` &rarr; `30` &rarr; `60` &rarr; `100` &rarr; `150` &rarr; `200`. |
| **Quadruple Click (4-Tap)** | **Animation Lock** | Locks the currently playing animation in place, pausing the auto-cycle timer. Displays a momentary 3-second blink alert. 4-tap again to unlock. |
| **Long Press (&ge;550ms)** | **Cycle Palettes** | Steps through the **50 curated color themes** (0 to 49). |

---

## 4. Multi-Stage Boot Engine & Panel-Grid Snapped Text Architecture

### Dynamic Boot Sequence Engine
On power-up or reboot, the display plays a fully configurable **5- to 12-stage startup sequence** executed by `BootSequenceManager`, non-blocking and persisted to NVS:
- Mix **Text Messages** (with custom transitions, palettes, hold durations, and panel snapping) and **Built-in Animations** (with custom durations and palette overrides).
- **Automated Color Variation Rule**: If the same animation ID appears across multiple stages (e.g., strobe bursts or transitions), the engine automatically assigns distinct color palettes to ensure visual freshness.

### Panel Grid Snap
Because the 64×8 display physically consists of eight 8×8 modules with a visible mechanical seam, resting text columns could straddle seams:
- **Strategy A (1 Char/Panel)**: Centers one 5×7 character inside each 8×8 panel (up to 8 characters). Seams are cleanly placed in the inter-character gaps.
- **Strategy B (2 Chars/Panel)**: Renders two compact 3×5 characters inside each 8×8 panel (up to 16 characters), each glyph cleanly contained within its 4-pixel column slice.
- **Off**: Standard continuous coordinate rendering.

### Text Trails & Hold Duration
- **Trail Decay (0–120, default 0 - Clean & Crisp)**: Prevents ghosting across transitions. At 0, the display performs crisp frame clears (zero trail blur); at 1–120, FastLED renders smooth decay trails. Ghost buffers are explicitly flushed on all phase transitions.
- **Configurable Hold Duration (0–10000ms, default 1500ms)**: Full control over resting text readability.

---

## 5. 140 Built-In Animations Library (Authentic Integer Math)

All 140 animations feature genuinely unique, dedicated algorithmic waveforms and structural motion:
1. **0–9: Automotive, Text & Marquees**: Ribbon Marquee, 3D Wrap Marquee, Neon Pulse Marquee, Wave Marquee, Split-Icon Marquee, Glitch Marquee, Amber Caution Cascade, Dual Horizon Pulse, Cyber Tunnel Zoom, Full-Spectrum Radar.
2. **10–19: Classic Retro & Arcade**: Matrix Rain, Pac-Man Ghost Chase, Space Invaders, 8-Bit Pong, Breakout Bricks, Tetris Blocks, Snake Crawl, Mario Jump, Diamond Strobe Pulse, Glitch Byte.
3. **20–29: Natural Fluids & Elements**: 64-Col Fireplace, Water Ripples, Lava Drift, Forest Canopy, Aurora Borealis, Campfire Sparks, Raindrops, Fog Drift, Ocean Surf, Solar Flare Mega Blast.
4. **30–39: Plasmas, Noise & Math**: 2D Multi-Sine Plasma, Perlin 2D Noise, Simplex Flow, Hypnotic Swirl, Lissajous Curve, Mandelbrot Scan, Chebyshev Waves, Hexagonal Lattice, Sine Interference Wave, Quantum Field Topology.
5. **40–49: Audio & Equalizers**: Audio VU Meter, Dual Peak EQ, Center-Out EQ, Radial Pulse VU, Frequency Waterfall, Oscilloscope Wave, Multi-Band Sparkle, Beat Strobe Bar, Stereo Mirror Meter, Bass Thump Bar.
6. **50–59: Medical & Biometrics**: Cardiac EKG Trace, Neural Synapse, DNA Double Helix, Bionic Pulse Resonance, Pulse Oximeter, WLED Pacifica Ocean, Heartbeat Glow, Cell Division, Biosensor Sweep, Vital Monitor.
7. **60–69: Automotive & Strobes**: Police Wig-Wags, Amber Caution Sweep, Quad-Pop Strobe, Pursuit Interceptor, Fire Dept Strobe, Hazard Breathe, Runway Chase, Pace Car Bar, Aviation Beacon, Highway Arrow.
8. **70–79: Light Chases & Cylons**: Knight Rider Cylon, Theater Chase, Radar Sweep, Laser Beam, Hyper Laser Sweep, Dual Helix Chase, Segment Bounce, Accel-Decel Cylon, Bouncing Dot, Warp Speed Stars.
9. **80–89: Celestial & Cosmic**: Night Sky Twinkle, Shooting Stars, Supernova Burst, Spiral Galaxy, Black Hole Event, Solar Prominence, Orbiting Planets, Nebula Cloud, Amber Highway Chevrons, Hyperdrive Jump.
10. **90–99: Kaleidoscopic Art**: Rainbow 2D Wave, Radial Kaleidoscope, Checkerboard Wave, Diamond Expansion, Concentric Rings, Tunnel Zoom, Hexagonal Mosaic, Shifting Chevron, Mosaic Morph, Vortex Diamond Warp.
11. **100–109: Hyper-Speed & Quantum Sci-Fi FX**: Quantum Warp Tunnel, Hyper-Drive Star Surge, Neon Matrix Cascade, Prism Laser Interceptor, Solar Coronal Blast, Sonic Shockwave Rings, Cyber Highway Pursuit, Galactic Supercluster, Strobe Aurora Horizon, Omega Event Singularity.
12. **110–114: WLED Signature 2D FX**: WLED 2D Pulser, WLED Frizzles Sparks, WLED 2D Sinusoidal Dots, WLED Popcorn Bursts, WLED Flow & Dynamic Smooth.
13. **115–139: Chasers, Panel-Fillers & Strobes (New)**:
    - **115: Panel Cascade Chaser**: Sequential 8x8 panel lighting with trailing decay.
    - **116: 8x8 Panel Block Fill (In/Out)**: Progressive 64-pixel fill per panel block.
    - **117: Split Panel Wig-Wag Strobe**: Left 4 panels vs Right 4 panels rapid alternating tactical strobe.
    - **118: Dual-End Chaser Collision**: Beams shooting from panel 0 & 7 colliding at center with flash burst.
    - **119: 8x8 Matrix Curtain Drop**: Vertical curtain drop cascading across chained panels.
    - **120: Hyper Xenon Quad Burst**: Blinding 4-burst xenon flash followed by color palette wash.
    - **121: Zig-Zag Panel Serpent**: 2-pixel snake navigating serpentine across all 8 panels.
    - **122: Center-Out Panel Bloom**: Symmetrical expansion from panels 3&4 out to panels 0&7.
    - **123: Chevrons Directional Rush**: Directional `>` chevrons racing at high speed with trailing tail.
    - **124: Emergency Amber Quad-Strobe**: Multi-pulse high-intensity amber caution flash.
    - **125: 8x8 Spiral Matrix Fill**: Concentric spiral matrix fill inward inside each 8x8 panel.
    - **126: Multi-Tier Cylon Scanner**: Dual bouncing scanner eyes across top and bottom tiers.
    - **127: Panel Ripple Wave**: Amplitude-modulated sine wave traveling across the 8-panel bar.
    - **128: Diamond Burst Strobe**: Diamond geometric expansion centered in each 8x8 panel.
    - **129: Alternating Odd/Even Panel Strobe**: Panels 0,2,4,6 strobe alternating with panels 1,3,5,7.
    - **130: 8x8 Panel Radar Sweep**: 360-degree rotating radar scanline per panel.
    - **131: Photon Particle Rush**: High-speed particle beams with stochastic light sparks.
    - **132: Hazard Wig-Wag Ramp**: Alternating amber hazard strobe with accelerating ramp rate.
    - **133: 8x8 Box Perimeter Chase**: Synchronized perimeter race around 8x8 panel borders.
    - **134: Strobe Waterfall Cascade**: Falling light columns with bottom splash bounce.
    - **135: Quantum Ping-Pong Ricochet**: Elastic bouncing particles reflecting off panel edges.
    - **136: Panel Shutter Blind Wipe**: Venetian blinds shutter wipe opening and closing.
    - **137: Interceptor Pursuit Strobe**: Police pursuit red/blue outer and amber/white inner pulse.
    - **138: 8x8 Matrix Raindrop Dissolve**: Digital rain cells dissolving into glowing color pools.
    - **139: Hyper Warp Speed Strobe**: Horizon star stretch accelerating into hyperspace flash.

---

## 10. 50 FPS Panoramic Text Display Engine

OSPREY-64x8 includes a standalone, deterministic 50 FPS non-blocking text rendering state machine (`include/TextEngine.h` and `include/TextEngineImpl.h`). All math is 100% integer-based (`uint8_t`, `int16_t`, `scale8`, `qadd8`, `qsub8`, `triwave8`, `sin8`) with zero dynamic memory allocation (`malloc`/`new`) per frame.

### 10.1 Font Engine
- **High-Visibility 5×7 ASCII Font**: Full alphanumeric coverage (A–Z, a–z, 0–9, punctuation, and symbols). Rendered with 1-column proportional inter-character spacing (6 columns total per glyph).
- **Compact 3×5 Font**: Alphanumeric digits and uppercase glyphs for split-screen secondary text and telemetry counters.
- **Ultra-Compact 2×5 Numeric Font**: 2-column digits (0–9) designed for badges, micro status indicators, and timer badges.

### 10.2 Animation & Steady-State Matrix

#### Entry Animations (Phase 1)
| Type | Style Key | Scope | Visual FX Description |
| :--- | :--- | :--- | :--- |
| **Any-Length** | `SLIDE_IN` | Any length | Text translates smoothly from off-screen into view along the movement axis. |
| **Any-Length** | `CURTAIN_OPEN`| Any length | Text expands symmetrically from center outwards (columns 31–32 &rarr; 0/63). |
| **Any-Length** | `WIPE_IN` | Any length | High-tech scanline wipe with pseudo-random byte glitch noise that resolves to clean glyphs. |
| **Any-Length** | `SPARKLE` | Any length | Text pixels materialize progressively from a sparkling starfield bed. |
| **Short-Only** | `TYPEWRITER` | &le; 64 cols | Terminal character-by-character reveal with blinking cursor block. |
| **Short-Only** | `ZOOM_IN` | &le; 64 cols | Expansion burst outward from center point to full 5×7 glyph size. |
| **Short-Only** | `FLIP_IN` | &le; 64 cols | Mechanical split-flap flip-in emulation (top/bottom halves rotate into place). |
| **Short-Only** | `DIGITAL_ASSEMBLE` | &le; 64 cols | Scattered floating pixel fragments converge inward to lock into glyph slots. |
| **Short-Only** | `MATRIX_RAIN` | &le; 64 cols | Digital code streams cascade down 8 rows, leaving locked text pixels in their wake. |

#### Steady-State Styles (Phase 2)
| Style Key | Scope | Description |
| :--- | :--- | :--- |
| `CONTINUOUS_RIBBON` | Any length | Sub-pixel smooth horizontal ribbon scroll across all 64 columns with seamless wrap. |
| `CELL_FLIP_SCROLL` | Any length | Airport-departure split-flap board: stepping across fixed-width 6px cells every 220ms with mechanical top/bottom flip cards. |
| `BOUNCING_MARQUEE` | Short-only (&le; 64 cols) | Classic DVD bounce: text bounces off left (col 0) and right boundaries using integer triangle wave (`triwave8`). |
| `SPLIT_SCREEN` | Short-only (&le; 64 cols) | Static 8×8 passenger/vehicle icon docked on columns 0–7; active text marquee runs across columns 8–63. |
| `STATIC_HOLD` | Short-only (&le; 64 cols) | Static centered display held rock-steady for `hold_ms` duration before triggering exit animation. |

#### Exit Animations (Phase 3)
| Type | Style Key | Scope | Visual FX Description |
| :--- | :--- | :--- | :--- |
| **Any-Length** | `SLIDE_OUT` | Any length | Text translates smoothly off-screen opposite the entry vector. |
| **Any-Length** | `CURTAIN_CLOSE`| Any length | Screen collapses inward symmetrically toward center columns (0/63 &rarr; 31–32). |
| **Any-Length** | `WIPE_OFF` | Any length | Glitch-pixel tear and scanline wipe clearing across the matrix. |
| **Any-Length** | `FADE_SHRINK` | Any length | Luminance fades progressively to black while scale contracts. |
| **Short-Only** | `ZOOM_OUT` | &le; 64 cols | Glyph collapses rapidly inward to center point before vanishing. |
| **Short-Only** | `FLIP_OUT` | &le; 64 cols | Split-flap cards flip rapidly downward into blank positions. |
| **Short-Only** | `DISSOLVE_UP` | &le; 64 cols | Text erodes into rising heat embers drifting upward off row 0. |
| **Short-Only** | `MELT_DOWN` | &le; 64 cols | Pixels drip downward into puddles off row 7. |
| **Short-Only** | `EXPLODE` | &le; 64 cols | High-velocity pixel scatter and particle dispersion outward. |

### 10.3 Message-Length-Aware Selection & Auto-Substitution

When a message is received:
1. The engine computes `textPixelWidth = strlen(msg) * 6`.
2. If `textPixelWidth <= 64`, all short-only and any-length styles are valid.
3. If `textPixelWidth > 64`, short-only styles are automatically substituted with their long-text equivalents:
   - `FLIP_IN` &rarr; `CELL_FLIP_SCROLL`
   - `TYPEWRITER` / `ZOOM_IN` / `DIGITAL_ASSEMBLE` / `MATRIX_RAIN` &rarr; `SLIDE_IN`
   - `BOUNCING_MARQUEE` / `SPLIT_SCREEN` / `STATIC_HOLD` &rarr; `CONTINUOUS_RIBBON`
   - `ZOOM_OUT` / `FLIP_OUT` / `DISSOLVE_UP` / `MELT_DOWN` / `EXPLODE` &rarr; `WIPE_OFF`
4. When auto-substitution occurs, the firmware broadcasts a `TEXT_AUTO_SUB` JSON packet to Web Serial so the Studio UI updates controls dynamically:
   ```json
   {
     "type": "TEXT_AUTO_SUB",
     "orig_entry": "TYPEWRITER",
     "new_entry": "SLIDE_IN",
     "orig_exit": "EXPLODE",
     "new_exit": "WIPE_OFF",
     "orig_steady": "STATIC_HOLD",
     "new_steady": "CONTINUOUS_RIBBON",
     "msg": "WELCOME ABOARD VIP PASSENGER JOHN DOE"
   }
   ```

### 10.4 Web Serial JSON Protocol for Text Engine

Queue custom text messages with custom animations and timing:
```json
{
  "cmd": "TEXT",
  "msg": "WELCOME ABOARD",
  "entry": "FLIP_IN",
  "steady": "STATIC_HOLD",
  "exit": "DISSOLVE_UP",
  "hold_ms": 3500,
  "direction": "LEFT",
  "color": "#FF8C00"
}
```

The message queue buffer holds up to 8 messages and plays them sequentially with seamless phase transitions.

---

## 11. System Default Values Reference Table

| Parameter | Configuration Key / Command | Factory Default | Valid Range / Options |
| :--- | :--- | :--- | :--- |
| **LED Current Limit** | `max_ma` | `2500` mA | 500 – 5000 mA |
| **Color Order** | `color_order` | `0` (GRB) | 0:GRB, 1:RGB, 2:BGR, 3:BRG, 4:RBG, 5:GBR |
| **Serpentine Scanning** | `serpentine` | `1` (Enabled) | 0: Progressive, 1: Serpentine |
| **Matrix Rotation** | `rotation` | `0` (0° Normal) | 0: 0°, 1: 90°, 2: 180°, 3: 270° |
| **Start Corner** | `start_corner` | `0` (Top-Left) | 0: TL, 1: TR, 2: BL, 3: BR |
| **Target Frame Rate** | `fps` | `50` FPS | 30, 40, 50, 60 FPS |
| **Gamma Correction** | `gamma` | `1` (Enabled) | 0: Disabled, 1: Enabled |
| **Master Brightness** | `brightness` | `60` (out of 255) | 5 – 255 |
| **Animation Mode** | `anim_mode` | `0` (Traveler Marquee) | 0 – 99 |
| **Color Theme Palette** | `palette` | `0` (Classic Gold) | 0 – 49 |
| **Auto Cycle Enable** | `auto_cycle` | `1` (Active) | 0: Locked, 1: Enabled |
| **Auto Cycle Duration**| `cycle_time` | `10` seconds | 3 – 120 seconds |
| **Vehicle Name** | `veh_name` | `"KHALNAYAK"` | UTF-8 String (&le;32 chars) |
| **Vehicle Tagline** | `veh_tag` | `"DONT TRIGGER MY EGO"` | UTF-8 String (&le;32 chars) |
| **Dedicated Greeting** | `msg` | `"WELCOME ABOARD"` | UTF-8 String (&le;64 chars) |
| **Default Entry FX** | `entry` | `SLIDE_IN` | 9 entry animation styles |
| **Default Steady FX** | `steady` | `STATIC_HOLD` (or `CONTINUOUS_RIBBON` if >64 cols) | 5 steady-state display styles |
| **Default Exit FX** | `exit` | `SLIDE_OUT` | 9 exit animation styles |
| **Default Hold Time** | `hold_ms` | `3000` ms | 500 – 15000 ms |
| **Default Text Color** | `color` | `#FF8C00` (Amber Gold) | Hex RGB `#RRGGBB` |

---

## 12. PlatformIO Build & Flash Instructions

```powershell
# Build for ESP32-C3
pio run -e esp32-c3-devkitm-1

# Flash firmware over USB
pio run -e esp32-c3-devkitm-1 -t upload

# Open Serial Monitor at 115200 baud
pio device monitor -b 115200
```

---

## 13. Web Companion Controller Studio (`index.html`)

The companion web studio is a **zero-install, single-file modern web application** (`index.html`) built with vanilla HTML5, CSS3 glassmorphism, and JavaScript. It communicates bi-directionally with the ESP32 over the browser's native **Web Serial API** at **115200 baud** (supported in Google Chrome, Microsoft Edge, and Opera on desktop).

### Core Features List

- **Autonomous 64×8 Display Preview (Client-Side Simulator)**:
  - Real-time 50 FPS canvas simulation mimicking all 140 animations, 50 palettes, and text marquees client-side.
  - Generates **zero serial bandwidth overhead** (telemetry transmits settings only, never raw LED streams).
  - Displays real-time estimated wattage (`W = (Volts × mA) / 1000`), active FPS, and lock status.

- **Dual Persistence Architecture & Studio Profile Backups**:
  - **Header `[💾 Save to NVS]`**: Unified master save committing all geometry, power, timing, text, and animation settings to flash memory in a single click.
  - **Individual Tab Save Buttons**: Dedicated save buttons on every tab (`Save Timer`, `Save Broadcast`, `Save Active Anim`, `Save Active Palette`, `Save Matrix Config`, `Save Pin Config`, `Save Custom FX`, `Save Strobe`) for focused changes.
  - **Header `[↺ Reset to Defaults]`**: Restores all matrix dimensions, hardware GPIO pin mappings, power limits, and animations to clean factory defaults with a single confirmation modal.
  - **Comprehensive JSON Backup & Restore**:
    - `[📥 Backup JSON]`: Exports full studio profile (Hardware GPIO Pin assignments, 2D Matrix Geometry, Power Caps, Live Broadcast Text, 140 Animations, 50 Palettes, Custom FX waveforms, Tactical Strobes, and Boot Sequence stages) to a structured `.json` backup file.
    - `[📤 Restore JSON]`: Imports and restores complete studio profile from any exported backup file, updates all UI controls, caches in local browser storage, and automatically commits directly to ESP32 NVS Flash memory over Web Serial if connected.

- **Studio Tab Breakdown**:
  1. **⚡ Dashboard**: Global animation auto-cycle toggle, duration slider (2s–60s) with preset pills (3s, 5s, 8s, 10s, 15s, 20s, 30s), gesture shortcuts (`1-Tap: Next Anim`, `2-Tap: Fav FX`, `4-Tap: Lock`), turn signal/hazard test stalk with Op-Amp polarity toggles.
  2. **💬 Live Broadcast**: Real-time marquee messaging with standard 5x7, bold 6x7, or compact 3x5 fonts; 6 motion styles (`Continuous Ribbon`, `3D Perspective Wrap`, `Breathing Neon Pulse`, `Sine Wave Undulation`, `Split-Screen Icon`, `Glitch Scan`); speed and trail decay controls.
  3. **🚀 Boot Sequence**: Dynamic 5-to-12 stage startup sequence designer with text hold times, panel snapping, and animation overrides.
  4. **✨ 140 Animations Library**: Searchable and filterable visual cards covering all 13 categories (including the 25 new chaser, strobe, and matrix-fill algorithms) with one-click activation.
  5. **🎨 50 Color Palettes**: Visual swatches displaying gradients and color stops for instant theme preview and selection.
  6. **⚙️ WLED Matrix & Pin Settings**: Custom width/height, 32×16 dual-tier grid, panel count/size, voltage, current limit cap, serpentine scanning, 4-way rotation, one-click `⚡ Apply Roof-Mount Preset`, and **Hardware GPIO Pin Configuration** (`DIN` GPIO 2, `Button` GPIO 6, `Right` GPIO 3, `Left` GPIO 4, `Hazard` GPIO 5).
  7. **🧪 Custom FX Builder**: Interactive generator blending geometric shapes (Rings, Stars, Diamonds, Bars, Crosses) with mathematical waveforms (Sine, Sawtooth, Bounce, Chaos Noise).
  8. **🚨 Strobe Lights Studio**: Emergency strobe suite featuring Police Dual-Color Wig-Wags, Amber Hazard Strobes, Xenon Quad-Pop Bursts, Runway Sweeps, and customizable Hz rates.

---

## 14. ESP Web Flasher Integration (In-Browser Firmware Updates)

The web controller includes built-in support for **ESP Web Tools**, enabling direct firmware flashing straight from the web browser without installing Python, PlatformIO, or esptool:

1. **One-Click Flash Initiation**:
   - Click the **`[⚡ Flash Firmware]`** button located at the top right header.
   - The web app **automatically disconnects the active Web Serial connection** to release the COM port, preventing serial lock contention.
2. **Interactive Flasher Modal**:
   - A step-by-step popup appears guiding the user to connect the ESP32-C3 via USB.
   - Click **`⚡ Connect & Flash Firmware`** inside the modal.
   - The browser's native serial port picker prompts for the ESP32-C3 port.
3. **Flashing Procedure**:
   - ESP Web Tools reads `manifest.json` and writes the 4 required binary files at their correct hardware offsets:
     - `bootloader.bin` &rarr; `0x0000` (Offset 0)
     - `partitions.bin` &rarr; `0x8000` (Offset 32768)
     - `boot_app0.bin` &rarr; `0xe000` (Offset 57344)
     - `firmware.bin` &rarr; `0x10000` (Offset 65536)
   - Progress bar displays real-time erase and flashing status.

---

## 15. Automated Build & GitHub Hosting Pipeline

The project includes an automated Python post-build exporter (`export_firmware.py`) wired directly into `platformio.ini`:

```ini
[env:esp32-c3-devkitm-1]
extra_scripts = post:export_firmware.py
```

### Automated Actions on Build:
Whenever you compile with PlatformIO (`pio run`), the script automatically:
1. Copies `firmware.bin`, `bootloader.bin`, and `partitions.bin` from `.pio/build/esp32-c3-devkitm-1/`.
2. Locates `boot_app0.bin` from the ESP32 framework partitions directory.
3. Generates a standard `manifest.json` configured for the ESP32-C3 chip family.
4. Places all `.bin` files and `manifest.json` directly into:
   - **Repository Root**: For hosting GitHub Pages directly from root (`/`).
   - **`/dist_github` Directory**: A clean, standalone deployment folder containing only the required web app and binary assets (`index.html`, `manifest.json`, and the 4 `.bin` files).

### Deploying to GitHub Pages:
1. Commit the repository to GitHub:
   ```bash
   git add .
   git commit -m "Add 140 FX, ESP Web Flasher, and manifest"
   git push origin main
   ```
2. In your GitHub repository:
   - Navigate to **Settings &rarr; Pages**.
   - Under **Build and deployment &rarr; Source**, choose **Deploy from a branch**.
   - Select your branch (`main`) and choose `/ (root)` or `/dist_github`.
   - Click **Save**.
3. Open your GitHub Pages URL (e.g. `https://your-username.github.io/your-repo/`) in Google Chrome or Microsoft Edge.
4. Connect to your display or flash updated firmware directly from the cloud!


