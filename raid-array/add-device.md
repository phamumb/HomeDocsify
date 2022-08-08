# Adding a Spare Device to an Array
Spare devices can be added to any arrays that offer redundancy (such as RAID 1, 5, 6, or 10). The spare will not be actively used by the array unless an active device fails. When this happens, the array will resync the data to the spare drive to repair the array to full health. Spares cannot be added to non-redundant arrays (RAID 0) because the array will not survive the failure of a drive.

To add a spare, simply pass in the array and the new device to the mdadm --add command:
```
    sudo mdadm /dev/md0 --add /dev/sde
```
If the array is not in a degraded state, the new device will be added as a spare. If the device is currently degraded, the resync operation will immediately begin using the spare to replace the faulty drive.

After you add a spare, update the configuration file to reflect your new device orientation:
```
    sudo nano /etc/mdadm/mdadm.conf
```
Remove or comment out the current line that corresponds to your array definition:
```/etc/mdadm/mdadm.conf
. . .
# ARRAY /dev/md0 metadata=1.2 name=mdadmwrite:0 UUID=d81c843b:4d96d9fc:5f3f499c:6ee99294
```

Afterwards, append your current configuration:
```
    sudo mdadm --detail --brief /dev/md0 | sudo tee -a /etc/mdadm/mdadm.conf
```
The new information will be used by the mdadm utility to assemble the array.
