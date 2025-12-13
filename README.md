# Adafruit nRF52 Bootloader

This repository is a fork of the official Adafruit repository. My keyboards use external LED indicators, so I have made changes that allow them to display the bootloader status. The original code supports blinking only a single LED, whereas my version adds the ability to work with multiple indicators. If you have any questions unrelated to indicators, you should probably contact the creators of the original repository.

I do not describe some details that are not important to me. I am only interested in building UF2 bootloaders for nRF52 locally in Fedora or using GitHub Actions.

## My configs
* [nice_nano_LED](https://github.com/aroum/nRF52_Bootloader_custom_LED/tree/master/src/boards/nice_nano_LED)
* [nice_nano_RGB](https://github.com/aroum/nRF52_Bootloader_custom_LED/tree/master/src/boards/nice_nano_RGB)
* [kabarga](https://github.com/aroum/nRF52_Bootloader_custom_LED/tree/master/src/boards/kabarga)
* [PNCATEHO MK Dose](https://github.com/aroum/nRF52_Bootloader_custom_LED/tree/master/src/boards/pncateho_mk_dose)
* [mouse](https://github.com/aroum/nRF52_Bootloader_custom_LED/tree/master/src/boards/mouse)

The mouse config was originally published [here](https://github.com/greengrocer98/Adafruit_nRF52_Bootloader/commit/b9c9ca6f4b4f314b8ebd4e130bb9b60f82ed159d).

## Burn & Upgrade with pre-built binaries
In most cases, you just need to connect the keyboard to your computer, double-press the MCU reset button, and copy the UF2 bootloader file to the newly appeared device. If you need to flash the bootloader using a programmer, follow this [guide](https://github.com/joric/nrfmicro/wiki/Bootloader#swd-programmers).

## How to build

You should only continue if you are looking to develop bootloader for your own.
You must have have a J-Link available to "unbrick" your device.

## Creating a Configuration with Regular LEDs

To create your own bootloader variant for n!n, copy the `src/boards/nice_nano_LED` folder and rename it, for example, to `kabarga`. Then, edit the `board.h` file inside this folder.

Set `LED_PRIMARY_PIN` and `LED_SECONDARY_PIN` to the appropriate LED pins. Also, replace the following lines:

````
#ifndef _NICENANO_LED_H  
#define _NICENANO_LED_H  
````

with

```
#ifndef _KABARGA_H  
#define _KABARGA_H  
```
At the bottom of the file, you can change the device name.

Finally, build your bootloader using the command:

```BOARD=kabarga all```

The lighting parameters are configured in the `src/boards/boards.c` file, starting from line 365.

## Creating a Configuration with Addressable LEDs

To create your own bootloader variant for n!n, copy the `src/boards/nice_nano_RGB` folder and rename it, for example, to `kabarga_RGB`. Then, edit the `board.h` file inside this folder.

Set `LED_NEOPIXEL` and `LED_NEOPIXEL` to the appropriate LED pins. Set the `NEOPIXELS_NUMBER` parameter (number of LEDs) and `BOARD_RGB_BRIGHTNESS` (their brightness).

Also, replace the following lines:

```
#ifndef _NICENANO_RGB_H  
#define _NICENANO_RGB_H  
```

with

```
#ifndef _KABARGA_RGB_H  
#define _KABARGA_RGB_H  
```

At the bottom of the file, you can change the device name.

Finally, build your bootloader using the command:

```BOARD=kabarga-rgb all```

The lighting parameters are configured in the `src/boards/boards.c` file, starting from line 396.

## Building with GitHub Actions

* Fork this repository.
* Make the necessary changes.
* Commit and push.
* Enable GitHub Actions on the corresponding tab.
* Create an empty release with the tag 0.9.2.
* Wait for GitHub Actions to finish building your bootloader.

## Local Build

I use Fedora, but the commands will be similar on other distributions.

### Prerequisites

```
sudo dnf install python3-pip gcc-arm-none-eabi make arm-none-eabi-newlib arm-none-eabi-gcc-cs git --skip-unavailable    
pip3 install --user adafruit-nrfutil intelhex
```

### Build:

Firstly clone this repo with following commands

```
cd $HOME
git clone https://github.com/aroum/nRF52_Bootloader_custom_LED
cd nRF52_Bootloader_custom_LED
git submodule update --init
```

Then build it with `make BOARD={board} all`, for example:

```
make BOARD=feather_nrf52840_express all
```

For the list of supported boards, run `make` without `BOARD=` :

```
$ make
You must provide a BOARD parameter with 'BOARD='
Supported boards are: feather_nrf52840_express feather_nrf52840_express pca10056
Makefile:90: *** BOARD not defined.  Stop
```
Here, {board} matches the name of the configuration folder in `src/boards/{board}`.

The bootloader file can be found here: `_build/build-kabarga/update-kabarga_bootloader-XXX.uf2`

### Common makefile problems

If you get the following error ...

```
lib/sdk11/components/libraries/bootloader_dfu/bootloader_settings.c: In function 'bootloader_mbr_addrs_populate':
lib/sdk11/components/libraries/bootloader_dfu/bootloader_settings.c:45:7: error: array subscript 0 is outside array bounds of 'const uint32_t[0]' {aka 'const long unsigned int[]'} [-Werror=array-bounds=]
45 | if (*(const uint32_t *)MBR_BOOTLOADER_ADDR == 0xFFFFFFFF)
compilation terminated due to -Wfatal-errors.
cc1: all warnings being treated as errors
make: *** [Makefile:412: _build/build-feather_nrf52840_express/bootloader_settings.o] Error 1`
```

You are probably using arm-none-eabi-gcc version 14 or higher. You can check it with the following command:

```arm-none-eabi-gcc --version```

If the version is 15 or higher, you need to modify one line in the Makefile. Replace:

```filter 12.% 13.% 14.%,$```

with:

```filter 12.% 13.% 14.% 15.%,$```

Here is an example command to do this automatically:

```
sed -i 's/ifneq (,$(filter 12.% 13.% 14.%,$(shell \$(CC) -dumpversion 2>\/dev\/nul
l)))/ifneq (,$(filter 12.% 13.% 14.% 15.%,$(shell \$(CC) -dumpversion 2>\/dev\/null)))/g' "Makefile"
```

[original issues](https://github.com/adafruit/Adafruit_nRF52_Bootloader/issues/339)

---

## nRF52 P0.18 Pin Reassignment: Enabling GPIO Functionality on the Reset Pin

The P0.18 pin on the nRF52 series microcontrollers is typically configured as the dedicated Pin Reset input. However, its functionality is configurable via the `CONFIG_GPIO_AS_PINRESET` register, allowing it to be used as a standard General Purpose I/O (GPIO) pin.

This document details the procedure and constraints involved in enabling P0.18 as a GPIO pin for use in custom firmware, such as ZMK.

### 1. Enabling P0.18 as GPIO

To use P0.18 as a standard GPIO pin, the `CONFIG_GPIO_AS_PINRESET` definition must be explicitly disabled in the bootloader source code.

**Note:** This modification requires direct source code editing and a hardware programmer (e.g., J-Link, ST-Link) to flash the resulting bootloader image, as this option is not typically accessible through standard configuration files.

#### Required Source Code Modifications

The following changes must be applied to the bootloader's build configuration files:

| File Path                           | Modification                                                                 |
| ----------------------------------- | ---------------------------------------------------------------------------- |
| `src/boards/yourboard/board.h`             | Add the following directives to override the configuration:                  |
| `Makefile`                          | Remove the line defining the CFLAG for the reset configuration:              |
| `/lib/tinyusb/hw/bsp/nrf/family.mk` | Remove the line defining the CFLAG within this submodule's build definition: |

**`src/yourboard/board.h`**

```Python
#ifdef CONFIG_GPIO_AS_PINRESET
#undef CONFIG_GPIO_AS_PINRESET
#endif
```

**`Makefile`**

Locate and **remove** the following line:

```Python
CFLAGS += -DCONFIG_GPIO_AS_PINRESET
```

**/lib/tinyusb/hw/bsp/nrf/family.mk**

Locate and **remove** the following line:

```Python
  -DCONFIG_GPIO_AS_PINRESET
```

### 2. P0.18 Usage as a DFU Trigger

Once `CONFIG_GPIO_AS_PINRESET` is disabled, P0.18 can be configured within the bootloader to function as a DFU (Device Firmware Update) entry switch.

**Mechanism:** If P0.18 is tied to ground when power is applied to the microcontroller, the system will enter the flashing mode instead of running the application firmware.

### 3. P0.18 Usage in ZMK (Standard GPIO)

When configured as a GPIO, P0.18 becomes available for general purposes within ZMK, such as:

- Switching Bluetooth profiles.
- Battery charge indication.
- Triggering deep sleep mode.

### 4. Limitations and Workarounds

#### Deep Sleep Wake-up Conflict

Using P0.18 as a DFU trigger imposes a significant constraint on its ability to wake the microcontroller from deep sleep:

- **Conflict:** If P0.18 is configured as a simple external wake-up pin (shorting it to ground to wake the MCU), the action of grounding the pin will instead initiate DFU mode, preventing the application from resuming.
- **Exception (Matrix):** This issue does not apply if P0.18 is used as a column or row pin within a keyboard matrix, as the matrix logic handles the pin state.

#### Programmatic DFU Function Removal

To fully utilize P0.18 for deep sleep wake-up while retaining the modified bootloader:

- Use the `&bootloader` node configuration within ZMK's device tree.
- The ZMK application can then programmatically override and disable the DFU trigger functionality associated with P0.18.
- This allows the pin to be used reliably as a full-featured GPIO, including waking the MCU from deep sleep.

### 5. Board-Specific Notes (e.g., n!n v2)

On certain custom PCBs, such as the n!n v2, the P0.14, P0.16, and P0.18 pins are connected together for PCB routing simplicity. Developers should be aware of this hardware connection when assigning functions to any of these pins, as their state may influence each other.

### 6. Flashing and Deployment

- Due to the deep modifications made to the bootloader's configuration registers, the following deployment steps are mandatory:
- Local Compilation: The modified bootloader must be compiled locally to ensure the changes are correctly integrated.
- Using a Programmer: Flashing must be performed using a hardware programmer (e.g., J-Link, ST-Link).
- File Selection: The required .hex file is located in the build/yourboard/ directory. Select the file with the longest filename (this typically corresponds to the final, complete bootloader image).
- UF2 Restriction: Flashing the bootloader via .uf2 files will not correctly rewrite the necessary configuration registers, thus failing to enable P0.18 as a GPIO pin. A hardware programmer must be used.