> [!NOTE]
> If you discover issues with custom builds or are not using the provided binaries in the releases page, **make sure you inform in the issue of what you modified in the code.** If it's a general firmware issue, see if it happens in the precompiled builds first.

## OpenFIRE Build Manual
 - [Arduino (-cli) Setup](#arduino--cli-setup)
 - [Sketch Configuration](#sketch-configuration)
 - [Define Buttons & Timers](#define-buttons--timers)

### Arduino (-cli) Setup
Compiling from the cli is necessary for build flags to apply to the whole project, rather than just `SamcoEnhanced.ino`. *You will run into issues building from the IDE.*
 1. [Download the Arduino-cli tool for your system](https://github.com/arduino/arduino-cli/releases/latest) and install it to where it's most convenient (or for Linux users, install from your system's package manager).
    - These instructions are tailored to Linux, but Windows users can use `arduino-cli.exe` whenever the tool is referenced.
 3. Install the patched RP2040 core for OpenFIRE:
    ```bash
    $ arduino-cli core install rp2040:rp2040 --additional-urls https://github.com/SeongGino/arduino-pico/releases/download/3.9.2-fix/package_rp2040_fix_index_orig.json
    ```
    - Optional: for *Arduino Nano RP2040 Connect*, also install its WiFiNINA library:
      ```bash
      $ arduino-cli lib install WiFiNINA
      ```
 4. Clone the repository, making sure to also download its submodules:
    ```bash
    $ git clone --recursive https://github.com/TeamOpenFIRE/OpenFIRE-Firmware
    ```
 5. Find the proto name of the board to build for:
    ```bash
    $ arduino-cli board listall rp2040
    
    Board Name                           FQBN
    0xCB Helios                          rp2040:rp2040:0xcb_helios
    Adafruit Feather RP2040              rp2040:rp2040:adafruit_feather
    Adafruit Feather RP2040 CAN          rp2040:rp2040:adafruit_feather_can
    Adafruit Feather RP2040 DVI          rp2040:rp2040:adafruit_feather_dvi
    Adafruit Feather RP2040 Prop-Maker   rp2040:rp2040:adafruit_feather_prop_maker
    Adafruit Feather RP2040 RFM          rp2040:rp2040:adafruit_feather_rfm
    ...
    ```
 6. Build OpenFIRE Firmware (replacing `{BOARD}` with your desired microcontroller's fqbn name:
    ```bash
    $ arduino-cli compile -e --fqbn rp2040:rp2040:{BOARD}:usbstack=tinyusb,opt=Optimize3 /path/to/OpenFIRE-Firmware/SamcoEnhanced --libraries /path/to/repo/libraries --build-property "build.extra_flags=-DUSES_DISPLAY=1 -DPLAYER_NUMBER=1 -DUSES_SOLENOID=1 -DUSES_RUMBLE=1 -DUSES_SWITCHES=1 -DMAMEHOOKER=1 -DUSES_ANALOG=1 -DCUSTOM_NEOPIXEL=1 -DFOURPIN_LED=1 -DDUAL_CORE=1"
    ```
    *for custom builds, feel free to configure the above build flags to your needs to enable/disable certain OpenFIREfw features.*
    
When successful, you will find the exported binary at `/path/to/OpenFIRE-Firmware/SamcoEnhanced/build/rp2040.rp2040.{BOARD}/SamcoEnhanced.ino.uf2`

### Sketch Configuration
Per-board build configurations for various microcontrollers are located in `SamcoPreferences.cpp - SamcoPreferences::LoadPresets()`, and the board report strings to identify the board in the OpenFIRE App can be found in `libraries/OpenFIREPosition/OpenFIREBoard.h`

### Define Buttons & Timers
Tactile extras can be defined/unset by simply (un)commenting the respective defines - though each one of these can be simply disabled at runtime even when the firmware is "fully kitted".

If your gun is going to be hardset to player 1/2/3/4 e.g. for an arcade build, change `#define PLAYER_NUMBER` to 1, 2, 3, or 4 depending on what keys you want the Start/Select buttons to correlate to. Remember that guns can be remapped to any player number arrangement at any time if needed by sending an `XR#` command over Serial - where # is the player number.

These parameters are easily found and can be redefined in the sketch configuration area - just above the `PLAYER_NUMBER` indicator!

```c++
#define MANUFACTURER_NAME "OpenFIRE"
#define DEVICE_NAME "FIRECon"
#define DEVICE_VID 0xF143
#define DEVICE_PID 0x1998
```

You may change these to suit whatever your heart desires - though the only parts *necessary to change for multiplayer* are the `DEVICE_VID` and `DEVICE_PID` pairs. Then, just reflash the board!

Remember that the sketch uses the Arduino GPIO pin numbers; on many boards, including the Raspberry Pi Pico and the Adafruit Itsybitsy RP2040, these are the silkscreen labels on the **underside** of the microcontroller (marked GP00-29). Note that this does not apply to the analog pins (A0-A3), which are macros for GP26-29.
Refer to [this interactive webpage](https://pico.pinout.xyz/) for detailed information on the Pico/W's layout, or your board vendor's documentation

The default button:pins layout used will be reflected by default in the OpenFIRE App, which can be used as reference or can be changed to any custom pins layout to suit your needs - custom settings will take priority over board defaults if enabled & detected.

Once the sketch is configured to your liking, plug the board into a USB port, click Upload (the arrow button next to the checkmark) and your board will reconnect a few times until it's recognized as a combined mouse/keyboard/gamepad device!
