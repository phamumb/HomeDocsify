# Increasing the Number of Active Devices in an Array

It is possible to grow an array by increasing the number of active devices within the assembly. The exact procedure depends slightly on the RAID level you are using.
## With RAID 1 or 10

Begin by adding the new device as a spare, just as demonstrated in the last section:
```
    sudo mdadm /dev/md0 --add /dev/sde
```
Find out the current number of RAID devices in the array:
```
    sudo mdadm --detail /dev/md0
```
```
Output
/dev/md0:
        Version : 1.2
  Creation Time : Wed Aug 10 15:29:26 2016
     Raid Level : raid1
     Array Size : 104792064 (99.94 GiB 107.31 GB)
  Used Dev Size : 104792064 (99.94 GiB 107.31 GB)
   Raid Devices : 2
  Total Devices : 3
    Persistence : Superblock is persistent

    . . .
```
We can see that in this example, the array is configured to actively use two devices, and that the total number of devices available to the array is three (because we added a spare).

Now, reconfigure the array to have an additional active device. The spare will be used to satisfy the extra drive requirement:
```
    sudo mdadm --grow --raid-devices=3 /dev/md0
```
The array will begin to reconfigure with an additional active disk. To view the progress of syncing the data, type:
```
    cat /proc/mdstat
```
You can continue to use the device as the process completes.
## With RAID 5 or 6

Begin by adding the new device as a spare, just as demonstrated in the last section:
```
    sudo mdadm /dev/md0 --add /dev/sde
```
Find out the current number of RAID devices in the array:
```
    sudo mdadm --detail /dev/md0
```
```
Output
/dev/md0:
        Version : 1.2
  Creation Time : Wed Aug 10 18:38:51 2016
     Raid Level : raid5
     Array Size : 209584128 (199.88 GiB 214.61 GB)
  Used Dev Size : 104792064 (99.94 GiB 107.31 GB)
   Raid Devices : 3
  Total Devices : 4
    Persistence : Superblock is persistent

    . . .
```
We can see that in this example, the array is configured to actively use three devices, and that the total number of devices available to the array is four (because we added a spare).

Now, reconfigure the array to have an additional active device. The spare will be used to satisfy the extra drive requirement. When growing a RAID 5 or RAID 6 array, it is important to include an additional option called --backup-file. This should point to a location off the array where a backup file containing critical information will be stored.

Note

The backup file is only used for a very short but critical time during this process, after which it will be deleted automatically. Because the time when this is needed is very brief, you will likely never see the file on disk, but in the event that something goes wrong, it can be used to rebuild the array. This post has some additional information if you would like to know more.
```
    sudo mdadm --grow --raid-devices=4 --backup-file=/root/md0_grow.bak /dev/md0
```
The following output indicates that the critical section will be backed up:
```
Output
mdadm: Need to backup 3072K of critical section..
```
The array will begin to reconfigure with an additional active disk. To view the progress of syncing the data, type:
```
    cat /proc/mdstat
```
You can continue to use the device as this process completes.

After the reshape is complete, you will need to expand the filesystem on the array to utilize the additional space:
```
    sudo resize2fs /dev/md0
```
Your array should now have a filesystem that matches its capacity.
## With RAID 0

Because RAID 0 arrays cannot have spare drives (there is no chance for a spare to rebuild a damaged RAID 0 array), we must add the new device at the same time that we grow the array.

First, find out the current number of RAID devices in the array:
```
    sudo mdadm --detail /dev/md0
```
```
Output
/dev/md0:
        Version : 1.2
  Creation Time : Wed Aug 10 19:17:14 2016
     Raid Level : raid0
     Array Size : 209584128 (199.88 GiB 214.61 GB)
   Raid Devices : 2
  Total Devices : 2
    Persistence : Superblock is persistent

    . . .
```
We can now increment the number of RAID devices in the same operation as the new drive addition:
```
    sudo mdadm --grow /dev/md0 --raid-devices=3 --add /dev/sdc
```
You will see output indicating that the array has been changed to RAID 4:
```
Output
mdadm: level of /dev/md0 changed to raid4
mdadm: added /dev/sdc
```
This is normal and expected. The array will transition back into RAID 0 when the data has been redistributed to all existing disks.

You can check the progress of the action by typing:
```
    cat /proc/mdstat
```
Once the sync is complete, resize the filesystem to use the additional space:
```
    sudo resize2fs /dev/md0
```
Your array should now have a filesystem that matches its capacity.