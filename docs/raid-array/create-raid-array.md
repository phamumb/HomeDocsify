# How To Create RAID Arrays with mdadm on Ubuntu 16.04
![raid-array](assets/raid-array.png)
## Resetting Existing RAID Devices
Find the active arrays in the /proc/mdstat file by typing:
```
cat /proc/mdstat
```

```
Output
Personalities : [raid0] [linear] [multipath] [raid1] [raid6] [raid5] [raid4] [raid10] 
md0 : active raid0 sdc[1] sdd[0]
      209584128 blocks super 1.2 512k chunks
            
            unused devices: <none>
```

Unmount the array from the filesystem:
```
sudo umount /dev/md0
```

Then, stop and remove the array by typing:
```
sudo mdadm --stop /dev/md0
sudo mdadm --remove /dev/md0
```

Find the devices that were used to build the array with the following command:
```
lsblk -o NAME,SIZE,FSTYPE,TYPE,MOUNTPOINT
```
```console
Output
NAME     SIZE FSTYPE            TYPE MOUNTPOINT
sda      100G                   disk 
sdb      100G                   disk 
sdc      100G linux_raid_member disk 
sdd      100G linux_raid_member disk 
vda       20G                   disk 
├─vda1    20G ext4              part /
└─vda15    1M                   part 
```

After discovering the devices used to create an array, zero their superblock to reset them to normal:
```
    sudo mdadm --zero-superblock /dev/sdc
    sudo mdadm --zero-superblock /dev/sdd
```

You should remove any of the persistent references to the array. Edit the `/etc/fstab` file and comment out or remove the reference to your array:
``` /etc/fstab
    sudo nano /etc/fstab
```

```
/dev/md0 /mnt/md0 ext4 defaults,nofail,discard 0 0
```
Also, comment out or remove the array definition from the /etc/mdadm/mdadm.conf file:
``` /etc/mdadm/mdadm.conf
sudo nano /etc/mdadm/mdadm.conf
```
```
ARRAY /dev/md0 metadata=1.2 name=mdadmwrite:0 UUID=7261fb9c:976d0d97:30bc63ce:85e76e91
```

Finally, update the initramfs again:
```
sudo update-initramfs -u
```
At this point, you should be ready to reuse the storage devices individually, or as components of a different array.

## Creating a RAID 5 Array

The RAID 5 array type is implemented by striping data across the available devices. One component of each stripe is a calculated parity block. If a device fails, the parity block and the remaining blocks can be used to calculate the missing data. The device that receives the parity block is rotated so that each device has a balanced amount of parity information.

- Requirements: minimum of 3 storage devices
- Primary benefit: Redundancy with more usable capacity.
- Things to keep in mind: While the parity information is distributed, one disk’s worth of capacity will be used for parity. RAID 5 can suffer from very poor performance when in a degraded state.

### Identify the Component Devices
To get started, find the identifiers for the raw disks that you will be using:
```
    lsblk -o NAME,SIZE,FSTYPE,TYPE,MOUNTPOINT
```
```
Output
NAME     SIZE FSTYPE TYPE MOUNTPOINT
sda      100G        disk
sdb      100G        disk
sdc      100G        disk
vda       20G        disk 
├─vda1    20G ext4   part /
└─vda15    1M        part
```

As you can see above, we have three disks without a filesystem, each 100G in size. In this example, these devices have been given the `/dev/sda`, `/dev/sdb`, and `/dev/sdc` identifiers for this session. These will be the raw components we will use to build the array.

### Create the Array

To create a RAID 5 array with these components, pass them in to the mdadm --create command. You will have to specify the device name you wish to create (/dev/md0 in our case), the RAID level, and the number of devices:

```
sudo mdadm --create --verbose /dev/md0 --level=5 --raid-devices=3 /dev/sda /dev/sdb /dev/sdc
```

The mdadm tool will start to configure the array (it actually uses the recovery process to build the array for performance reasons). This can take some time to complete, but the array can be used during this time. You can monitor the progress of the mirroring by checking the /proc/mdstat file:

```
cat /proc/mdstat
```

```
Output
Personalities : [raid1] [linear] [multipath] [raid0] [raid6] [raid5] [raid4] [raid10] 
md0 : active raid5 sdc[3] sdb[1] sda[0]
      209584128 blocks super 1.2 level 5, 512k chunk, algorithm 2 [3/2] [UU_]
      [===>.................]  recovery = 15.6% (16362536/104792064) finish=7.3min speed=200808K/sec
      
unused devices: <none>
```

### Create and Mount the Filesystem

Next, create a filesystem on the array:
```
    sudo mkfs.ext4 -F /dev/md0
```
Create a mount point to attach the new filesystem:
```
    sudo mkdir -p /mnt/md0
```
You can mount the filesystem by typing:
```
    sudo mount /dev/md0 /mnt/md0
```
Check whether the new space is available by typing:
```
    df -h -x devtmpfs -x tmpfs
```
```
Output
Filesystem      Size  Used Avail Use% Mounted on
/dev/vda1        20G  1.1G   18G   6% /
/dev/md0        197G   60M  187G   1% /mnt/md0
```
The new filesystem is mounted and accessible.
### Save the Array Layout

To make sure that the array is reassembled automatically at boot, we will have to adjust the /etc/mdadm/mdadm.conf file.

Before you adjust the configuration, check again to make sure the array has finished assembling. Because of the way that mdadm builds RAID 5 arrays, if the array is still building, the number of spares in the array will be inaccurately reported:
```
    cat /proc/mdstat
```
```
Output
Personalities : [raid1] [linear] [multipath] [raid0] [raid6] [raid5] [raid4] [raid10] 
md0 : active raid5 sdc[3] sdb[1] sda[0]
      209584128 blocks super 1.2 level 5, 512k chunk, algorithm 2 [3/3] [UUU]
      
unused devices: <none>
```
The output above shows that the rebuild is complete. Now, we can automatically scan the active array and append the file by typing:
```
    sudo mdadm --detail --scan | sudo tee -a /etc/mdadm/mdadm.conf
```

Afterwards, you can update the initramfs, or initial RAM file system, so that the array will be available during the early boot process:
```
    sudo update-initramfs -u
```
Add the new filesystem mount options to the /etc/fstab file for automatic mounting at boot:
```
    echo '/dev/md0 /mnt/md0 ext4 defaults,nofail,discard 0 0' | sudo tee -a /etc/fstab
```
Your RAID 5 array should now automatically be assembled and mounted each boot.