USB2Sun
=======
While I've found many projects for connecting the Sun serial based keyboards to a modern
machine via USB, I've never found anything to connect a USB keyboard onto a classical
Sun workstation.

This project allows you to connect a USB keyboard and mouse to a Sun workstation, using a Raspberry Pico as a bridge.

I've used two SN74LVC244AN as level shifters, one for 3.3V to 5V (from Pico to Sun),
and one for 5V to 3.3V (from Sun to Pico). There are probably smaller shifters than
the octal shifter I found, but I couldn't find any.

The Pico board is powered by the Sun workstation's 5V supply.

I've used Pico SDK v1.3 when developing, but note that the supplied TinyUSB version may have problems with the Pico. I personally did not get any USB devices recognized until after [ab760290a5](https://github.com/hathach/tinyusb/commit/ab760290a514bc239c5ef8901ff8d80f64d461f9). Connecting via a USB hub required [1915d69cb8](https://github.com/hathach/tinyusb/commit/1915d69cb8da8372b35a5067420c57ddb65d2cb1). Until a future Pico SDK release with updated TinyUSB stack, I recommend entering pico-sdk/lib/tinyusb and typing
```
git fetch
git checkout 1915d69cb8da8372b35a5067420c57ddb65d2cb1
```


Features
--------
* Emulate the left keyboard function keys by entering LeftShift+LeftControl and the corresponding key

  | PC  | Function   |
  |-----|------------|
  | F1  | Stop       |
  | F2  | Again      |
  | 1   | Props      |
  | 2   | Undo       |
  | Q   | Front      |
  | W   | Copy       |
  | A   | Open       |
  | S   | Paste      |
  | Z   | Find       |
  | X   | Cut        |

  Breaking into OpenFirmware would be LeftShift+LeftControl+F1+A

Missing Features
----------------
* Keyboard power on. Shorting minidin pin 7 to 5V rail powers on the system, however, the 5V rail is unpowered when the system is off - so the Pico is also unpowered in this state.
* LED indicators (num - caps and scroll lock).
* Switching layouts. PC or Unix layout and international layout (the emulated keyboard responds US for now).

Build
-----
Make sure the Pico SDK has been checked out, as detailed by the Pi foundation.

    mkdir build && cd build
    PICO_SDK_PATH=../../pico-sdk cmake ..
    make

If your flashing with a Raspberry Pi, flash the firmware using

    openocd -f interface/raspberrypi-swd.cfg -f target/rp2040.cfg -c "program usb2sun.elf verify reset exit"

Or copy usb2sun.uf2 onto the Pico when it's in flash mode.

Parts required
--------------
* Raspberry Pico
* 8 pin minidin male cable (I used
https://no.rs-online.com/web/p/din-cable-assemblies/0463518)
* 2 SN74LVC244AN

Wiring
------
Raspberry Pico

     UART0 TX   * Pin  1-----|USB|------Pin 40 *   5V
                * Pin  2     `‾‾‾´      Pin 39 * 
                * Pin  3                Pin 38 *   GND
                * Pin  4                Pin 37 * 
                * Pin  5                Pin 36 *   3V3
     UART1 TX   * Pin  6                Pin 35 * 
     UART1 RX   * Pin  7                Pin 34 * 
                * Pin  8                Pin 33 * 
                * Pin  9   .--------.   Pin 32 * 
                * Pin 10   |        |   Pin 31 * 
                * Pin 11   | RP2040 |   Pin 30 * 
                * Pin 12   `--------´   Pin 29 * 
                * Pin 13                Pin 28 * 
                * Pin 14                Pin 27 * 
                * Pin 15                Pin 26 * 
                * Pin 16                Pin 25 * 
                * Pin 17       π        Pin 24 * 
                * Pin 18                Pin 23 * 
                * Pin 19                Pin 22 * 
                * Pin 20----------------Pin 21 * 

5V to 3V3 level shifter

          GND   * Pin  1----------------Pin 20 *   3V3
       KBD RX   * Pin  2                Pin 19 *
                * Pin  3                Pin 18 *   UART1 RX
                * Pin  4                Pin 17 *
                * Pin  5  SN74LVC244AN  Pin 16 *
                * Pin  6                Pin 15 *
                * Pin  7                Pin 14 *
                * Pin  8                Pin 13 *
                * Pin  9                Pin 12 *
          GND   * Pin 10----------------Pin 11 *

3V3 to 5V level shifter

          GND   * Pin  1----------------Pin 20 *   5V
     UART1 TX   * Pin  2                Pin 19 *
                * Pin  3                Pin 18 *   KBD TX
     UART0 TX   * Pin  4                Pin 17 *
                * Pin  5  SN74LVC244AN  Pin 16 *   MOUSE TX
                * Pin  6                Pin 15 *
                * Pin  7                Pin 14 *
                * Pin  8                Pin 13 *
                * Pin  9                Pin 12 *
          GND   * Pin 10----------------Pin 11 *

MiniDin-8 plug

            , - ~._.~ - ,
        , '               ' ,
      ,     6     7     8     ,
     ,                         ,
    ,                           ,
    ,    3    4           5     ,
    ,                           ,
     ,                          ,
      ,_      1      2       _,
        `-.              .-´
          ' - , _ _ _ , -'

| Pin | Function   |
|-----|------------|
| 1   | GND        |
| 2   | GND        |
| 3   | 5V         |
| 4   | MOUSE TX   |
| 5   | KBD RX     |
| 6   | KBD TX     |
| 8   | 5V         |
