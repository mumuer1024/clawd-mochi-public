# Clawd Mochi — ESP32-C3 Desk Companion

A physical desk companion with animated eyes, web controller, terminal, canvas, monitor, and reminder features.

## Hardware

| Component | Spec |
|-----------|------|
| MCU | ESP32-C3 Super Mini |
| Display | ST7789 1.54" 240×240 TFT (SPI) |
| WiFi | Station mode, static IP 192.168.2.233 |

### Pin Mapping

| Display Pin | GPIO |
|-------------|------|
| SDA (MOSI) | GPIO 10 (hardware SPI) |
| SCL (SCK) | GPIO 8 (hardware SPI) |
| RST | GPIO 2 |
| DC | GPIO 1 |
| CS | GPIO 4 |
| BL (backlight) | GPIO 3 |
| VCC | 3V3 (never 5V!) |
| GND | GND |

## WiFi Configuration

- **Mode**: Station (client)
- **SSID**: `00000`
- **Password**: `83189906mac`
- **Static IP**: `192.168.2.233`
- **Gateway**: `192.168.2.1`
- **Web Server**: `http://192.168.2.233`

## Views (currentView)

| ID | Constant | Description |
|----|----------|-------------|
| 0 | `VIEW_EYES_NORMAL` | Animated pixel eyes (wiggle + blink) |
| 1 | `VIEW_EYES_SQUISH` | Squish eyes animation (not in web UI) |
| 2 | `VIEW_CODE` | Claude Code terminal display |
| 3 | `VIEW_DRAW` | Canvas drawing mode |
| 4 | `VIEW_MONITOR` | System stats from NanoPi |
| 5 | `VIEW_REMINDER` | Timed reminders display |

## Web API Endpoints

| Route | Method | Description |
|-------|--------|-------------|
| `/` | GET | Web controller HTML page |
| `/cmd?k=xxx` | GET | Execute command |
| `/char?c=x` | GET | Add char to terminal |
| `/speed?v=1-3` | GET | Set animation speed |
| `/bg?c=#RRGGBB` | GET | Set background color (deprecated, use /redraw) |
| `/redraw?bg=#RRGGBB` | GET | Set background and redraw current view |
| `/bl?on=true/false` | GET | Toggle backlight |
| `/backlight?on=1/0` | GET | Toggle backlight |
| `/canvas?on=1/0` | GET | Toggle canvas mode |
| `/draw/clear?bg=#RRGGBB` | GET | Clear canvas with background color |
| `/draw/stroke?pen=#RRGGBB&pts=x,y;x,y;...` | GET | Draw stroke on canvas |
| `/state` | GET | Get current state JSON |
| `/reminder` | GET/POST/PUT/DELETE | CRUD for reminders |

### Command Keys (`/cmd?k=xxx`)

**View commands (single char):**
- `w` → Normal eyes
- `s` → Squish eyes
- `d` → Code terminal
- `a` → Logo reveal animation
- `m` → Monitor
- `q` → Exit terminal mode

**Static faces (`face_xxx`):**
- `face_wuyu` → 无语 (speechless -_-)
- `face_wenhao` → ? 问号
- `face_gantanhao` → ! 感叹号
- `face_angry` → 生气 ><
- `face_yes` → ✓ 对号
- `face_X` → ✗ 叉叉
- `face_glass` → 墨镜 sunglasses

**Animated faces (`anim_xxx`):**
- `anim_jiyanjing` → 挤眼睛 (眨眼动画, 3 loops)
- `anim_yun` → 晕晕 (yun1/yun2/yun3, 3 loops)
- `anim_close` → 闭眼睛 (close1/close2/close3, 3 loops)
- `anim_dead` → 死掉了 (dead1/dead2/dead3, 3 loops)
- `anim_dian` → 等等 (dian1→dian2→dian3 stop)
- `anim_smile` → 笑笑 (smile1/smile2, 3 loops)
- `anim_look` → 看你 (look1/look2, 3 loops)
- `anim_hart` → 心跳 (hart1/hart2, 3 loops)
- `anim_zzz` → 睡着了 (zzz1/zzz2/zzz3 stop)
- `anim_ganga` → 尴尬 (ganga1/ganga2/ganga3 stop)

## Face Drawing System

### File Structure

| File | Purpose |
|------|---------|
| `clawd_mochi.ino` | Main firmware (~1700 lines) |
| `faces_code.h` | Face drawing functions (~1100 lines, auto-generated) |

### Drawing Method

Faces are drawn using `tft.fillRect()` calls with 6×6 pixel blocks on a 40×40 grid:
- **Foreground**: `C_BLACK` → black pattern
- **Background**: `animBgColor` → user-settable (default orange `#DA1100`)

Each face function (e.g., `drawFace_smile1()`) fills the screen with `animBgColor` then draws the pattern with black rectangles.

### Static Face Functions

| Function | Description |
|----------|-------------|
| `drawFace_wuyu()` | 无语 -_- |
| `drawFace_wenhao()` | ? 问号 |
| `drawFace_gantanhao()` | ! 感叹号 |
| `drawFace_angry()` | 生气 >< |
| `drawFace_yes()` | ✓ 对号 |
| `drawFace_X()` | ✗ 叉叉 |
| `drawFace_glass()` | 墨镜 sunglasses |

### Animated Face Functions

| Function | Frames | Behavior |
|----------|--------|----------|
| `anim_jiyanjing()` | jiyanjing1/jiyanjing2 | 3 loops, ends on jiyanjing1 |
| `anim_yun()` | yun1/yun2/yun3 | 3 loops, ends on yun1 |
| `anim_close()` | close1/close2/close3 | 3 loops, ends on close1 |
| `anim_dead()` | dead1/dead2/dead3 | 3 loops, ends on dead1 |
| `anim_dian()` | dian1→dian2→dian3 | Single pass, stops at dian3 |
| `anim_smile()` | smile1/smile2 | 3 loops, ends on smile1 |
| `anim_look()` | look1/look2 | 3 loops, ends on look1 |
| `anim_hart()` | hart1/hart2 | 3 loops, ends on hart1 |
| `anim_zzz()` | zzz1→zzz2→zzz3 | Single pass, stops at zzz3 |
| `anim_ganga()` | ganga1→ganga2→ganga3 | Single pass, stops at ganga3 |

## Animation Speed

`animSpeed`: 1=slow, 2=normal, 3=fast

```cpp
int speedMs(int ms) {
  if (animSpeed == 3) return ms / 2;
  if (animSpeed == 1) return ms * 2;
  return ms;
}
```

## Monitor System

- Fetches `stats.json` via HTTPS from `192.168.2.1`
- Auto-refresh every 50 seconds
- Required fields: `load`, `mem`, `temp`, `uptime`, `hour`, `minute`, `day`

## Reminder System

- Max 5 reminders
- 20-char message limit
- Uses NanoPi time (hour/minute/day from stats.json)
- 30-second display duration
- Checks every minute (60000ms)
- Web CRUD via `/reminder` endpoint

## Terminal System

| Setting | Value |
|---------|-------|
| Columns | 15 |
| Rows | 8 |
| Char Width | 12px |
| Char Height | 20px |
| Padding X | 8px |
| Padding Y | 18px |

## Build Instructions

1. Install Arduino IDE 2.x
2. Add ESP32 board support URL: `https://raw.githubusercontent.com/espressif/arduino-esp32/gh-pages/package_esp32_index.json`
3. Install libraries: `Adafruit GFX Library`, `Adafruit ST7735 and ST7789 Library`, `ArduinoJson`
4. Board settings:
   - Board: **ESP32C3 Dev Module**
   - USB CDC On Boot: **Enabled**
   - CPU Frequency: 160 MHz
   - Upload Speed: 921600
5. Open `clawd_mochi/clawd_mochi.ino`
6. Upload

## Key Source Files

| File | Purpose |
|------|---------|
| `clawd_mochi/clawd_mochi.ino` | Main firmware (~1700 lines) |
| `clawd_mochi/faces_code.h` | Face drawing functions (~1100 lines) |

## Coding Conventions

- Single-file architecture (`clawd_mochi.ino`) for beginner accessibility
- `faces_code.h` included for face drawing functions (auto-generated)
- Comments in both English and Chinese
- Animation functions prefix: `animXxx()` for animated, `drawFace_xxx()` for static
- Color constants: `C_ORANGE`, `C_BLACK`, `C_WHITE`, `C_DARKBG`, `C_MUTED`, `C_GREEN`
- `busy` flag prevents concurrent animation conflicts
- `server.handleClient()` called during long animations

## Eye Customization

```cpp
#define EYE_W   30    // eye width in pixels
#define EYE_H   60    // eye height in pixels
#define EYE_GAP 120   // gap between eyes
#define EYE_OX  0     // horizontal offset
#define EYE_OY  40    // vertical offset upward
```

## Colors

| Constant | RGB565 | Hex | Description |
|----------|--------|-----|-------------|
| `C_ORANGE` | `tft.color565(218, 17, 0)` | `#DA1100` | Default background |
| `C_DARKBG` | `tft.color565(10, 12, 16)` | `#0A0C10` | Dark background |
| `C_MUTED` | `tft.color565(90, 88, 86)` | `#5A5856` | Muted text |
| `C_GREEN` | `tft.color565(80, 220, 130)` | `#50DC82` | Green text |
| `C_WHITE` | `ST77XX_WHITE` | `#FFFF` | White |
| `C_BLACK` | `ST77XX_BLACK` | `#0000` | Black (face pattern) |