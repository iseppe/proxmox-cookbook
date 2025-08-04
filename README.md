# proxmox-cookbook

## Index
- [Resize VM Guest Disk](https://github.com/iseppe/proxmox-cookbook/blob/master/README.md#resize-vm-guest-disk)
- [Enable Serial Terminal (xterm.js)](https://github.com/iseppe/proxmox-cookbook/blob/master/README.md#enable-serial-terminal-xtermjs)
- [Mount NFS Share (OMV)](https://github.com/iseppe/proxmox-cookbook/blob/master/README.md#mount-nfs-share-omv)
- [Resize xterm.js Window](https://github.com/iseppe/proxmox-cookbook/blob/master/README.md#resize-xtermjs-window)

## Resize VM Guest Disk

> ONLINE = Resize operation performed while filesystem is mounted and in use
> 
> OFFLINE = Resize performed when filesystem is unmounted (not in use)
> 
> (Below we will see the ONLINE method without unmount)

**1. Resizing guest disk**

qm command:

```shell
qm resize <vmid> <disk> <size>
```

You can also select your VM from the list > Hardware > Hard Disk > Disk Action > Resize
   
<img width="200" height="120" alt="image" src="https://github.com/user-attachments/assets/fac76f0a-7070-4527-a9fd-4f0cae2a8a34" />


**2. Enlarge the partition(s) in the virtual disk**

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

**3. Enlarge the filesystem(s) in the partitions on the virtual disk**
 
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

## Enable Serial Terminal (xterm.js)

#### Add a Serial socket to the VM

```shell
qm set <vm_id> -serial0 socket
```
Or from GUI: `Hardware > Add > Serial Port`

#### Configure the Guest

**1. Check if you have the file `/etc/init/ttyS0.conf`. If not, create it:**

```shell
# ttyS0 - getty
#
# This service maintains a getty on ttyS0 from the point the system is
# started until it is shut down again.

start on stopped rc RUNLEVEL=[12345]
stop on runlevel [!12345]

respawn
exec /sbin/getty -L 115200 ttyS0 vt102
```

**2. Enable/start the systemd service**

```shell
systemctl enable serial-getty@ttyS0.service

systemctl start serial-getty@ttyS0.service
```

Alternatively, if your guest does not have systemd:

```shell
sudo start ttyS0
```

**You might consider creating the ttyS1.conf file as well, just as a backup in case you have a crash with ttyS0**

**3. Update GRUB2 configuration**

Edit this file `/etc/default/grub` to:

```shell
GRUB_CMDLINE_LINUX="quiet console=tty0 console=ttyS0,115200"
```

Then run:

```shell
# debian based
update-grub

# redhat based
grub2-mkconfig --output=/boot/grub2/grub.cfg
```

Now reboot you machine:

```shell
sudo reboot now
```

You should now be able to connect to the Serial Terminal or using xterm.js from the GUI

```shell
qm terminal <vm_id>
```

**Note: if it seems to be not working, and you have defined ttyS1, you can connect to it with the command:**

```shell
qm terminal <VMiD> -iface serial1
```

[See Proxmox Serial Terminal Reference](https://pve.proxmox.com/wiki/Serial_Terminal)

## Mount NFS Share (OMV)

> This section assumes you've already setup a NFS share (ex. OpenMediaVault)

**1. Test the exported path**

```shell
mount -t nfs <SERVER_IP>:path/to/nfs/export /path/to/local/mount
```

You can list the exported paths from the server:

```shell
showmount -e <SERVER_IP>

root@seedbox-ub24:~# showmount -e 10.0.0.117
Export list for 10.0.0.117:
/export                    10.0.0.0/24
/export/shared-folder-1000 10.0.0.0/24
```
**2. Edit `/etc/fstab/` and make the mount persistent**

```shell
<SERVER_IP>:/path/to/nfs/export /path/to/mount nfs defaults,_netdev,nofail,x-systemd.automount 0 0
```

- **defaults** – standard NFS options

- **_netdev** – ensures it waits for the network to be up

- **nofail** – lets boot continue even if the share is unavailable

- **x-systemd.automount** – auto-mounts on first access, improves boot time

**NOTE:** if using NFSv4, make sure to only include the root export path and not subpaths:

```shell
# NFSv4
# NO
10.0.0.117:/export/shared-folder-1000 /mnt/omv/data

# YES
10.0.0.117:/export /mnt/omv/data
```

**3. Reload and mount**

```shell
# Unmout path (Maybe unnecessary, but I do it)
umount /path/local/mount

# Reload systemd daemon
systemctl daemon-reload

# Mount the share
mount -a
```

## Resize xterm.js window

**1. Append the following function to `/etc/profile`:**

```shell
res() {

  old=$(stty -g)
  stty raw -echo min 0 time 5

  printf '\0337\033[r\033[999;999H\033[6n\0338' > /dev/tty
  IFS='[;R' read -r _ rows cols _ < /dev/tty

  stty "$old"

  # echo "cols:$cols"
  # echo "rows:$rows"
  stty cols "$cols" rows "$rows"
}

[ $(tty) = /dev/ttyS0 ] && res
```

**2. Logout**

You should now be able to resize the window correctly. (If it does not work, try resizing before performing the login)

## Remove Proxmox No Subscription Popup

**NOTE: the popup will come back whenever you upgrade your packages**

**1. Edit `/usr/share/javascript/proxmox-widget-toolkit/proxmoxlib.js`**

Find the string `res.data.status.toLowerCase() !== 'active'` and replace `!==` with `==`

**2. Restart pveproxy.service**

```shell
systemctl restart pveproxy.service
```

**OPTIONAL:** [Check this tool to automate this process between upgrades (not tested)](https://github.com/foundObjects/pve-nag-buster)
