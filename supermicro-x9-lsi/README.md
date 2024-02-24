# Flashing SuperMicro X9 motherboards to LSI IT mode

## Intro

This guide was created to help you flash a Supermicro X9 motherboard's onboard Broadcom 2208 MegaRaid controller into an LSI 9207 HBA (IT mode). According to [SuperMicro's X9 motherboards](https://www.supermicro.com/products/motherboard/archive/?mlg=0) this should work for the following motherboards:

- [X9DAX-7F](https://www.supermicro.com/products/motherboard/Xeon/C600/X9DAX-7F.cfm)
- [X9DAX-7F-HFT](https://www.supermicro.com/products/motherboard/Xeon/C600/X9DAX-7F-HFT.cfm)
- [X9DAX-7TF](https://www.supermicro.com/products/motherboard/Xeon/C600/X9DAX-7TF.cfm)
- [X9DR7-TF+](https://www.supermicro.com/products/motherboard/Xeon/C600/X9DR7-TF_.cfm)
- [X9DRFF-7+](https://www.supermicro.com/products/motherboard/Xeon/C600/X9DRFF-7_.cfm)
- [X9DRFF-7T+](https://www.supermicro.com/products/motherboard/Xeon/C600/X9DRFF-7T_.cfm)
- [X9DRH-7F](https://www.supermicro.com/products/motherboard/Xeon/C600/X9DRH-7F.cfm)
- [X9DRH-7TF](https://www.supermicro.com/products/motherboard/Xeon/C600/X9DRH-7TF.cfm)
- [X9DRL-7F](https://www.supermicro.com/products/motherboard/Xeon/C600/X9DRL-7F.cfm)
- [X9DRW-7TPF+](https://www.supermicro.com/products/motherboard/Xeon/C600/X9DRW-7TPF_.cfm)
- [X9DRW-7TPF](https://www.supermicro.com/products/motherboard/Xeon/C600/X9DRW-7TPF.cfm)

The bare minimum amount of files are inside of this directory to get you up and running.

## How to Flash

1. Create a FreeDOS/MSDOS bootable USB (Create one at https://rufus.ie)

1. Copy the `supermicro-x9-lsi` folder from this repo to the root of the drive.

1. Boot to the FreeDOS USB you have created.

1. List all megaraid controllers in your system

   ```dos
   cd supermicro-x9-lsi
   megarec -adplist
   ```

1. Backup your SBR and SPD files in case you ever want to revert. Assuming you only have one sas controller `0` indicates the id from the `-adplist` command from #2. Name these files whatever you want. For me it made sense to name them after my motherboard model number `x9drl-7f`.

   ```dos
   megarec -readsbr 0 x9drl-7f.sbr
   megarec -readspd 0 x9drl-7f.spd
   ```

   The second command `readspd` failed on my motherboard with the following error:

   ```dos
   timeout

   SPD Read Failed.
   Error code = 16384
   ```

   I don't think this file is used even to revert to the original firmware, so ignore at your own risk (I did).

1. Backup all the info for your sas controller. We'll use this later to reset the SAS address.

   ```dos
   megascu -adpallinfo -a0 > x9drl-7f.txt
   ```

1. Now begins the flashing phase. Flash an empty sbr to the controller.

   ```dos
   megarec -writesbr 0 sbrempty.bin
   ```

   This should be fairly instant and output `Success`

1. Wipe the flash memory

   ```dos
   megarec -cleanflash 0
   ```

   This should take 5-10 seconds and output `Success`

   Ctrl+Alt+Delete (to reboot)

1. Boot to UEFI Shell (motherboard should have this as a boot option). Mine was "UEFI: Built-in EFI Shell"

1. See if you can now see the card.

   ```uefi
   fs0:
   cd supermicro-x9-lsi
   sas2flash -list
   ```

   You should now see your card listed.

1. Flash the IT firmware and the card BIOS

   ```uefi
   sas2flash -o -f 9207-8.bin -b mptsas2.rom
   ```

1. Now look at the sas card again and you should see a card with all zeros for it's sas address

   ```uefi
   sas2flash -list
   ```

1. Set the SAS address to the one in the `.txt` file you created earlier

   ```uefi
   sas2flash -o -sasadd 5003048011838700
   ```

   You're done! Reboot your computer and you should now see an LSI bootrom instead of a MegaRaid one.

## Credits

This guide created from pulling from the following sources:

- https://mywiredhouse.net/blog/flashing-lsi-2208-firmware-use-hba/
- https://forums.serverbuilds.net/t/flashing-sas2208-to-it-mode-when-sas2flsh-does-not-detect-it/2357
