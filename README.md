# proxmox-cookbook

## Resize VM Guest Disk

> ONLINE = Resize operation performed while filesystem is mounted and in use
> 
> OFFLINE = Resize performed when filesystem is unmounted (not in use)
> 
> (Below we will see the ONLINE method without unmount)

1. **Resizing guest disk**

qm command:

```shell
qm resize <vmid> <disk> <size>
```

You can also select your VM from the list > Hardware > Hard Disk > Disk Action > Resize
   
<img width="200" height="120" alt="image" src="https://github.com/user-attachments/assets/fac76f0a-7070-4527-a9fd-4f0cae2a8a34" />


2. **Enlarge the partition(s) in the virtual disk**

**Online for Linux Guests**

Here we will enlarge a LVM PV partition, but the procedure is the same for every kind of partitions. Note that the partition you want to enlarge should be at the end of the disk. If you want to enlarge a partition which is anywhere on the disk,   use the offline method.

**Below I extended the first partition even if it wasn't at the end, though it worked fine**
  
**Example with EFI**

```shell
# Find the disk and partition(s) ex /dev/sda1
lsblk
```

```shell
parted /dev/sda
(parted) print
Warning: Not all of the space available to /dev/sda appears to be used, you can
fix the GPT to use all of the space (an extra 268435456 blocks) or continue
with the current setting? 
Fix/Ignore? F
```

```shell
# Not the last partition at end of disk but still worked fine
(parted) resizepart 1 100%
(parted) quit
```

3. **Enlarge the filesystem(s) in the partitions on the virtual disk**
 
**Online for Linux guests with LVM**
 
Enlarge the physical volume to occupy the whole available space in the partition:

```shell
pvresize /dev/sda1
```

List logical volumes:

```shell
lvdisplay

--- Logical volume ---
LV Path                /dev/{volume group name}/root
LV Name                root
VG Name                {volume group name}
LV UUID                DXSq3l-Rufb-...
LV Write Access        read/write
LV Creation host, time ...
LV Status              available
# open                 1
LV Size                <19.50 GiB
Current LE             4991
Segments               1
Allocation             inherit
Read ahead sectors     auto
- currently set to     256
Block device           253:0
```

Enlarge the logical volume and the filesystem (the file system can be mounted, works with ext4 and xfs). Replace "{volume group name}" with your specific volume group name:

```shell
# Use all the remaining space on the volume group
lvresize --extents +100%FREE --resizefs /dev/{volume group name}/root
```

**Online for Linux guests without LVM**

```shell
resize2fs /dev/sda1
```

[See Proxmox official docs as reference](https://pve.proxmox.com/wiki/Resize_disks)
  
