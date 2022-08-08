# Removing a Device from an Array

Removing a drive from a RAID array is sometimes necessary if there is a fault or if you need to switch out the disk.

For a device to be removed, it must first be marked as “failed” within the array. You can check if there is a failed device by using mdadm --detail:
```
    sudo mdadm --detail /dev/md0
```
```
Output
/dev/md0:
        Version : 1.2
  Creation Time : Wed Aug 10 21:42:12 2016
     Raid Level : raid5
     Array Size : 209584128 (199.88 GiB 214.61 GB)
  Used Dev Size : 104792064 (99.94 GiB 107.31 GB)
   Raid Devices : 3
  Total Devices : 3
    Persistence : Superblock is persistent

    Update Time : Thu Aug 11 14:10:43 2016
          State : clean, degraded 
 Active Devices : 2
Working Devices : 2
 Failed Devices : 1
  Spare Devices : 0

         Layout : left-symmetric
     Chunk Size : 64K

           Name : mdadmwrite:0  (local to host mdadmwrite)
           UUID : bf7a711b:b3aa9440:40d2c12e:79824706
         Events : 144

    Number   Major   Minor   RaidDevice State
       0       0        0        0      removed
       1       8        0        1      active sync   /dev/sda
       2       8       16        2      active sync   /dev/sdb
       0       8       32        -      faulty   /dev/sdc
```
The highlighted lines all indicate that a drive is no longer functioning (/dev/sdc in this example).

If you need to remove a drive that does not have a problem, you can manually mark it as failed with the --fail option:
```
    sudo mdadm /dev/md0 --fail /dev/sdc
```
```
Output
mdadm: set /dev/sdc faulty in /dev/md0
```
If you look at the output of mdadm --detail, you should see that the device is now marked faulty.

Once the device is failed, you can remove it from the array with mdadm --remove:
```
    sudo mdadm /dev/md0 --remove /dev/sdc
```
```
Output
mdadm: hot removed /dev/sdc from /dev/md0
```
You can then replace it with a new drive, using the same mdadm --add command that you use to add a spare:
```
    sudo mdadm /dev/md0 --add /dev/sdd
```
```
Output
mdadm: added /dev/sdd
```
The array will begin to recover by copying data to the new drive.