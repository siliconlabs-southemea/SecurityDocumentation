---
sort: 2
---
# Bootloader Project Creation

1. Plug-in your radio board + dev kit to your computer
2. Open `Simplicity Studio > File > New > Silicon Labs Project Wizard`
3. If your radio board was plugged in and GSDK installed, you should have all fields pre-filled.

   Otherwise just customize the project base settings accordsing to your needs

   ![1710845538151](image/BootloaderProjectCreationAndConf/1710845538151.png)

   In this case we will be using a `BRD4186C` radio board
4. On the subsequent screen select `Bootloader - NCP UART XMODEM`

   This will create a Geckpo Bootloader project with upgrade over UART support

   It will bring in the minimum required to get started with a radio board

   ![1710845600454](image/BootloaderProjectCreationAndConf/1710845600454.png)

   Click Next
5. Rename your project as you whish

   ![1710845692297](image/BootloaderProjectCreationAndConf/1710845692297.png)

   If you wish to, you can also set your project to copy all sources from the Gecko SDK locally upon configuration

   This allows for easier versioning but complexifies import/export
6. Click Finish to proceed with project creation
