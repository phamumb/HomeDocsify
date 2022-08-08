# Deleting an Array

To destroy an array, including all data contained within, begin by following the process we used to stop an array.

First, unmount the filesystem:
```
    cd ~
    sudo umount /mnt/md0
```
Next, stop the array:
```
    sudo mdadm --stop /dev/md0
```
Afterwards, delete the array itself with the --remove command targeting the RAID device:
```
    sudo mdadm --remove /dev/md0
```
Once the array itself is removed, you should use mdadm --zero-superblock on each of the component devices. This will erase the md superblock, a header used by mdadm to assemble and manage the component devices as part of an array. If this is still present, it may cause problems when trying to reuse the disk for other purposes.

You can see that the superblock is still present in the array by checking out the FSTYPE column in the lsblk --fs output:
```
    lsblk --fs
```
```
Output
NAME    FSTYPE            LABEL        UUID                                 MOUNTPOINT
sda     linux_raid_member mdadmwrite:0 bf7a711b-b3aa-9440-40d2-c12e79824706 
sdb     linux_raid_member mdadmwrite:0 bf7a711b-b3aa-9440-40d2-c12e79824706 
sdc     linux_raid_member mdadmwrite:0 bf7a711b-b3aa-9440-40d2-c12e79824706 
sdd                                                                         
vda                                                                         
├─vda1  ext4              DOROOT       4f8b85db-8c11-422b-83c4-c74195f67b91 /
└─vda15
```
In this example, `/dev/sda`, `/dev/sdb`, and `/dev/sdc` were all part of the array and are still labeled as such.

Remove the labels by typing:
```
    sudo mdadm --zero-superblock /dev/sda /dev/sdb /dev/sdc
```
Next, make sure you remove or comment out any references to the array in the /etc/fstab file:
```
    sudo nano /etc/fstab
```
```/etc/fstab

. . .
# /dev/md0 /mnt/md0 ext4 defaults,nofail,discard 0 0
```
Save and close the file when you are finished.

Remove or comment out any references to the array from the /etc/mdadm/mdadm.conf file as well:
```
    nano /etc/mdadm/mdadm.conf
```
```/etc/mdadm/mdadm.conf

# ARRAY /dev/md0 metadata=1.2 name=mdadmwrite:0 UUID=bf7a711b:b3aa9440:40d2c12e:79824706 
```
Save and close the file when you are finished.

Update the initramfs by typing:
```
    sudo update-initramfs -u
```
This should remove the device from the early boot environment.