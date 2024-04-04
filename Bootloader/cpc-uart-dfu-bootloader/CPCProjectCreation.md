---
sort: 3
---
# CPC Application Project and GBL File Creation

## CPC Application creation

### OpenThread RCP + Zigbee NCP (Dual Protocol) project creation

Refer to OpenThread Border Router CPC for the Concurrent Multi Protocol Project Creation

### RAILTEST over CPC

Refer to Proprietary RAILTEST CPC Interface to know how to add support of CPC to RAILTEST. Additionally, bootloader interface may be required to be added as it is not there by default

### OpenThread RCP (Single Protocol) project creation

Refer to OpenThread RCP over CPC for the RCP Project Creation

### TODO1 : Create Bluetooth HCI over CPC

### TODO2 : Create Zigbee NCP over CPC

## Generating application GBL for Upgrading application

First you need to build one of the above mentioned project to get a working binary (.hex or .s37)

By default the application uses version one, but can be redefined using `SL_APPLICATION_VERSION`

Once done, we will be using Simplicity commander to generate our GBL :

```bash
.\commander gbl create .\ot-rcp-cpc.gbl --app .\ot-rcp-cpc.s37
```

In case GBL signature is required :

```bash
.\commander gbl create .\ot-rcp-cpc.gbl --app .\ot-rcp-cpc.s37 --sign .\sign_key.pem
```
