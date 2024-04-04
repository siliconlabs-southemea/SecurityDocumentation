---
sort: 4
---
# Uploading CPC Application GBL to RCP device

Start by uploading the GBL file generated previously on your CPCd host

From there, steps to follow are the ones from [the cpc documentation](https://github.com/SiliconLabs/cpc-daemon/blob/main/doc/firmware_upgrade.md#cpc-activation).

***NOTE*** : At the time of writing (GSDK 4.4.1), in case GPIO activation is not available, it is possible that the `cpcd` does not detect that UART-XMODEM is in bootload mode

Assuming the gbl file is named `ot-rcp-cpc.gbl` and is located in `~` , start the firmware upgrade with :

```bash
cpcd --firmware-update ot-rcp-cpc.gbl
```

Output should be :

```bash
[2024-04-04T15:05:21.449378Z] Info : Starting daemon in firmware update mode
[2024-04-04T15:05:21.449388Z] Info : Requesting versions via CPC...
[2024-04-04T15:05:21.461630Z] Info : Connecting to Secondary...
[2024-04-04T15:05:21.759370Z] Info : RX capability is 256 bytes
[2024-04-04T15:05:21.759398Z] Info : Connected to Secondary
[2024-04-04T15:05:21.763233Z] Info : Secondary Protocol v5
[2024-04-04T15:05:21.772128Z] Info : Secondary bootloader: Gecko SDK
[2024-04-04T15:05:21.776889Z] Info : Secondary CPC v4.4.1
[2024-04-04T15:05:21.784981Z] Info : Secondary bus bitrate is 115200
[2024-04-04T15:05:21.793584Z] Info : Secondary APP vUNDEFINED
[2024-04-04T15:05:21.794105Z] Info : Server core cleanup
[2024-04-04T15:05:21.795257Z] Info : Firmware file version not provided, forcing update
[2024-04-04T15:05:21.795274Z] Info : Requesting reboot into bootloader via CPC...
[2024-04-04T15:05:21.814358Z] Info : Secondary is in bootloader
[2024-04-04T15:05:21.814754Z] Info : Server core cleanup
[2024-04-04T15:05:21.815073Z] Info : Transferring firmware...
[2024-04-04T15:05:39.346075Z] Info : Firmware upgrade successful
[2024-04-04T15:05:39.346110Z] Info : Exiting CPCd...
```

The case where the bootloader UART settings do not match with the RCP settings is covered in the guide and proposes as an alternative :

```bash
cpcd -c application.conf --enter-bootloader
```

Resulting output should end by :

```bash
[2024-04-04T13:47:08.921732Z] Info : Starting daemon in firmware update mode
[2024-04-04T13:47:08.921742Z] Info : Performing a firmware upgrade while assuming the bootloader is already running, skipping placing the secondary in bootloader mode
[2024-04-04T13:47:08.921746Z] Info : Transferring firmware...
[2024-04-04T13:47:27.453991Z] Info : Firmware upgrade successful
[2024-04-04T13:47:27.454026Z] Info : Exiting CPCd...
```

Assuming the gbl file is named `ot-rcp-cpc.gbl` and is located in `~` , start the file transfer with :

```bash
cpcd -c  --connect-to-bootloader --firmware-update ot-rcp-cpc.gbl
```

The output should look like : 

```bash
[2024-04-04T13:47:08.921732Z] Info : Starting daemon in firmware update mode
[2024-04-04T13:47:08.921742Z] Info : Performing a firmware upgrade while assuming the bootloader is already running, skipping placing the secondary in bootloader mode
[2024-04-04T13:47:08.921746Z] Info : Transferring firmware...
[2024-04-04T13:47:27.453991Z] Info : Firmware upgrade successful
[2024-04-04T13:47:27.454026Z] Info : Exiting CPCd...
```

In case the CPC secondary is stuck in bootloding mode, the only way to get out of that state is to perform an upgrade by calling the last command once again
