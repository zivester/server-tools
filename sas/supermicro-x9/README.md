# Flashing SuperMicro X9 motherboards to LSI IT mode

1. Boot to FreeDOS (Create one at https://rufus.ie)

2. List all megaraid controllers in your system

```dos
megarec -adplist
```

3. Backup your SBR and SPD files in case you ever want to revert. Assuming you only have one sas controller `0` indicates the id from the `-adplist` command from #2. Name these files whatever you want. For me it made sense to name them after my motherboard model number `x9drl-7f`.

```dos
megarec -readsbr 0 x9drl-7f.sbr
megarec -readspd 0 x9drl-7f.spd
```

The `readspd` failed on my motherboard with the following error.:

```dos
timeout

SPD Read Failed.
Error code = 16384
```

I don't think this file is used even to revert to the original firmware, so ignore at your own risk (I did).

4. Backup all the info for your sas controller. We'll use this later to reset the SAS address.

```dos
megascu -adpallinfo -a0 > x9drl-7f.txt
```

5. Now begins the flashing phase. Flash an empty sbr to the controller.

```dos
megarec -writesbr 0 sbrempty.bin
```

This should be fairly instand and output `Success`

6. Wipe the flash memory

```dos
megarec -cleanflash 0
```

This should take 5-10 seconds and output `Success`

Ctrl+Alt+Delete (to reboot)

7. Boot to UEFI Shell (motherboard should have this as a boot option). Mine was "UEFI: Built-in EFI Shell"

8. See if you can now see the card.

```uefi
fs0:
cd supermicro-x9
sas2flash -list
```

You should now see your card listed.

9. Flash the IT firmware and the card BIOS

```uefi
sas2flash -o -f 9207-8.bin -b mptsas2.rom
```

10. Now look at the sas card again and you should see a card with all zeros for it's sas address

```uefi
sas2flash -list
```

11. Set the SAS address to the one in the `.txt` file you created earlier

```uefi
sas2flash -o -sasadd 5003048011838700
```

You're done! Reboot your computer and you should now see an LSI bootrom instead of a MegaRaid one.

## Credits

- https://mywiredhouse.net/blog/flashing-lsi-2208-firmware-use-hba/
- https://forums.serverbuilds.net/t/flashing-sas2208-to-it-mode-when-sas2flsh-does-not-detect-it/2357
