# Conventions in Part Docs

## Tables
All part specific documentation contains a table like this. See also the table in the readme comparing the part families. If a specification does not change between different parts within a family, and would not be expected to, and it also does not depend on the bootloader used, then it will be listed on that table, not here.


Specification         |       ATtiny88 |       ATtiny88 |       ATtiny88 |      ATtiny48  |       ATtiny48 |
----------------------|----------------|----------------|----------------|----------------|----------------|
Bootloader            |                |       Optiboot |   Micronucleus |                |       Optiboot |
Uploading uses        |   ISP/SPI pins | Serial Adapter | USB (directly) |   ISP/SPI pins | Serial Adapter |
Flash available       |     8192 bytes |     8192 bytes |     6550 bytes |     4096 bytes |     3456 bytes |
RAM                   |      512 bytes |      512 bytes |      512 bytes |      256 bytes |      256 bytes |
EEPROM                |       64 bytes |       64 bytes |       64 bytes |       64 bytes |       64 bytes |
GPIO Pins             |     26 + RESET |     26 + RESET |     25 + RESET |     26 + RESET |     26 + RESET |
ADC Channels          |   8 (6 in DIP) |   8 (6 in DIP) |              8 |   8 (6 in DIP) |   8 (6 in DIP) |
PWM Channels          |      2 (9, 10) |      2 (9, 10) |      2 (9, 10) |      2 (9, 10) |      2 (9, 10) |
Interfaces            |       SPI, I2C |       SPI, I2C | vUSB, SPI, I2C |       SPI, I2C |       SPI, I2C |
Clocking Options      |         in MHz |         in MHz |         in MHz |         in MHz |         in MHz |
Int. Oscillator       |     8, 4, 2, 1 |     8, 4, 2, 1 |  Not supported |     8, 4, 2, 1 |     8, 4, 2, 1 |
WDT Oscillator        |        128 kHz |  Not supported |  Not supported |        128 kHz |  Not supported |
Internal, with tuning |    8, 12, 12.8 |    8, 12, 12.8 |  Not supported |    8, 12, 12.8 |    8, 12, 12.8 |
External Crystal      |  Not supported |  Not supported |  Not supported |  Not supported |  Not supported |
External Clock        |   All Standard |   All Standard | **16**,8,4,2,1 |   All Standard |   All Standard |

* "Uploading Uses" lists the protocol used to upload sketches. All parts supported by ATTinyCore can ONLY have fuses written using ISP programming. Note that while most tinies *can* use a self-programming sketch like the Micronucleus Updater which erases the bootloader and puts in a different one, this has only been done for Micronucleus (this was done to address the need to update the bootloader on the original Digispark board, which shipped with RSTDSBL set, though it is widely used for all Micronucleus boards now) That is the only time this core provides for updating the bootloader without using an ISP programmer.
* Flash Available - usable flash for user code after the overhead of the bootloader. This assumes that the newest bootloader shipped with ATTinyCore is used. Micronucleus boards should have bootloader upgraded, as the stock bootloaders are often much worse.
* Interfaces does not count software/bitbanged interfaces like software serial. All parts that do not have hardware serial have an internal software serial port as described elsewhere in the documentation.
* The "standard" internal oscillator speeds are 8, 4, 2, or 1 MHz. Additionally there is often an option derived from the watchdog timer (on the x41, several) - if you thought the calibration on the main oscillator was lousy, you ain't seen nothing. It is not even LISTED on the datasheet, for most parts. On a few, it is spec'ed at +/- 30%. You use it to save power when you don't really care what exactly the frequency is. It has a stronger dependence on voltage and temperature compared to the high speed internal oscillator, and little if any calibration is available for it (the parts with a ULP allow you to choose between 4 "calibrations").
* The "standard" crystal frequencies are 20, 16, 12, 8 and 6 MHz. Parts are supported up to their maximum rated speed + 4 MHz  With 2.0.0 we have dropped support for external 1 and 4 MHz crystals - I have not once seem anyone mention using such a configuration, nor have I encountered a crystal with either of those speeds while examining surplus consumer electronics for design insights. Prescaling a higher frequency crystal to get the desired system clock is internally supported, but this is only exposed on the ATtiny167 Micronucleus (Digispark Pro) configuration, which uses a 16 MHz crystal for USB.
* The "extended" crystal frequencies include the standard ones plus the USART speeds.
* External clock is supported on all parts as of 2.0.0, at all standard speeds that the part supports, plus any options within 4 MHz of the maximum rated speed. Where one speed is in bold, that is the only external clock supported, but the others listed can be generated throuogh prescaling (only found on the ATtiny88)
* "With Tuning"  There is an option to upload a tuner when bootloading with optiboot, or "bootloading" to set fuses and clear flash in a no-bootloader configuration. If using ArduinoAsISP++ (included with this core) this is done automatically if it detects the tuner sketch on the target. (wait until the light on the programmer stops blinking). Micronucleus using the internal oscillator uses a different method that retunes it from the USB clock every time it is started on USB; this "boot tuning" is by default carried over to the sketch, but the core may be set to switch back to factory cal - or even the results of the tuning sketch, if either greater accuracy or a different tuned speed is required.
* Why 12.8?
  * It is a Micronucleus magic speed - though the binary size for this is very large. TBD whether it will have any use in the micronucleus board defs.
  * It naturally gives exact micros/millis timing (being 1/5th of 64) so you don't pay a "strange clock tax" in binary size 3.
  * Just about every chip can get to 12.8 by adjusting OSCCAL, it's decent for hardware UART too (better matchup with 115200 vs 16 MHz, and can do 230400), and thee's nothing better between 8 and the 14-15 that most parts can hit with OSCCAL maxed.

## Tools submenus
This core, being designed to handle almost every configuration, has a large number of tools submenus

### Tools -> Board
This control two things: Which microcontroller or family will be used (a "family" or "series" is 2 or more microcontrollers that differ only in memory sizes), and what bootloader, if any, will used to bootload and be assumed assumed when uploading. Possibilities here are no bootloader, "Optiboot", or "Micronucleus" "Urboot" may be added, in which case Optiboot would become deprecated, as it has one very serious bug. Parts with at least 4k of flash and which are available in a version with at least 8k of flash have an Optiboot bootloader available. Micronucleus bootloaders are provided for boards with atleast 8k of flash, with the exception of the 828 which is such a sorry piece of hardware it is not worth the time to make it wiork. If you do not know if a chip has been bootloaded, or know that it hasn't, you should select the definition with the bootloader you *want*, select the options you want, and connect an ISP programmer, and choose Tools -> Burn Bootloader.

### Tools -> Chip
Very simple. When a family (as opposed to single chip) is selected, this is where you choose which one you are using.

### Clock Source and Speed
Often a VERY long menu, this lists all supported combinations of clock source and clock speed for the system clock. The options are listed in descending order of popularity/usefulness. Internmal and PLL speeds (if present) are at the top. Then come Crystal speeds, starting with common ones, then USART-crystals (the speeds that provide perfect UART baud rates, at the expence of making timekeeping slower and worse). After that are the rarely used "tuned" bootloader options that allow a chip which has been "tuned" by running the tuning sketch. These speeds are generally 8 MHz (user calibration at operating temperature and voltage beats factory cal significantly), 12.0 MHz, and on most parts, 12.8 (12.8 is mathematically favorable since 64 divides evenly into it, vastly simplifying the math. All of these speeds can almost always be reached with the internal oscillator and tuning. On the ATtiny841 and 441, the oscillator is much fancier, and an additional 16 MHz tuning is attempted. Most chips can do it (as long as they're running at around 5v) a few have unusually slow oscillators and cannot. Finally, we end on the external CLOCK options. An external clock is a component that looks near identical to a square, golden crystal package most of the time, but the pins are typically Out and Enable in place of the two crystal pins (if enable control is ot needed, enable should be tied high, low, or allowed to float depeding on the device, refer to the datasheet). The other two opposite corners are Vdd and Gnd. It is imperitve that these never be powered baclwardsl they will burn out aslmost instantly. External clocks also cost a lot more money than crystals, and the "china discount" for buying parts direct from china is much smaller than with crystals".
