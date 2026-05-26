# Overview
![](/docs/img/printer.jpg)  
![](/docs/img/motherboard.jpg)  

Custom Marlin 2.0 firmware for the **Geeetech A10M** with a **GT2560 V4** board (ATmega2560) with non-replaceable A4988 stepper motor driver chips.  
This is a personalized build - read the section below carefully before flashing to your own printer.

# Things to change for your printer

This firmware is **not a drop-in replacement** for a stock Geeetech A10M. It targets a specific hardware setup. Review each item below and revert anything that doesn't match your printer.

## 1. Stepper motor precision (steps per mm)

This build uses **0.9° stepper motors** (Usongshine 17HS4401S-0.9) which have twice the step resolution of the stock 1.8° motors, combined with a **T8R2 lead screw** (2mm/rev) which has x4 of the stock Z steps.

To revert to stock 1.8° motors and standard lead screw, replace in `Configuration.h`:
```cpp
// Replace this:
#define DEFAULT_AXIS_STEPS_PER_UNIT   { 160.61, 160.61, 1600.00, 432.00 }
// With the original stock values:
#define DEFAULT_AXIS_STEPS_PER_UNIT   { 80, 80, 400, 98 }
```

## 2. Aftermarket dual-drive extruders (reversed extrusion direction)

This build uses aftermarket dual-drive extruders (e.g. BMG clones) on both E0 and E1, which run in the opposite direction to the stock extruder.

To revert to stock extruder direction, replace in `Configuration.h`:
```cpp
// Replace this:
#define INVERT_E0_DIR true
#define INVERT_E1_DIR true
// With the original:
#define INVERT_E0_DIR false
#define INVERT_E1_DIR false
```

## 4. BLTouch probe

BLTouch is enabled and configured with this probe-to-nozzle offset:
```cpp
#define NOZZLE_TO_PROBE_OFFSET { -39, 4, -0.49 }
```

The Z offset (`-0.49`) is calibrated for this specific printer — you **must** calibrate yours using the paper method or `PROBE_CALIBRATE` marlin feature. The X/Y values (`-39, 4`) are physical measurements of where the BLTouch is mounted relative to the nozzle — measure yours and update accordingly. But for the default printhead cage of A10M those should work well. 

To disable BLTouch entirely, comment out in `Configuration.h`:
```cpp
//#define BLTOUCH
```

## 5. Bed dimensions

The actual printable area was remeasured and corrected from the Geeetech defaults:
```cpp
#define X_MIN_POS -4
#define Y_MIN_POS -8
#define Z_MIN_POS 0

#define X_BED_SIZE 221
#define Y_BED_SIZE 223

#define X_MAX_POS X_BED_SIZE 
#define Y_MAX_POS Y_BED_SIZE
#define Z_MAX_POS 225
```

The original firmware had `225 x 230` (no matter that the bed size is `235 x 235` - the real usable area is slightly lower).  
I found that at some point the nozzle gets out of the printable area and at the max X, for example, there's no way to get to the edge of the bed, so I strongly suggest to remeasure using caliper yours and adjust to match your actual bed if different.

## 6. Probing margin

Decreased for more accuracy because of more accurate printable area dimensions sat earlier, works pretty well but you can change it back:
```cpp
#define PROBING_MARGIN 5
```

To revert to the Marlin default:
```cpp
#define PROBING_MARGIN 30
```

## 7. LCD encoder sensitivity

The encoder wheel on this unit required 4 pulses per detent instead of the default 2:
```cpp
#define ENCODER_PULSES_PER_STEP 4
```

If your scroll wheel skips steps or moves double, try:
```cpp
#define ENCODER_PULSES_PER_STEP 2
```

## 8. LCD controller type

The Geeetech A10M with a GT2560 V4 board targets the **YHCB2004** display (20x4 LCD shipped only with the A10M GT2560 V4 variant). The upstream config had `REPRAP_DISCOUNT_SMART_CONTROLLER` which causes a blank screen on this board.

```cpp
//#define REPRAP_DISCOUNT_SMART_CONTROLLER
#define YHCB2004
```

If you have any issues with the LCD - revert it for a standard RepRap Smart Controller:
```cpp
#define REPRAP_DISCOUNT_SMART_CONTROLLER
//#define YHCB2004
```

## 9. Machine name

Feel free to change `"Opportunity"` to whatever you want your printer to display on the LCD:
```cpp
#define CUSTOM_MACHINE_NAME "Opportunity"
```

# How to flash to your printer

## Prerequisites

1. [Visual Studio Code](https://code.visualstudio.com/)
2. [PlatformIO IDE](https://marketplace.visualstudio.com/items?itemName=platformio.platformio-ide) extension for VS Code
3. USB-A to USB-B cable (the square Arduino-style plug)
4. Git Bash, PowerShell, or any terminal

## Backing up the original firmware

Before flashing for the first time, back up the factory firmware so you can restore it if something goes wrong.

```bash
avrdude -p atmega2560 -c wiring -P COM3 -b 115200 -U flash:r:firmware_backup/my_backup.hex:i
```

To restore from backup:
```bash
avrdude -p atmega2560 -c wiring -P COM3 -b 115200 -U flash:w:firmware_backup/my_backup.hex:i
```

The `firmware_backup/` folder in this repo contains the backups taken before this build was first created.

## Option A — Flash using VS Code tasks (recommended)

1. Open this folder in VS Code (`File → Open Folder`)
2. Plug the USB cable into the printer and your PC
3. Press `Ctrl+Shift+P` → type **Run Task** → press Enter
4. Select **PlatformIO: Upload (mega2560)**
5. Watch the terminal — success looks like: `avrdude done. Thank you.`

Other available tasks (`Ctrl+Shift+P → Run Task`):

| Task | What it does |
|------|-------------|
| `PlatformIO: Upload (mega2560)` | Compile + flash firmware |
| `PlatformIO: Upload and Monitor (mega2560)` | Compile, flash, then open serial monitor |
| `PlatformIO: Monitor (mega2560)` | Open serial monitor only (no upload) |

---

## Option B — Flash using the PlatformIO status bar

1. Open this folder in VS Code
2. Look at the **bottom blue status bar** — find the PlatformIO toolbar icons
3. Click the **→ (Upload)** arrow to compile and flash
4. Click the **plug 🔌** icon to open the serial monitor

---

## Option C — Flash using CLI

Run from the project root in Git Bash or PowerShell:

```bash
~/.platformio/penv/Scripts/platformio run -e mega2560 --target upload --upload-port COM3 2>&1
```

Replace `COM3` with your actual port. To find it:
- **Windows**: Device Manager → Ports (COM & LPT)
- **Linux/Mac**: `ls /dev/tty*` before and after plugging in

To compile without uploading (check for errors only):
```bash
~/.platformio/penv/Scripts/platformio run -e mega2560 2>&1
```

To open the serial monitor after flashing:
```bash
~/.platformio/penv/Scripts/platformio device monitor
```

> [!NOTE] Note
> The serial monitor baud rate is `250000` — make sure any external serial tool matches this.


## After flashing — first steps

1. **Reset EEPROM** — LCD: *Configuration → Advanced Settings → Initialize EEPROM*  
   or via serial: send `M502` then `M500`
2. **Calibrate Z offset** — LCD: *Configuration → Probe Z Offset*  
   Adjust until the first layer adheres correctly
3. **Run bed leveling** — send `G29` or LCD: *Motion → Bed Leveling*
4. **Save settings** — send `M500` or LCD: *Configuration → Store Settings*
