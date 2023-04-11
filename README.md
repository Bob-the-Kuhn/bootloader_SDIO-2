If you get the following error then you need to install the 64bit version of GCC or mingw.  Turns out you're missing the libgcc library but the 32 bit version is only available in the 64 bit package.
    arm-none-eabi/bin/ld.exe: cannot find -lgcc: No such file or directory


## STM32F407VE bootloader using SDIO interface

Functionality:

    - Copies image from the file firmware.bin to FLASH
    - Loading starts at 0x0800 9000 (can be changed)
    - Write protection is automatically removed
    - File/image checksum/crc checking is not done
    - UART1 @115200 can be used to monitor the boot process.
    - FAT32 file system with 512 - 4096 byte AUs.

Load point is set by APP_ADDRESS in bootloader.h.

Image filename is set by CONF_FILENAME in main.h.
Target card was a STM32_F4VE.

Pinout is at:  https://stm32-base.org/boards/STM32F407VET6-STM32-F4VE-V2.0.html

This code is based on the booloader and main routines from:
  https://akospasztor.github.io/stm32-bootloader

Steps to create project:
1) copy already existing project to the new folder
2) delete the Core, FATFS and Middlewares folders
3) create base in STM32cubeIDE & generate code
4) copy Core, FATFS and Middlewares over to new folder
5) update platformio.ini and build.bat
6) update main.h and main.c (compare already existing project vs. new files & copy items as needed)
7) update bootloader files for the new CPU



## Hardware notes:

SDIO is used in low speed polling mode.  SDIO DMA mode worked for cards with
512 byte allocation units but not with 4096 byte allocation units.

This board does NOT have a hardware "SD card is present" pin so some
code was commented out.

Erasing is done via sectors. All sectors that don't have the boot loader image in it are erased. 

This program comes in at about 22,500 bytes which means it fills up sector 0 and extends in to
sector 1.  That means the lowest load point is the beginning of sector 2 (0x0800 8000). 

APP_ADDRESS can be set to any 512 byte aligned address in any erased sector.

## Building the image:

I used platformio within VSCode.  You'll need to set the workspace to the top
directory.  In the terminal window set the directory to the same one as the workspace.
Use the command "platformio run" command to build the image.  


## Porting to another processor:

Porting to the STM32F407ZG is very easy.  The only difference is the
larger FLASH.  You'll need to modify the FLASH defines to add the larger
number of sectors. 

Porting to other processors requires looking at the sector layout, erase
mechanisms and FLASH programming mechanisms. 