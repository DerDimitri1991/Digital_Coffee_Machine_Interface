# Digital_Coffee_Machine_Interface

An Android-based control and monitoring system for a modified **De'Longhi Magnifica ESAM3200** coffee machine.

The project extends the machine's normal user interface with
an Android application connected to a machine-side controller over
Bluetooth. The Android app can operate the coffee-machine buttons, set
coffee intensity and coffee amount, display the machine LEDs and status
information, store drink presets, monitor water amount, water and cleaning age, detect
a cup, and guide the user through maintenance procedures.

> **Important:** This build modifies the coffee machine's original control PCB.
> Replicate it only on a matching machine/board, using the annotated
> PCB photographs and final schematic. Do not connect controller GPIO directly
> to unknown machine signals or mains-voltage circuitry.

## Android app and source code

Download the **Android Studio project** and **Android APK** here:

**[Google Drive — Android source code and APK](https://drive.google.com/drive/folders/1SdYM_TjMqACc8H_o7WoYMGXB4GDJf7PA?usp=sharing)**

The app requires the matching modified machine hardware and controller firmware.

## Reproducing the hardware modification

The detailed, illustrated **Coffee Machine Hardware Modification Guide** contains
photographs of the control board, annotated solder points and trace cuts,
optocoupler-module modifications, and the final wiring schematics.


### Hardware used in this build

| Component | Model | Quantity |
| --- | --- | ---: |
| Coffee machine | De'Longhi Magnifica ESAM3200 | 1 |
| Main controller | LOLIN32 Lite | 1 |
| 8-channel isolation modules | PC817 modules, modified as described below | 2 |
| 4-channel physical-button input module | PC817 module, unmodified | 1 |
| I²C isolator | ISO1540 | 1 |
| Real-time clock | PCF8563 | 1 |
| Ultrasonic distance sensor | GL-A22 | 1 |
| I²C multiplexer | PCA9546 (4-channel) | 1 |
| I²C I/O expander | MCP23017 (16-bit) | 1 |
| Infrared object sensor | E18-D80NK | 1 |
| Digital potentiometers | MCP4018 | 2 |
| ADC | ADS1115 | 1 |
| AC–DC supply | 5 V module | 1 |

### 1. Modify the original control board

1. Unplug the machine and let it cool. Access the original control board.
2. Carefully remove the protective hot glue without damaging the board.
3. **Cut the marked PCB traces** as shown in the annotated control-board photo (guide page 4).
4. Solder the **COFFEE_5V** and **COFFEE_GND** wires at their annotated points. The button supply is taken from the machine control board's 5 V connection.
5. Wire the physical-button signals, LED signals, potentiometer wipers, and digital-potentiometer connections exactly as labeled in the guide and schematic.
6. Keep exposed wire ends short, inspect for solder bridges, and secure wires with strain relief and hot glue after checking connections.

The control-board connection labels include `B_1Cup`, `B_2Cup`, `B_Power`,
`B_Steam`, `B_Rinse`, `B_ECO`, `Poti_Intensity`, `Poti_Amount`,
`Intensity_Poti_Board`, `Amount_Poti_Board`, `Q1`, `Q2`, `R14`–`R18`,
and button-simulation connections `D1`, `D2`, `R24`–`R26`.
**The buttons are multiplexed.**

### 2. Modify the optocoupler modules

- **8-channel button module:** Cut the shared GND traces **on both sides** to create electrically separated **2-channel and 6-channel groups**. The two channels detect physical button switches alongside the 4-channel PC817 module; the six channels simulate button presses. Replace the required PC817 optocouplers with **G3VM-61A1 PhotoMOS relays**, and replace their input resistors with **200 Ω** resistors.
- **8-channel LED module:** Cut the shared GND traces **on both sides** to create electrically separated **3-channel and 5-channel groups**.
- **4-channel PC817 module:** Used for physical button inputs **without modification**.

Verify the GND separations using a multimeter before connecting the modules.
Use the guide photographs and schematic for the exact cuts and orientation.

### 3. Connect the custom PCBs

**Custom PCB 1 — LOLIN32 Lite controller:** Routes controller pins to terminal
blocks and carries the PCF module connected over I²C. The custom PCB and PCF
module use the **LOLIN32 Lite 3.3 V supply**.

**Custom PCB 2 — ADC and digital potentiometers:** Routes inputs and outputs to
terminal blocks and holds the ADC, digital potentiometers and isolated I²C
interface. Its module-side I²C, VCC and GND connections are wired as shown in
the schematic. This PCB and its modules use **5 V from the coffee-machine
control board**. Preserve the I²C isolation boundary shown in the schematic;
do not join isolated grounds across it.

**Final wiring:** Connect everything according to the schematic; Qwiic
connectors were used where practical. **Wire colours in build photos do not
always match the schematic** because the build evolved over time. Follow the
**labels and schematic**, not the wire colours.

### 4. Inspect before powering up

- Confirm every trace cut and solder point against the annotated photographs.
- Check for shorts, solder bridges, and unintended continuity across isolated GND groups.
- Verify PhotoMOS orientation and the 200 Ω input resistors on modified channels.
- Confirm correct 3.3 V and 5 V power domains and the I²C isolation boundary.
- Verify that button outputs remain inactive at startup and on controller reset.


## What is implemented in the coffee machine

To reproduce the complete system, the coffee machine needs a controller
that provides the following functions:

1.  Bluetooth serial communication.
2.  Electronic emulation of the machine's front-panel buttons.
3.  Reading of machine LED/status signals.
4.  Adjustable intensity control from `0` to `100`.
5.  Adjustable beverage amount from `0` to `160 ml`.
6.  Water-level and water-age tracking.
7.  Cleaning-age/status tracking.
8.  Cup-presence detection.
9.  Physical-control enable/disable.
10. Persistent logging.
11. Date/time synchronization from the Android device.

The Android application is already designed around this interface.


## 1. Button interface

The Android app exposes these machine controls:

``` text
Power ON/OFF
Steam ON/OFF
One Cup
Two Cups
Eco mode ON/OFF
Rinsing
```

A normal button operation is sent as:

``` text
press power
press steam
press one
press two
press eco
press rinsing
```

The controller reproduces a normal physical button press on the
corresponding coffee-machine input.

The app can also request a long press:

``` text
longpress power
longpress steam
longpress one
longpress two
longpress eco
longpress rinsing
```

The exact long-press duration is implemented on the machine-side
controller. It matches the duration expected by the original
machine. For example, the application uses a long press of the rinsing
control to enter the machine's descaling procedure.

## 2. LED/status inputs

The Android application expects exactly eight LED values in this order:

    Index Meaning
  ------- ------------------------
        0 Alarm
        1 Water
        2 Trash / puck container
        3 Steam
        4 Two Cups
        5 One Cup
        6 Eco
        7 Rinsing / descaling

Transmit the LED state as:

``` text
L,<alarm>,<water>,<trash>,<steam>,<two>,<one>,<eco>,<rinsing>
```

Example:

``` text
L,0,0,0,0,0,1,0,0
```

Each value is `0` or `1`.

The app interprets combinations and timing of these signals to identify
states such as:

-   machine ready,
-   preparing one cup,
-   preparing two cups,
-   steam heating,
-   steam ready,
-   door open,
-   water tank empty,
-   puck container full,
-   descaling request/mode,
-   slow coffee flow,
-   failed coffee preparation,
-   missing infuser,
-   heater too hot.

## 3. Intensity and beverage amount

The Android app sends:

``` text
I <value>
A <value>
```

where:

``` text
I = intensity, 0 ... 100
A = beverage amount, 0 ... 160 ml
```

Examples:

``` text
I 70
A 40
```

The controller reports the current values back as:

``` text
P,<intensity>,<amount>
```

Example:

``` text
P,70,40
```

## 4. Water monitoring

The app assumes a nominal tank capacity of:

``` text
1500 ml
```

The controller reports:

``` text
W,<water_ml>,<age>,<status>
```

Example:

``` text
W,1250,1d 4h,OK
```

For old water:

``` text
W,900,3d 2h,OLD
```

The app displays the amount either in millilitres or percent and warns
the user when the controller reports `OLD`.

## 4. Cup sensor

The controller reports cup state with:

``` text
CUP,<present>,<connection>
```

Examples:

``` text
CUP,1,ONLINE
CUP,0,ONLINE
CUP,0,OFFLINE
```

Meaning:

-   `1` = cup detected
-   `0` = no cup detected
-   `ONLINE` = cup sensor is operating/available
-   `OFFLINE` = cup sensor is unavailable

When no cup is detected, the Android app warns before starting a one-cup
or two-cup operation, while still allowing the user to override the
warning.

## 5. Cleaning state

The controller reports machine-cleaning information as:

``` text
M,<age>,<status>
```

Examples:

``` text
M,2d 6h,clean
M,8d 1h,not clean
```

The Android app calculates a seven-day cleaning progress from the
supplied age. At seven days or more, cleaning is treated as overdue.

When the user completes the cleaning workflow, the app sends:

``` text
clean
```

The controller uses this command to reset the stored cleaning
timestamp/status.

The cleaning timestamp is stored persistently so it survives
controller restarts.

## 6. Date and time synchronization

Immediately after connecting, the Android application sends the current
phone date/time:

``` text
time YYYY-MM-DD HH:mm:ss
```

Example:

``` text
time 2026-09-29 14:30:00
```

## 7. Physical front-panel controls

The Android application supports enabling or disabling the machine's
physical controls.

Commands:

``` text
phis on
phis off
```

The controller reports the current state as:

``` text
S,1
```

or:

``` text
S,0
```

where `1` means physical controls are enabled.


## 8. Logging

The app can request:

``` text
log stats
```

and delete logs with:

``` text
log delete
```

The expected response starts with:

``` text
LOG STATS
```

followed by lines in this general form:

``` text
YYYY-MM-DD: ... terminal=<number> ... physical=<number> ... total=<number>
```

## 9. Android application setup

Open the Android project in Android Studio and build it as a normal
Kotlin/Jetpack Compose application.

The application requires Bluetooth permissions. On Android 12 and newer
it uses:

``` text
BLUETOOTH_CONNECT
BLUETOOTH_SCAN
```

After installing the app:

1.  Power the machine-side controller.
2.  Ensure Bluetooth is enabled on the Android device.
3.  Give the app the requested Bluetooth permissions.
4.  Start the app.
5.  The app scans for `Coffee_Machine`.
6.  Select the discovered device.
7.  The app opens the connection.
8.  The current phone time is sent automatically.
9.  The controller should begin sending machine-state packets.


## 10. Protocol summary

### Android -\> controller

``` text
press power
press steam
press one
press two
press eco
press rinsing

longpress power
longpress steam
longpress one
longpress two
longpress eco
longpress rinsing

I <0..100>
A <0..160>

clean

time YYYY-MM-DD HH:mm:ss

phis on
phis off

log stats
log delete
```

### Controller -\> Android

``` text
L,<alarm>,<water>,<trash>,<steam>,<two>,<one>,<eco>,<rinsing>

P,<intensity>,<amount>

W,<water_ml>,<age>,<OK|OLD>

M,<age>,<clean|not clean>

S,<0|1>

CUP,<0|1>,<ONLINE|OFFLINE>

LOG STATS
...
```
### App Password

Some functions of the Android application are password-protected to prevent accidental changes.

The password is the **current day and month** in `DDMM` format.

**Example:** October 8 → `0810`

The password changes automatically every day.
