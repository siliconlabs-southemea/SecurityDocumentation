---
sort: 3
---
# RTSL Enablement and Bootloader Signature

> WARNING
> Even though below steps are separated, it is important to note that SECURE_BOOT and/or SECURE_BOOT_PAGE_LOCK are set together
>
> The Security config can be provisioned only ONCE per device

## RTSL Enablement and Bootloader Page Lock (optional)

This will mostly be a reuse of [AN1218](https://www.silabs.com/documents/public/application-notes/an1218-secure-boot-with-rtsl.pdf) section 3, using Simplcity Commander

### Provision OTP with signing key

**NOTE** : Always provide --serialno and the right --device OPN

1. Generate your key pair in PEM format (we will use commander but other options are available)

   ```bash
   commander util genkey --type ecc-p256 --privkey sign_key.pem --pubkey sign_pubkey.pem
   ```

2. Flash the public key into the EFR32 MG24's OTP dedicated slot

   ```bash
   commander security writekey --sign sign_pubkey.pem --device EFR32MG24B210F1536 --serialno 440048205
   ```

3. Read key back :

   ```bash
   commander security readkey --sign --device EFR32MG24B210F1536 --serialno 440048205
   ```

### Provision OTP with security configuration (No Page Lock, anti rollback enabled)

1. Generate the security configuration JSON Structure, make sure it is done for the right OPN

   ```bash
   commander security genconfig --nostore --outfile user_configuration.json --device EFR32MG24B210F1536 --serialno 440263650
   ```

2. Edit the security configuration according to below

   ```json
   "mcu_flags": {
        "SECURE_BOOT_ANTI_ROLLBACK": true,
        "SECURE_BOOT_ENABLE": true,
        "SECURE_BOOT_PAGE_LOCK_FULL": false,
        "SECURE_BOOT_PAGE_LOCK_NARROW": false,
        "SECURE_BOOT_VERIFY_CERTIFICATE": false
    },
   ```

3. Provision the device with the generated configuration

   ```bash
   commander security writeconfig --configfile user_configuration.json --device EFR32MG24B210F1536 --serialno 440263650
   ```

4. Verify the current OTP configuration

   ```bash
   commander security readconfig --device EFR32MG24B210F1536 --serialno 440263650
   ```

5. Reset the device

   ```bash
   commander device reset --device EFR32MG24B210F1536 --serialno 440263650
   ```

### Provision OTP with security configuration (Flash Page Lock enabled)

It is possible to lock secure booted flash pages via the Secure Engine

This is covered by the OTP flags `SECURE_BOOT_PAGE_LOCK_FULL` and `SECURE_BOOT_PAGE_LOCK_NARROW`

The key difference between those is that FULL will protect all the secure booted application (here bootloader), INCLUDING the space where App Properties is

NARROW will avoid locking the page where the app properties is, allowing for certificate use and revocation.

1. To enable page lock simply change the OTP configuration :

   ```json
      "mcu_flags": {
         "SECURE_BOOT_ANTI_ROLLBACK": true,
         "SECURE_BOOT_ENABLE": true,
         "SECURE_BOOT_PAGE_LOCK_FULL": true,
         "SECURE_BOOT_PAGE_LOCK_NARROW": false,
         "SECURE_BOOT_VERIFY_CERTIFICATE": false
      },
   ```

2. And provision this new config :

   ```bash
      commander security writeconfig --configfile user_configuration.json --device EFR32MG24B210F1536 --serialno 440263650
   ```

3. Verify the current OTP configuration

   ```bash
   commander security readconfig --device EFR32MG24B210F1536 --serialno 440263650
   ```

4. Reset the device

   ```bash
   commander device reset --device EFR32MG24B210F1536 --serialno 440263650
   ```

### Signing And flashing the bootloader

1. Start by signing the bootloader using the below command :

   ```bash
   commander convert bootloader-storage-internal-single-1536k-ROMed.s37 --secureboot --keyfile sign_key.pem --verify sign_pubkey.pem --outfile bootloader-storage-internal-single-1536k-ROMed_signed.s37
   ```

   **NOTE** : Once secure boot is enabled, if no valid image is present in the main flash, the Cortex-M33 will hang up on reset state (See AN1218)
   It is therefore more stable to unlock it prior to flashing a signed bootloader or image

2. Check the device status using :

   ```bash
   commander security status --device EFR32MG24B210F1536 --serialno 440263650
   ```

   The device should be in a failed boot state, with signing key present and secure boot enabled :

   ```bash
   Secure boot           : Enabled
   Boot status           : 0x14 - Failed: Error while finding host application properties pointer
   Sign key installed    : True
   ```

3. Unlock the device using :

   ```bash
   commander device unlock --device EFR32MG24B210F1536 --serialno 440263650
   ```

   This will trigger a mass erase.

4. Then flash your signed image using :

   ```bash
   commander flash bootloader-storage-internal-single-1536k-ROMed_signed.s37 --masserase --device EFR32MG24B210F1536 --serialno 440263650
   ```

5. Check the device status again :

   ```bash
   commander security status --device EFR32MG24B210F1536 --serialno 440263650
   ```

   The device should be in boot state OK, with signing key present and secure boot enabled :

   ```bash
   Secure boot           : Enabled
   Boot status           : 0x20 - OK
   Sign key installed    : True
   ```

## Permanently locking Bootloader flash space (Not recommended as prevents bootloader updates)

Once Page lock is enabled, Cortex-M33 becomes unable to write/erase the dedicated space to the bootloader

However, the Secure Engine still can edit these 2 pages of flash over either SWD or SE Mailbox

To completely block the state of these pages two other Secure Engine flags must be written as per configuration 2 below.

Remember this is the most risky setting if device is not provisioned with the right binaries in the first place.

This could result in bricking you chipset if no binary or bootloadeer with upgrade capability is flashed prior to locking

Available flags are :

* Debug Lock (Debug port kept locked on boot) – Defaults to false

* Device Erase (Device Erase command is availble after boot) – Defaults to True

* Secure Debug (Secure Debug Unlock is made available) – Defaults to False
   Allows for an external authenticated connnection via DCI (Debug Challenge) interface

Default state is DL = 0; DE = 1; SD = 0

Possible locking configurations are :

1. Standard Debug Lock
   Unlock possible via mass erase
   DL = 1 ; DE = 1 ; SD = 0

   ```bash
   commander security lock --device EFR32MG24B210F1536 --serialno 440263650
   ```

2. Permanent Debug Lock - ROMs already flashed bootloader, prevents from writing an app without DFU capable bootloader
   **Risk of brick, --disabledeviceerase should be the last step in production**
   DL = 1 ; DE = 0 ; SD = 0

   ```bash
   commander security lock --device EFR32MG24B210F1536 --serialno 440263650
   commander security --disabledeviceerase --device EFR32MG24B210F1536 --serialno 440263650
   ```

3. Secure Debug Lock
   Unlock possible, mass erase disabled
   **Risk of brick, --disabledeviceerase should be the last step in production**
   DL = 1 ; DE = 0 ; SD =1

   ```bash
   commander security lockconfig --secure-debug-unlock enable --device EFR32MG24B210F1536 --serialno 440263650
   commander security lock --device EFR32MG24B210F1536 --serialno 440263650 
   or
   commander security lock --trustzone 1100 --device EFR32MG24B210F1536 --serialno 440263650
   commander security disabledeviceerase --device EFR32MG24B210F1536 --serialno 440263650
   ```

### Secure Debug Lock / Unlock

Using Secure Debug Lock / Unlock it is however possible to securely regain access to the SWD interface

This allows for updating the bootloader from SWD

Below will mostly follow [AN1190](https://www.silabs.com/documents/public/application-notes/an1190-efr32-secure-debug.pdf) section 6.3.3

The first action to perfor is to generate a key pair for securing debug access

This is referred to as "Command Key"

1. Generate your key pair in PEM format (we will use commander but other options are available)

   ```bash
   commander util genkey --type ecc-p256 --privkey command_key.pem --pubkey command_pubkey.pem
   ```

2. Provision your command key to the OTP

   ```bash
   commander security writekey --command command_pubkey.pem --device EFR32MG24B210F1536 --serialno 440263650
   ```

3. Read back the command key

   ```bash
   commander security readkey --command --device EFR32MG24B210F1536 --serialno 440263650
   ```

4. Enable Secure Debug

   ```bash
   commander security lockconfig --secure-debug-unlock enable --device EFR32MG24B210F1536 --serialno 440263650
   ```

5. Locking the device depends on whether the application will be TrustZone aware or not

   For TrustZone `unaware` application use the below command

   ```bash
   commander security lock --device EFR32MG24B210F1536 --serialno 440263650
   ```

   For TrustZone `aware` application use the below command

   ```bash
   commander security lock --trustzone 1100 --device EFR32MG21A010F1024 --serialno 440048205
   ```

6. Finally disable device erase

   ```bash
   commander security disabledeviceerase --device EFR32MG24B210F1536 --serialno 440263650
   ```

7. Check the device security status 

   ```bash
   commander security status --device EFR32MG24B210F1536 --serialno 440263650
   or
   commander security status --trustzone --device EFR32MG24B210F1536 --serialno 440263650
   ```

8. Unlock debug access
   To securely unlock debug access a whole flow with tokenized challenge is put in place.

   Unlock should follow section 6.4.2 of [AN1190](https://www.silabs.com/documents/public/application-notes/an1190-efr32-secure-debug.pdf)

   The initial idea was that third parties can get debug access provided by the device manufatcurer, without sharing the private command key

   It is possible to simplify section 6.4.2 by using the command private key as follows (**RISK OF PRIVATE KEY LEAK**):

   ```bash
   commander security unlock --command-key command_key.pem --unlock-param 1111 --device EFR32MG24B210F1536 --serialno 440263650
   ```
