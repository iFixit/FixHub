# USB Hub and Iron Command Line Interface Documentation

The iFixit FixHub power station and smart soldering iron provide a serial interface for low-level communications.

Hardware damage caused by inventive tinkering is not covered under our warranty. But it's your FixHub, have fun! We left the locks off and the pins probe-able.

Improvements to this documentation are welcome; open a pull request.

## Introduction

This document provides information on the command-line interfaces (CLI) for two separate tools:
1. A USB hub with one charge port in the back and two tool ports in the front
2. A smart iron tool

Each tool has its own set of commands, which are detailed in their respective sections below.
# Command Line Interface Documentation

1. [Common Commands (Hub and Iron)](#common-commands)
   1. [adc](#adc)
   2. [bootloader](#bootloader)
   3. [comms](#comms)
   4. [gpio](#gpio)
   5. [help](#help)
   6. [i2c](#i2c)
   7. [log](#log)
   8. [logging](#logging)
   9. [mcu_sn](#mcu_sn)
   10. [ob](#ob)
   11. [otp](#otp)
   12. [pubsub](#pubsub)
   13. [reset](#reset)
   14. [uptime](#uptime)
   15. [usbpd](#usbpd)
   16. [version](#version)

2. [Hub-Specific Commands](#hub-commands)
   1. [errorlog](#errorlog)
   2. [hwid](#hwid)
   3. [idle](#idle)
   4. [max17205](#max17205)
   5. [pdmcudfu](#pdmcudfu)
   6. [pwrsrc](#pwrsrc)
   7. [rt9490](#rt9490)
   8. [shutdown](#shutdown)
   9. [toolcomms](#toolcomms)
   10. [ui](#ui)

3. [Iron-Specific Commands](#iron-commands)
   1. [heater](#heater)
   2. [mxc4005](#mxc4005)
   3. [settings](#settings)
   4. [thermistor](#thermistor)
   5. [thermocouple](#thermocouple)

---

## adc

Commands related to analog-to-digital conversion.

### Usage
```
adc [subcommand]
```

### Subcommands
- `stream <device>`: Continuously stream ADC values from the specified device
  - Not available on production firmware
- `read <device>`: Read a single ADC value from the specified device
- `list`: List all controllable ADCs
  - Not available on production firmware

---

## bootloader

Reboot into MCUBoot serial recovery mode.

### Usage
```
bootloader reboot
```

### Description
Replies with 'OK' then reboots the device into MCUBoot serial recovery mode.

---

## comms

MCU - MCU Communication Commands

### Usage
```
comms [subcommand]
```

### Subcommands
- `tx-test`: Transmit the Test command
- `g0version`: Get the G0's FW version
 - On error: 'Timeout. Is the other MCU subscribed?'

---

## errorlog

ErrorLog commands

### Usage
```
errorlog [subcommand]
```

### Subcommands
- `clear`: Clear the ErrorLog
- `write <fault>`: Write a fault to the ErrorLog
  - Not available on production firmware

---

## gpio

GPIO commands

### Usage
```
gpio [subcommand]
```

### Subcommands
- `write <name> <0|1>`: Set a gpio state
- `read <name>`: Read a gpio state
- `read <name>`: List gpios
  - Not available on production firmware

Caution: changing GPIO values can cause unexpected behavior. GPIO writes are reverted on restart.
GPIO names are derived from the device tree and are printed using the cmd_gpio_list function.

---

## help

Lists available commands.

### Usage
```
help
```

---

## hwid

Get the hardware ID of the device.

### Usage
```
hwid get
```
Ex: `hwid get`: `DVT2`

---

## i2c

I2C commands

### Usage
```
i2c [subcommand]
```

### Subcommands
- `scan <device>`: Scan I2C devices on the specified bus

Ex:
`i2c scan I2C_1`

```
     0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f
00:             -- -- -- -- -- -- -- 0b -- -- -- -- 
10: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- 
20: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- 
30: -- -- -- -- -- -- 36 -- -- -- -- -- -- -- -- -- 
40: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- 
50: -- -- -- 53 -- -- -- -- -- -- -- -- -- -- -- -- 
60: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- 
70: -- -- -- -- -- -- -- --                         
3 devices found on I2C_1
```
---

## idle

Get idle timeout parameters

### Usage
```
idle get
```

Ex: `idle get`
```
Idle timer running
   Initial seconds: 1800 S
 Remaining seconds: 1405 S
  Already expired?: No
```

---

## log

Logging related commands.

### Usage
```
log [subcommand]
```

### Subcommands
- `show`: Display current log
- `level <level>`: Set log level (e.g., INFO, DEBUG, ERROR)

Not working in production.

---

## logging

Shell UART logging backend commands

### Usage
```
logging [subcommand]
```

### Subcommands
- `enable`: Enable the shell UART logging backend
- `disable`: Disable the shell UART logging backend

---

## max17205

MAX17205 Fuel Gauge Commands

### Usage
```
max17205 [subcommand]
```

### Subcommands

1. `read <virtual-device-number> <8b-register-address>`
   - Description: Read from the MAX17205 registers
   - Parameters:
     - `<virtual-device-number>`: The virtual device number (0 or 1)
     - `<8b-register-address>`: The 8-bit register address to read from (in hexadecimal)
   - Note: Valid virtual device numbers are 0 or 1
   - Example: `max17205 read 0 0x0D`

2. `configure`
   - Description: Perform the configuration routine for the MAX17205 Fuel gauge
   - Note: This command completes the configuration but does not store values in NVM
   - Example output:
     ```
     Config complete, but registers values are not stored in NVM
     Use the 'nvm_programming' command to update the NVM
     ```

3. `details`
   - Description: Display detailed information about the current state of the MAX17205 fuel gauge
   - Example output:
     ```
     Pack voltage: 11671mV
     Pack current: -3mA
     Pack soc: 79%
     IC die temp: 50C
     NTC1 temp: 54C
     NTC2 temp: 47C
     Battery state: Dischg Warning Hot
     ```

4. `nvm_updates`
   - Description: Determine the number of NVM updates remaining
   - Note: This command shows how many times the NVM has been updated and how many updates are left

5. `nvm_programming`
   - Description: Program the Fuel gauge NVM
   - Usage: `nvm_programming confirm`
   - Note: Use this command with extreme caution as it permanently alters the fuel gauge's configuration
   - Example output:
     ```
     Usage: nvm_programming confirm
     DO NOT run this command if you don't know what it does
     ```

6. `hardreset`
   - Description: Perform a reset of the fuel gauge
   - Usage: `hardreset confirm`
   - Note: This command performs a reset of the fuel gauge just like if the battery pack was removed and reinstalled. It doesn't reset the fuel gauge to the default state (it doesn't alter non-volatile memory, for instance). 

7. `health`
   - Description: Get the Battery health
   - Example output: `100 %`

8. `version`
   - Description: Get the MAX17205 Fuel gauge configuration version
   - Example output: `1`


### Notes
- The MAX17205 is a sophisticated fuel gauge IC that provides accurate battery state-of-charge (SOC) and other battery monitoring functions.
- Always exercise caution when using commands that modify the fuel gauge's configuration or perform resets, as these actions may affect the device's performance or battery monitoring accuracy.
- The `read` command allows direct access to the fuel gauge's registers. Refer to the MAX17205 datasheet for the meaning and location of specific registers.
- The `configure` command sets up the fuel gauge but doesn't store values in non-volatile memory (NVM). Use `nvm_programming` to update the NVM after configuration.
- The `test_params` command: DEPRECATED. This command is removed from current firmware. Incorrect use of this command can cause hardware problems.
- The `health` command provides a percentage indicating the overall health of the battery.
- The `version` command returns the configuration version of the fuel gauge, which is used for tracking fuel gauge configuration changes.

---

## mcu_sn

Get the MCU serial number.

### Usage
```
mcu_sn
```
Ex: `mcu_sn`
```
0026002C-58305007-20343452
```

---

## pubsub

Low Level USART commands

### Usage
```
pubsub [subcommand]
```

### Subcommands
- `enable <device>`: Enable control
- `disable <device>`: Disable control

---

## pwrsrc

Get the power source of the board

### Usage
```
pwrsrc get
```

Ex:
pwrsrc get
```
Rear
```

---

## reset

Reset the device.

### Usage
```
reset
```

---

## rt9490

RT9490 PMIC (Power Management Integrated Circuit) Commands

### Usage
```
rt9490 [subcommand]
```

### Subcommands

...

4. `get_adc <dev>`
   - Description: Read the value from a specific ADC channel of the RT9490 PMIC.
   - Usage: `rt9490 get_adc <dev>`
   - Parameters:
     - `<dev>`: The name of the ADC device to read from.
   - Example usage:
     ```
     rt9490 get_adc RT9490_ADC_VBUS
     rt9490 get_adc RT9490_ADC_IBAT
     ```
   - Note: Use the exact ADC names as shown in the `all_adc` command output.

5. `all_adc`
   - Description: Display readings from all ADC channels of the RT9490 PMIC.
   - Usage: `rt9490 all_adc`
   - Output: This command provides a comprehensive list of all ADC readings, including:
     - RT9490_ADC_IBUS: Input bus current (mA)
     - RT9490_ADC_IBAT: Battery current (mA)
     - RT9490_ADC_VBUS: Input bus voltage (mV)
     - RT9490_ADC_VAC1: AC input voltage 1 (mV)
     - RT9490_ADC_VAC2: AC input voltage 2 (mV)
     - RT9490_ADC_VBAT: Battery voltage (mV)
     - RT9490_ADC_VSYS: System voltage (mV)
     - RT9490_ADC_TS: Temperature sensor reading (°C)
     - RT9490_ADC_TDIE: Die temperature (°C)

   - Example output:
     ```
     RT9490_ADC_IBUS: 25mA
     RT9490_ADC_IBAT: 63mA
     RT9490_ADC_VBUS: 5258mV
     RT9490_ADC_VAC1: 0mV
     RT9490_ADC_VAC2: 0mV
     RT9490_ADC_VBAT: 12605mV
     RT9490_ADC_VSYS: 12905mV
     RT9490_ADC_TS  : 0C
     RT9490_ADC_TDIE: 27C
     ```

### Notes
- ADC readings can help in understanding the current power state of the system, including battery status, power consumption, and temperature.

---
## toolcomms

Display the active connection status of iFixit tools on the device ports.

### Usage
```
toolcomms
```

### Description
This command checks and displays the connection status of iFixit tools on the device's ports. It shows whether an active iFixit tool is plugged into each port.

### Output Format
```
Port1: [status]
Port2: [status]
```
Where:
- `Port1` refers to the port on the right side of the device.
- `Port2` refers to the port on the left side of the device.
- `[status]` is either 0 (no active tool connected) or 1 (active tool connected).

### Interpretation
- 0: No active iFixit tool is plugged into the port.
- 1: An active iFixit tool is plugged into the port and communicating properly.

### Example Outputs

1. No tools connected:
   ```
   Port1: 0
   Port2: 0
   ```

2. Tool connected to right port only:
   ```
   Port1: 1
   Port2: 0
   ```

3. Tools connected to both ports:
   ```
   Port1: 1
   Port2: 1
   ```

---

## ui

UI Commands

### Usage
```
ui [subcommand]
```

### Subcommands
- `record`: Record UI events
- `unsubscribe`: Unsubscribe from UI events

---

## uptime

Get system uptime in milliseconds.

### Usage
```
uptime
```

Ex: `uptime`: `304993`

---
## usbpd

USB Power Delivery (USB-PD) related commands for monitoring and managing USB-PD ports.

### Usage
```
usbpd [subcommand]
```

### Subcommands

1. `status`
   - Description: Display detailed status information for USB-PD ports.
   - Usage: `usbpd status`
   - Output: Provides comprehensive information about each USB-PD port, including:
     - LoadSwitch status (En/Off)
     - Attachment status (Y/N)
     - Port role (UFP/DFP)
     - Orientation (Swapped/Normal)
     - Explicit Contract status (Y/N)
     - Power role (SNK/SRC)
     - Partner Known status (Y/N)
     - Discovery Done status (Y/N)
     - iFixit AltMode status (Y/N)
     - Force Detach status (Y/N)
     - This Device's Source (SRC) and Sink (SNK) Power Data Objects (PDOs)
     - Port Partner's SRC and SNK PDOs
     - Currently used PDO with detailed information

   Example output:
   ```
   Port 1:
            LoadSwitch: En
              Attached: Y
                  Port: UFP
   Orientation Swapped: Y
     Explicit Contract: Y
                 Power: SNK
         Partner Known: N
        Discovery Done: Y
        iFixit AltMode: N
          Force Detach: N
   This Device's SRC PDOs
   This Device's SNK PDOs
          Fixed:  5000mV @ 3000mA   NoDRS NoDRP
          Fixed:  9000mV @ 3000mA   NoDRS NoDRP
          Fixed: 15000mV @ 3000mA   NoDRS NoDRP
          Fixed: 20000mV @ 2250mA   NoDRS NoDRP
   Port Partner's SRC PDOs
   -->    Fixed:  5000mV @ 3000mA     DRS   DRP
   Port Partner's SNK PDOs
   Object Used
       [1](0x2F01912C) => Fixed:  5000mV @ 3000mA     DRS   DRP   USB | mismatch? N
   ```

2. `vbus`
   - Description: Display current voltage and current for each USB-PD port.
   - Usage: `usbpd vbus`
   - Output: Shows the voltage (in mV) and current (in mA) for each port.

   Example output:
   ```
   Port1:  5215mV @  243mA
   ```

3. `summary`
   - Description: Provide a summary of the state and power consumption for all USB-PD ports.
   - Usage: `usbpd summary`
   - Output: Displays the state (as a hexadecimal value) and power consumption (in mW) for each port.

   Example output:
   ```
   TOOLPORT1 state: 0x05ee
   TOOLPORT1 power: 238 mW
   TOOLPORT2 state: 0x07ee
   TOOLPORT2 power: 1479 mW
   REAR state: 0x049e
   REAR power: 10014 mW
   ```

### Notes
- The `status` command provides the most detailed information and is useful for debugging USB-PD issues.
- The `vbus` command gives a quick view of the current power delivery situation on each port.
- The `summary` command is helpful for a quick overview of all ports' states and power consumption.

### Interpretation
- In the `status` output:
  - UFP: USB Upstream Facing Port (in this case, the device is acting as a sink)
  - SNK: Power Sink (confirming the device is receiving power)
  - PDO: Power Data Object, describing power capabilities
  - DRS: Data Role Swap supported (seen in the port partner's capabilities)
  - DRP: Dual Role Power supported (seen in the port partner's capabilities)

- The `vbus` output shows the actual voltage and current being delivered. In the example, Port1 is receiving 5215mV at 243mA.

- The `summary` output provides:
  - State information as a hexadecimal value for each port (e.g., 0x05ee, 0x07ee, 0x049e). The meaning of these specific values would require further information from the system documentation or source code.
  - Power consumption in mW for each port, allowing quick comparison between ports.

- The system recognizes three ports: TOOLPORT1, TOOLPORT2, and REAR.

Notes:
- The "Explicit Contract: Y" indicates that power negotiation has successfully completed.

---

## version

Display detailed version and build information about the system.

### Usage
```
version
```

### Description
This command provides information about the software versions, build environment, and platform details of the system.

### Output Format
The command returns the following information:

```
Zephyr Kernel Version: [kernel_version]
    Zephyr Kernel Git: [kernel_git_version]
              App Git: [app_git_hash]
         Compile Time: [compilation_timestamp]
           Compile OS: [build_os]
             Platform: [board_name] ([soc_name])
       Compile Author: [author_name]
             Compiler: [compiler_name_and_version]
           Build Type: [build_type]
```

### Field Descriptions

- `Zephyr Kernel Version`: The version of the Zephyr RTOS kernel being used.
- `Zephyr Kernel Git`: The Git commit or tag of the Zephyr kernel.
- `App Git`: The Git commit hash of the application code.
- `Compile Time`: The timestamp when the software was compiled.
- `Compile OS`: The operating system where the compilation was performed.
- `Platform`: The name of the board and the specific SoC (System on Chip) being used.
- `Compile Author`: The name or identifier of the user who compiled the software.
- `Compiler`: The name and version of the compiler used to build the software.
- `Build Type`: The type of build (e.g., debug, release).

### Example Output
```
Zephyr Kernel Version: 3.6.0
    Zephyr Kernel Git: v3.6.0-91-gace6e447cf3d
              App Git: 3009749
         Compile Time: 2024-07-14T05:45:37
           Compile OS: Linux-6.5.0-1023-azure
             Platform: ifixit_smarthub_l5 (stm32l552xx)
       Compile Author: runner
             Compiler: GCC 12.2.0
           Build Type: release
```

### Usage
```
version
```
# Iron commands

## heater

Commands related to the soldering iron heater control and monitoring.

### Usage
```
heater [subcommand]
```

### Subcommands

1. `details`
   - Description: Display detailed information about the heater's current state and settings.
   - Usage: `heater details`
   - Output: Provides comprehensive information about the heater, including:
     - Temperature settings (setpoint, active, idle)
     - Current temperature readings (thermocouple, raw, filtered)
     - PID control parameters and current values
     - Power and duty cycle information
     - Heater and tip status

   Example output:
   ```
            setpoint temp: 50 C
              active temp: 350 C
                idle temp: 200 C
   current thermocouple temp: 27.99 C
         current raw temp: 21.55 C
    current filtered temp: 21.55 C
                  delta_C: 0.00 C
       delta_C_integrated: 0.00 C
       delta_C_derivative: 0.00 C
             tip power mW: 14750
     pid output desired W: 0
           pwm duty cycle: 0
               switch on?: No
           tip installed?: Yes
                    state: Off
               buck state: High power
           control loop P: 1.00
           control loop I: 1.00
           control loop D: 0.10
      integral windup max: 40.00
      control loop period: 300000 uS
   ```

2. `controlloop`
   - Description: Likely intended to provide real-time control loop data or adjust control loop parameters.
   - Usage: `heater controlloop`
   - Note: This subcommand currently locks up the serial interface. Use with caution.

3. `tip_installed_uptime`
   - Description: Display the uptime (in seconds) since the current tip was installed or last detected.
   - Usage: `heater tip_installed_uptime`
   - Output: A single integer representing the number of seconds.

   Example output:
   ```
   308
   ```
### Caution
- Altering heater settings or behavior could affect soldering performance and potentially pose safety risks. Always follow proper safety guidelines when working with heating elements.

## mxc4005

Commands related to the MXC4005 accelerometer.

### Usage
```
mxc4005 [subcommand]
```

### Subcommands

1. `magnitude`
   - Description: Display the current magnitude of acceleration detected by the MXC4005 accelerometer.
   - Usage: `mxc4005 magnitude`
   - Output: A single integer value representing the magnitude of acceleration.
   
   Example output:
   ```
   1062
   ```

    ### Used to calculate Freefall and Idle status
    - The threshold for the iron being considered in freefall is configured here: https://github.com/iFixit/ifixit-schooner-fw/blob/a248b91345161b3ef3e2e7005d969e072421ee5d/app/src/mxc4005xc.h#L29
    - The calculation for detecting if the iron is idle is located here: https://github.com/iFixit/ifixit-schooner-fw/blob/a248b91345161b3ef3e2e7005d969e072421ee5d/app/src/mxc4005xc.c#L191-L192

### Use Cases
- Detecting movement or orientation changes of the device.


Thank you for providing that output. I'll update the documentation to reflect this more accurate information:

## thermocouple

Commands related to the thermocouple temperature sensor in the soldering iron.

### Usage
```
thermocouple [subcommand]
```

### Subcommands

1. `get`
   - Description: Display the current temperature reading from the thermocouple sensor.
   - Usage: `thermocouple get`
   - Output: Provides the temperature reading in Celsius from the thermocouple sensor.

   Example output:
   ```
   THERMOCOUPLE: 27.13 C
   ```

### Notes
- The thermocouple is used for precise temperature measurements, especially for high-temperature applications like the soldering iron tip.

## thermistor

Commands related to the thermistor temperature sensor in the soldering iron.

### Usage
```
thermistor [subcommand]
```

### Subcommands

1. `get`
   - Description: Display the current temperature reading from the cold junction thermistor sensor.
   - Usage: `thermistor get`
   - Output: Provides the temperature reading in Celsius from the cold junction thermistor.

   Example output:
   ```
   COLD_JUNC_TEMP: 27.23 C
   ```

### Notes
- The thermistor in this case is specifically used to measure the cold junction temperature.
- The cold junction temperature is important for accurate thermocouple readings, as it's used for temperature compensation.
- Temperature readings are provided in degrees Celsius.

### Additional Information
The thermistor (cold junction) and thermocouple work together to provide accurate temperature measurements:
- The thermocouple measures the temperature at the tip of the soldering iron.
- The thermistor measures the temperature at the cold junction (where the thermocouple connects to the measuring circuit).
- The cold junction temperature is used to compensate for the thermocouple reading, improving overall accuracy.

Using both commands together can give you a complete picture of the temperature measurement system in the soldering iron.

---
