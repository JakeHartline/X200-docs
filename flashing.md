![1](/pictures/2.jpg)
# Overview on external flashing
This breifly describes flashing process bios chip on Thinkpad X200 directly (not from a running OS).

## Prepare the programmer
There are a few options when choosing a programmer. Those listed on libreboot.org use SoC of sorts, which is unnecessary. There are 2 better options:

### CH341A
TL;DR: it uses 5v on signal lanes, while the bios flash chips are 3.3v devices. Not great.\
[Thread on EEVblog forum.](https://www.eevblog.com/forum/repair/ch341a-serial-memory-programmer-power-supply-fix/2) \
[Issue on libreboot bugtracker.](https://notabug.org/libreboot/libreboot/issues/637)

### Bluepill
This is a generic dev board with stm32 microcontroller. It needs to be flashed with [vserprog firmware](https://github.com/dword1511/stm32-vserprog) to be used as a programmer.\
If you're going to buy one just for flashing, get one with pin headers already soldered.

#### Building and flashing vserprog
Get following packages. On arch based system:
```
pacman -S arm-none-eabi-gcc arm-none-eabi-newlib
```

Download and build the firmware
```
git clone --recurse-submodules https://github.com/dword1511/stm32-vserprog.git
cd stm32-vserprog
make BOARD=stm32-bluepill
```
Flash it using ST-link (ST-link is a programmer for stm devices) \
```
make BOARD=stm32-bluepill flash-stlink
```

## The clip
I designed and used a 3d printed jig that would align and press jumper cables against chip pins. \
It could really use a rework. [More here.](/jig/README.md)

### Wiring
[SOIC-16 legs](https://i.imgur.com/z2kbRml.png)

| chip    | bluepill |
| ------- | ------   |
| VCC     | 3.3      |
| GND     | GND      |
| CS      | A4       |
| CLK     | A5       |
| MISO    | A6       |
| MOSI    | A7       |

## Flashing
### Probe for chips
Get flashrom
```
pacman -S flashrom
```
Probe for chips
```
flashrom -p serprog:dev=/dev/ttyACM0:4000000
```

You'll get a bunch of lines like
```
serprog: requested mapping AT45CS1282 is incompatible: 0x1080000 bytes at 0xfef80000.
```
However, what you're looking for are lines similar to
```
Found Macronix flash chip "MX25L6405" (8192 kB, SPI) on serprog.
```
Remember the size. \
\
If no chips are found - something is wrong, double check your connections.\
In case flashrom finds multiple chips, `-c "your_chip_model"` option will be needed. Any found chip can be specified, they are from the same family and work the same (is this correct?). Otherwise don't use `-c` option at all.
[More details on programmer usage.](https://github.com/dword1511/stm32-vserprog#usage)

### Backup default bios (optional)
If you ever need to, there is no generic lenovo bios you can roll back to. You can only roll back to a backup of your original bios.\
You can skip this if you don't care about having default bios.
```
flashrom -p serprog:dev=/dev/ttyACM0:4000000 -c MX25L6405 -r x200bios.bin
```
Really you should read a couple of times and compare the output to make sure you are getting consistent results (diff output should be empty):
```
flashrom -p serprog:dev=/dev/ttyACM0:4000000 -c MX25L6405 -r x200bios2.bin
diff x200bios.bin x200bios2.bin
```
Additionally check if the output file is not filled with zeros.

### Download bios rom.
Official builds from libreboot.org are very old. I had better experience with builds from [this repo.](https://github.com/JaGoLi/Libreboot-X200-Updated) \
Download appropriate X200 rom for your chip size from [here.](https://github.com/JaGoLi/Libreboot-X200-Updated/tree/main/roms) \
Names ending with *microcode* contain microcode update blob, *free* do not.

### Write libreboot rom
Flashing, at last
```
flashrom -p serprog:dev=/dev/ttyACM0:4000000,spispeed=512 -c MX25L6405 -w x200_8mb_free.rom
```
At spi speed 512 it will take a while.

When you see
```
Verifying flash... VERIFIED.
```
Congrats, you're done.

### About spispeed=512
Writing to x200 is supposedly inconsistent. There is a [patch](https://notabug.org/libreboot/libreboot/src/master/projects/flashrom/patches/0002-Workaround-for-MX25-chips.patch) for flashrom to fix that.\
However, with unmodified flashrom from my disto repos, I was able to read the default bios just fine. Writing worked with 1024 (possibly will work with higher speed, I didn't test).\
Check flashrom manpage for more info on spispeed.\
[Reddit post on the issue.](https://old.reddit.com/r/libreboot/comments/icvr6t/attempting_to_flash_x200_erase_functions_fail/)

## Additional resourses

[External flashing, libreboot.org](https://libreboot.org/docs/install/x200_external.html)\
[Coreboot wiki entry for x200](https://www.coreboot.org/Board:lenovo/x200)\
[Wolfgang's video on flashing x200](https://www.youtube.com/watch?v=ktcvWkEVBE0)
