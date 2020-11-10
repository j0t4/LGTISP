# LGTISP
LGT8Fx8p ISP protocol implementation.

## introduction
This is an implementation of LGT8Fx8p ISP protocol. 
You can turn a LGT8Fx8p or an ATmega328p Arduino board into an ISP for LGT8Fx8P.

This https://github.com/SuperUserNameMan/LGTISP version supports the `dump flash`, `dump eeprom`, `dump lock` and `write lock 0 0` commands of AVRdude in terminal mode.

## warning :
- Once a newly programmed LGT8Fx is powerred-down, the access to its flash memory is locked. `dump flash` will displays `0xFF` everywhere.
- Unlocking is destructive. The first 1KB of flash will be erased, but the rest of the flash (including the emulated EEPROM) will be readable. When using AVRdude as terminal, locked devices can be unlocked manually using a `write lock 0 0`.
- for more details, see note regarding LGT8F328p flash protection, below.

## supported functions
- Arduino IDE and AVRdude commande line :
   - [x] upload/restore LGT8Fx8p bootloaders from the Arduino IDE
   - [x] upload sketches to LGT8Fx8p board from the Arduino IDE (Ctrl + Shift + U)
- AVRdude terminal mode :
   - [x] `dump flash` command
   - [x] `dump eeprom` command
   - [X] `dump lock` command : `0x3E` means `locked`, `0x3F` means `unlocked`.
   - [X] `write lock 0 0` command : to trigger a destructive unlock (the first 1KB of the flash will be erased).
   - [ ] `write XXX` command
   - [ ] `erase` command
   - [x] `sig` command
   - [x] `part` command

## usage

1. **Turn an other Nano board (ATmega328p or LGT8F328p) into an LGTISP :**
   1. If you want to use a real ATmega328p or ATmega168 Nano board as ISP :
       * select "Tools / Board type / Arduino AVR board / Arduino Nano".
       * select the appropriate "Tools / Processor". 
       * into your Arduino IDE installation directory, edit `hardware/arduino/avr/cores/arduino/HardwareSerial.h`.
       * at the beginning of the file, temporarily add `#define SERIAL_RX_BUFFER_SIZE 256` and save. 
   2. If you want to use a LGT8F328p Nano-like board as ISP :
       * select "Tools / Board type / Logic Green Arduino AVR compatible board / LGT8F328".
       * select the appropriate "Tools / Processor".
       * select "Tools / Arduino as ISP / \[To burn an ISP\] SERIAL_RX_BUFFER_SIZE 250".
   2. Upload this sketch into your Nano board.  
      * If the board is real ATmega328p Nano board, you can now remove the `#define SERIAL_RX_BUFFER_SIZE 256` from `HardwareSerial.h`. 
   3. If you want to avoid bootloader from executing (so you don't reprogram the ISP board accidentally) short `RESET` pin and `VCC` pin of board.
   4. The board has now become a LGTISP ! 
   5. Connect your Nano to the target :   
      | Nano || LGT ||
      |:-:|:-:|:-:|:-:|
      | D13 | -> | SWC ||
      | D12 | -> | SWD ||
      | D10 | -> | RST | (optional)|
      
2. **To burn a bootloader into a LGT8Fx8p board :**
   1. select "Tools / Board type / Logic Green Arduino AVR compatible board / LGT8F328".
   2. select the appropriate "Tools / Processor type".
   3. select "Tools / Programmer / AVR ISP".
   4. click "Tools / Burn Bootloader".
   
3. **To upload a sketch into a LGT8Fx8p board using your ISP :**
   1. select "Tools / Board type / Logic Green Arduino AVR compatible board / LGT8F328".
   2. select the appropriate "Tools / Processor type".
   3. select "Tools / Programmer / AVR ISP".
   4. click "Sketch / Upload using a programmer" or press `Ctrl + Shift + U`.
   
4. **To display the content of the flash using AVRdude :**
   1. open a sketch in the Arduino IDE, and click "Files / Preferences".
   2. check the "Display detailed results while [X] uploading", and press OK.
   3. upload this sketch using the LGTISP.
   4. into the console, look for the white line containing the avrdude commant line, and copy paste it into text editor. It should looks like something like this:
   `<path>/arduino-1.8.13/hardware/tools/avr/bin/avrdude -C<path>/arduino-1.8.13/hardware/tools/avr/etc/avrdude.conf -v -patmega328p -cstk500v1 -P<isp_serial_adapter> -Uflash:w:/tmp/arduino_build_379084/<sketchname>.ino.hex:i`
   5. change it so it looks like this:
   `<path>/arduino-1.8.13/hardware/tools/avr/bin/avrdude -C<path>/arduino-1.8.13/hardware/tools/avr/etc/avrdude.conf -v -patmega328p -cstk500v1 -P<isp_serial_adapter> -t`
   6. `-t` will open AVRdude in terminal mode.
   7. open a terminal, and run this command. AVRdude should open in terminal mode.
   7. if the flash memory is unlocked, command `dump flash <addr> <length>` will let you display the content of the flash, and command `dump eeprom 0 1024` will display the content of the EEPROM.
   8. commant `quit` to quit, and `help` for help.
   
## Note regarding LGT8F328p flash protection :

When a LGT8F328p chip is powered down just after it was programmed, its Flash memory access is locked from the ISP : it becomes impossible to `dump flash` and to reprogram it using the ISP.

The only known way to unlock a device was to erase it completly, which was not a problem at all when using the ISP to upload new sketch from the Arduino IDE.
On the other hand, inspecting the content of the flash using AVRdude terminal was impossible on locked devices, since they had to be erased to be unlocked ...

Later, brother-yan/LGTISP found that the chip could be unlocked by only erasing the first 1KB of flash, which made possible to inspect the rest of a locked device using AVRdude terminal.

To know if a device is locked you can use the `dump lock` command : `0x3E` means the chip is locked. `0x3F` means it is unlocked.

You can force the destructive unlock with a `write lock 0 0` command : only the first 1KB of the flash will be erased to `0xFF`. All the rest of the flash memory will remain intact, including the EEPROM, for inspection. However, once done, the chip will be bricked (the first 1KB contained the Interrupts Vectors table as well as the beginning of the sketch), so you'll have to reprogram it using the ISP.
   
The only known way to inspect the content of the Flash in a non destructive manner is to keep the device powered after it was programmed.
   
## Note regarding the EEPROM on LGT8F328p :

On the LGT8F328p, the EEPROM is emulated and stored into the main Flash memory.

The size of this emulated EEPROM can be set by software using bits 1 and 0 of the `ECCR` register :

| bit 1 | bit 0 | EEPROM size | Available bytes | Size in flash memory |
|:-:|:-:|:-:|:-:|:-:|
| 0 | 0 | 1KB | 1024 - 2 | 2048 | 
| 0 | 1 | 2KB | 2048 - 4 | 4096 |
| 1 | 0 | 4KB | 4096 - 8 | 8192 |
| 1 | 1 | 8KB | 8192 - 16 | 16384 |

However, because AVRdude thinks the LGT8F328p is an ATmega328p, `dump eeprom` will only display 1KB.

Also, because EEPROM is emulated using the main Flash memory, it can be dumped using `dump flash 0x7800 1024` and `dump flash 0x7c00 1024`.

There are two pages, because the hardware algorithm swap them each time the content of the emulated EEPROM is updated.

The hardware algorithm does like this : each time the EEPROM is updated, the EPCTL erases the new page, copies the old page + the updates into the new page, swaps the old and the new pages, and adds a 2bytes long code at the end of each page to remember which one is new, and which one is old.

This means that 1KB of EEPROM will use 2KB of flash, 2KB will use 4KB, etc, and also that the last 2 bytes of every 1024 bytes pages will contain a code.

So on 1024 bytes, only 1022 are actually available, and the 2 lasts ones will be overwritten by the LGT8F328p.


## reference
- Forked from : [brother-yan/LGTISP](https://github.com/brother-yan/LGTISP)
- recommanded LGT8F core for Arduino IDE : [dbuezas/lgt8fx](https://github.com/dbuezas/lgt8fx)
- Atk500v1 protocol : http://ww1.microchip.com/downloads/en/Appnotes/doc2525.pdf 
- AVRdude : http://savannah.nongnu.org/projects/avrdude
- AVR ISP protocol : [AVR910 - In-System-Programming protocol](http://ww1.microchip.com/downloads/en/Appnotes/Atmel-0943-In-System-Programming_ApplicationNote_AVR910.pdf)
- Related discussion : https://github.com/dbuezas/lgt8fx-forum/issues/1
