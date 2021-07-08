# smartcard-luks-osk

OpenPGP smartcard setup with Linux boot for the PinePhone with Mobian (Librem 5 with PureOS might also work).
This is a modified version of the Luks GPG encryption configuration Script from Purism (https://source.puri.sm/pureos/packages/smartcard-key-luks). This version is extended to replace the standard decrypt_gnupg-sc keyscript and use acustom version of the keyscript that integrates the osk by Postmarket OS (https://gitlab.com/postmarketOS/osk-sdl).
The modified keyscript (files/decrypt_gnupg-sc-osk) will try to use the smartcard for around 90 secondes. After that the keyboard will be shown too, but the output is then directly forwarded without first going through gpg. If you still have a Luks keyslot with a normal passphrase, this can be used as a fallback, as every keyslot will be checked. If you get the on screen keyboard before 90 seconds are over, this means the smartcard was detected and you need to provide your pin for the gpg key and not a passphrase for a different Luks keyslot.

## Notes

There is no real visual feedback. If you type in your pin/password incorrectly, the keyscript will start again from the beginning. 
Smartcards normaly block after 3 retries, don't retry too often with your smartcard connected. 

Normal osk full disk encryption needs to be configured beforehand.

The script was tested with a fresh Mobian eMMC install using the installer version 20210516, but there is no gurantee it won't break your encrypted volume or the unlock mechanism.
Also please be aware that every change in the official osk package could break this.

Your previous Luks keyslots won't be deleted by the script.

I noticed some problems with the recognition of my USB smartcard. Based on the logs from the UART console I think this might be a problem with the anx7688 kernel module, which is handling the USB-C port controller. This module is needed and will be added to the modules in the initramfs by the script (if you define "pinephone"/"pp" as device). I had the best results inserting the smartcard when the initramfs was already up. 

## Usage

1. cd into the folder of the repository
2. ./smartcard-luks-osk "your public key (file)" "device name"

For now there is only the pinephone. You can hand over "pinephone" or "pp" as device values. This will ensure the anx7688 kernel module (USB-C port controller) will be loaded in the initramfs.

## Steps the script performs

1. Getting the Luks device
2. Checking if scdaemon is installed. If not it tries to install it
3. Gererating a keyfile and pubring, encrypting keyfile with gpg, placing files in /etc/cryptsetup-initramfs, testing decryption, adding key to Luks device
5. Creating new initramfs hook (/usr/share/initramfs-tools/hooks/cryptgnupg-sc-osk)
6. Copying new keyscript to /lib/cryptsetup/scripts/decrypt_gnupg-sc-osk
7. Adding anx7688 to /etc/initramfs-tools/modules if device is pinephone
8. Adding keyfile to crypttab, replacing keyscript entry in crypttab

## License

Unless otherwise stated in the source files themself, the code is licensed under the license given in the LICENSE file within the main directory of the repository (LGPLv3). 
