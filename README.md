# ifmLabz2026 - Arduino Nano ESP32 Interactive Game System

## Project Overview

This gaming console is an interactive embedded system project built during IFM Labz 2026 laboratories, using the Arduino Nano ESP32 platform. The system features a menu-driven interface with multiple games and utilities displayed on an ST7735 LCD screen, controlled via push buttons and a joystick.

## Current Status: Fully Functional

The project is currently in a **stable, working state** with all core features implemented and tested.

---

## Hardware Components

### Display & Input
- **LCD Display**: Adafruit ST7735 (128x96 pixels)
- **Push Buttons**: 4 momentary buttons (SW1-SW4) with debouncing
- **Joystick**: Analog XY joystick with push button capability

### Peripherals
- **I2C GPIO Expander**: PCA9557 for LED control (8 outputs)
- **SD Card Module**: For image storage and retrieval
- **RGB LED**: 3-color indicator (active-low configuration)

### Communication
- **SPI Bus**: Shared interface for LCD and SD card
- **Serial/USB-CDC**: Debug output at 115200 baud

---

## Software Architecture

### Menu System (6 Screens)

1. **MENU_INIT** - Initialization splash screen
2. **MENU_DEFAULT** - Home screen with elapsed time display
3. **MENU_IMAG** - SD card image viewer (BMP format)
4. **MENU_JOY_POS** - Joystick crosshair position display
5. **MENU_DICE** - Electronic dice game (1-6)
6. **MENU_REAC** - Two-player reaction time game

### Core Features

#### Menu Navigation
- **SW1**: Previous menu (wraps around)
- **SW4**: Next menu (wraps around)
- Serial debug output for menu changes

#### Image Viewer (MENU_IMAG)
- Displays BMP images from SD card
- **SW2**: Previous image
- **SW3**: Next image
- Image name displayed at top of screen
- Wraps around when reaching start/end of list

#### Joystick Display (MENU_JOY_POS)
- Real-time crosshair visualization
- Automatic center calibration (16-sample average at startup)
- Two-segment mapping for smooth response
- Cyan dot follows joystick position
- Gray crosshair lines for reference
- Dead zone: >2 pixel movement threshold before redraw

#### Dice Game (MENU_DICE)
- Generates random numbers (1-6) on button press
- **SW3**: Roll dice
- Displays number on LCD
- Controls 8 LEDs via PCA9557:
  - Dice value 1 → LED 1
  - Dice value 2 → LED 2
  - Dice value 3 → LED 3
  - Dice value 4 → LED 4
  - Dice value 5 → LED 5
  - Dice value 6 → LED 6

#### Reaction Game (MENU_REAC)
- **Two-player competitive game**
- **SW3** to start round
- **SW2** (Player 1) and **SW3** (Player 2) to react

**Game Flow:**
1. Press SW3 - BLUE screen (waiting for signal)
2. Random delay (500-2500ms)
3. Screen flashes GREEN
4. Players react as fast as possible
5. Results displayed with:
   - Individual reaction times in milliseconds
   - Winner announcement
   - False start detection (press before GREEN)

**False Start Handling:**
- Player marked with time = 9999ms
- Opponent wins automatically
- Clear indication: "FALSE START" on results screen

#### Time Display (MENU_DEFAULT)
- Elapsed time since startup displayed at bottom
- Format: HH:MM:SS
- Serial output every second

---

## Control Layout

### Button Mapping
| Button | Function |
|--------|----------|
| **SW1** | Previous Menu |
| **SW2** | Image Prev / Player 1 React |
| **SW3** | Image Next / Dice Roll / Player 2 React / Start Game |
| **SW4** | Next Menu |

### Joystick
- **VRx**: Horizontal position (0-4095)
- **VRy**: Vertical position (0-4095)
- **Button**: Currently unused

---

## Technical Implementation

### Task Scheduler
- **Cooperative multitasking** with 3 periodic tasks:
  - `Task1_10ms()` - 100ms interval (menu, dice, reaction game)
  - `Task2_5ms()` - 50ms interval (joystick continuous read)
  - `Task3_1s()` - 1s interval (elapsed time update)

### Debouncing
- ISR-based button debouncing
- 500ms debounce period per button
- Atomic flag handling with interrupt disable

### Display Management
- `resultsDisplayed` flag prevents flicker on reaction game results
- Screen redraws only when data changes
- Optimized rendering in Task1_10ms()

### Memory Management
- 10 BMP files max on SD card
- 15-char max filename length
- Joystick calibration stored in EEPROM-equivalent globals

### Requirements
-Arduino IDE 1.8.19+
- Arduino Nano ESP32 board support
- Required libraries:
  - Adafruit ST7735 and ST789 Library
  - Adafruit BusIO
  - Adafruit GFX Library
  - Adafruit seesaw Library
  - SD (built-in)
