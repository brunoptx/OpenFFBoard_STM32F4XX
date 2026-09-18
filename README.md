<div align="center">
    <a href="https://github.com/Ultrawipf/OpenFFBoard">
        <img width="200" height="200" src="doc/img/ffboard_logo.svg">
    </a>
	<br>
	<br>
	<div style="display: flex;">
		<a href="https://discord.gg/gHtnEcP">
            <img src="https://img.shields.io/discord/704355326291607614">
		</a>
		<a href="https://github.com/Ultrawipf/OpenFFBoard/stargazers">
            <img src="https://img.shields.io/github/stars/Ultrawipf/OpenFFBoard">
		</a>
		<a href="https://github.com/Ultrawipf/OpenFFBoard/actions/workflows/build-firmware.yml">
            <img src="https://github.com/Ultrawipf/OpenFFBoard/actions/workflows/build-firmware.yml/badge.svg?branch=master">
		</a>
	</div>
</div>

***
# OpenFFBoard - STM32F407ZG (Black Board) + BTS7960

This fork adapts the OpenFFBoard firmware for the generic **STM32F407ZG Black Board V3.0**, specifically configured to replace the original controller of a **Logitech Driving Force Pro** using a **BTS7960** H-Bridge motor driver.

## Hardware Mappings & Pinout

Due to physical hardware differences between the STM32F4 Discovery (default target) and the F407ZG Black Board, the peripherals have been remapped via STM32CubeMX.

### 1. Steering Wheel Encoder
Moved from TIM2 to **TIM3** to avoid conflicts with the physical `WK_UP` button and debounce capacitor present on PA0 on the Black Board.
* **Encoder A**: `PC6` (TIM3_CH1)
* **Encoder B**: `PC7` (TIM3_CH2)

### 2. Motor Driver (BTS7960)
* **L_PWM**: `PE9` (TIM1_CH1)
* **R_PWM**: `PE11` (TIM1_CH2)
* **L_EN / R_EN (Enable)**: `PD14` / `PD15` (Mapped as GPIO Output to enable the H-bridge).

### 3. Status LEDs
Remapped from the Discovery board layout to the onboard LEDs of the Black Board.
* **LED 1**: `PF9`
* **LED 2**: `PF10`
*(Note: The LEDs on this board are active-low, so they might behave inversely compared to the Discovery board).*

### 4. Emergency Stop (E-Stop)
* **E-Stop Pin**: `PD5` 
*(Important: This pin must be connected to **GND** if you are not using a physical Emergency Stop button, otherwise the firmware will block motor initialization and trigger the Error LED).*


***


# Open FFBoard
The Open FFBoard is an open source force feedback interface with the goal of creating a platform for highly compatible FFB simulation devices like steering wheels and joysticks.

This firmware is optimized for the Open FFBoard and mainly designed for use with DD steering wheels.
Remember this software is experimental and is intended for advanced users. Features may contain errors and can change at any time.

More documentation about this project is on the [hackaday.io page](https://hackaday.io/project/163904-open-ffboard).

The hardware designs are found under [OpenFFBoard-hardware](https://github.com/Ultrawipf/OpenFFBoard-hardware).

The GUI for configuration is found at [OpenFFBoard-configurator](https://github.com/Ultrawipf/OpenFFBoard-configurator).

These git submodules can be pulled with `git submodule init` and `git submodule update`

Updates often require matching firmware and GUI versions!
For binary releases check the [github release section](https://github.com/Ultrawipf/OpenFFBoard/releases).

## Documentation
Documentation will be updated in the [GitHub Wiki](https://github.com/Ultrawipf/OpenFFBoard/wiki).

Available commands are listed on the [Commands wiki page](https://github.com/Ultrawipf/OpenFFBoard/wiki/Commands)

Code summary and documentation of the latest stable version is available as a [Doxygen site](https://ultrawipf.github.io/OpenFFBoard/doxygen/).

For discussion and progress updates we have a [Discord server](https://discord.com/invite/gHtnEcP).


## Features
For more information, [game compatibility](https://github.com/Ultrawipf/OpenFFBoard/wiki/Games-setup#compatibility-list) and common setups check the [GitHub Wiki](https://github.com/Ultrawipf/OpenFFBoard/wiki).
### Main modes:
See [other mainclasses](https://github.com/Ultrawipf/OpenFFBoard/wiki/Commands#other-mainclasses) for full list.
* **FFB Wheel**: 	USB 1 Axis force feedback device with HID FFB support, multiple analog axis inputs (see analog sources) and digital buttons (see digital sources)
* **FFB Joystick**: USB 2 Axis force feedback device

* **EXT FFB Gamepad**: USB 2 axis gamepad device without HID FFB. Use this for simple non FFB devices or send custom FFB data via commands (See [Ext FFB mode](https://github.com/Ultrawipf/OpenFFBoard/wiki/External-FFB-mode))
* **CAN Remote Analog/Digital**: Send all supported digital and analog inputs as CAN packets to a main FFBoard (Receive using CAN Analog/Digital source)
---
* **CAN Interface**: GVRET compatible CAN interface for CAN bus debugging

### Supported motor drivers
|Name|Interface|1ch Wheel support|2ch Joystick support|Supported motors|Supported encoders|Enc forwarding (see [FFBoard supported encoders](#supported-encoders))|Dual interface encoder
|--|--|--|--|--|--|--|--|
**OpenFFBoard TMC4671**|SPI|yes✅|limited⚠️|3 phase servo (BLDC), 2ph stepper (w. encoder), (DC) | ABZ, SinCos, (Digital Hall), Analog hall | yes✅|yes✅
ODrive|CAN|yes✅|yes✅|3 phase servo | ABZ, SPI, Internal (See ODrive docs) | no❌|no❌
VESC|CAN|yes✅|yes✅|3 phase servo| ABZ | no❌|yes✅
PWM|PWM pins|yes✅|no❌|Any external driver (PWM+Dir, Centered, RC PPM, 2x PWM)| Any [FFBoard encoder](#supported-encoders) | no❌|no❌
MyActuator|CAN|yes✅|yes✅|Integrated in servo| Internal | no❌|no❌
Simplemotion|RS432 w. adapter⚠️|yes✅|no❌|3 phase servo, Stepper| ABZ, BISS-C | no❌|no❌


### Supported encoders
Encoders supported directly by FFBoard firmware and main board

Can be forwarded to TMC4671 using external encoder forwarding

|Name|Interface|Absolute|
|-|-|-|
|ABZ Quadrature|ABZ|no❌|
|BISS-C|SPI w. Adapter⚠️|yes✅|
|MagnTek (MT6825,MT6835)|SPI + ABZ|yes✅|
|SSI|SPI|yes✅|


### Extensions
The modular structure means you are free to implement your own main classes.
Take a look into the FFBoardMain and ExampleMain class files in the UserExtensions folder.
Helper functions for parsing CDC commands and accessing the flash are included.

The firmware is class based in a way that for example the whole main class can be changed at runtime and with it for example even the usb device and complete behavior of the firmware.

For FFB the motor drivers, button sources or encoders also have their own interfaces.

A unified command system supporting different interfaces is available and recommended for setting parameters at runtime. (see `CommandHandler.h` and the example mainclass)


### Copyright notice:
Some parts of this software may contain third party libraries and source code licenced under different terms.
The license applying to these files is found in the header of the file.
For all other parts in the `Firmware/FFBoard` folder the LICENSE file applies.
