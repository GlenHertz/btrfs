Notes for my zfs->btrfs migration
==============

### Boot into Ubuntu 14.10 live USB

1. Enable UEFI boot in bios
2. Use `gparted` to create a new GPT partition table
3. Make a 10MB partition 1 and set type to "bios_gpt", LABEL=BIOS
4. Make a 500MB partition 2 for ext4, LABEL=BOOT
5. Make the rest into a btrfs partition 3, LABEL=BTRFS
6. 

### From a Terminal:

```
mkdir /mnt/btrfs
mount -o defaults /dev/sda3 /mnt/btrfs
cd !$
# Setup standard Ubuntu subvolumes:
btrfs subvolume create @
btrfs subvolume create @home
# Copy old files across to btrfs:
mkdir /mnt/old
mount /dev/sdb2 /mnt/old
rsync -av --exclude 'mnt' --exclude 'home'  /mnt/old/ /mnt/btrfs/@/  # After we know we can boot cp excluded
cd @/etc
# find . -name '*zfs*'  # remove these files
cd /mnt/btrfs/@
for dir in dev proc sys; do mount -o bind /$dir ./$dir; done
chroot .
mount /dev/sda2 /boot
ls -l /dev/disk/by-uuid
```
### Edit `etc/fstab` to be something like:
```
# / was on /dev/sda3 during installation
UUID=15060261-d3a4-4c1d-840c-bce526a72e62 	/               btrfs defaults,subvol=@ 0 1
# /boot was on /dev/sda2 during installation
UUID=1183af99-3714-4e95-8cbc-8b20c3a9fb81       /boot           ext4    defaults        0       2
```

### Update software:
```
apt-get update
apt-get upgrade
```

# old stuff (WIP):


```
df -h
mkfs.btrfs -L btrfs_pool -m raid1 -d raid1 /dev/sda /dev/sdb
btrfs filesystem show
mkdir /mnt/btrfs_pool
mount -o compress=lzo,noatime /dev/sdb /mnt/btrfs_pool  # pick any dev from raid
btrfs subvolume list /mnt/btrfs_pool 
btrfs filesystem df /mnt/btrfs_pool # true usage numbers
df -h # false usage numbers?
umount /mnt/btrfs_pool
```

Install using Ubuntu installer (do not format, pick one dev or raid1 as / mount point)
```
mint ~ # btrfs subvolume list /mnt/btrfs_pool
ID 258 gen 26 top level 5 path @
ID 259 gen 21 top level 5 path @home
```

```
mint ~ # btrfs subvolume create /mnt/btrfs_pool/@albums
Create subvolume '/mnt/btrfs_pool/@albums'
mint ~ # btrfs subvolume create /mnt/btrfs_pool/@mythtv
Create subvolume '/mnt/btrfs_pool/@mythtv'
mint ~ # btrfs subvolume create /mnt/btrfs_pool/@scratch
Create subvolume '/mnt/btrfs_pool/@scratch'
mint ~ # btrfs subvolume list /mnt/btrfs_pool
ID 258 gen 26 top level 5 path @
ID 259 gen 21 top level 5 path @home
ID 263 gen 29 top level 5 path @albums
ID 264 gen 30 top level 5 path @mythtv
ID 265 gen 31 top level 5 path @scratch

mint / # umount /mnt/btrfs_pool
# Make a directory structure so it is the same as source copy:
mint / # mkdir /mnt/btrfs_compat_dirs
mint / # mount -o compress=lzo,noatime,subvol=@ /dev/sdb /mnt/btrfs_compat_dirs/
mint / # mount -o compress=lzo,noatime,subvol=@home /dev/sdb /mnt/btrfs_compat_dirs/home
mint / # mkdir /mnt/btrfs_compat_dirs/mnt/albums
mint / # mount -o noatime,subvol=@albums /dev/sdb /mnt/btrfs_compat_dirs/mnt/albums
mint / # mkdir /mnt/btrfs_compat_dirs/mnt/mythtv
mint / # mount -o noatime,subvol=@mythtv /dev/sdb /mnt/btrfs_compat_dirs/mnt/mythtv
mint / # mkdir /mnt/btrfs_compat_dirs/mnt/scratch
mint / # mount -o compress=lzo,noatime,subvol=@scratch /dev/sdb /mnt/btrfs_compat_dirs/mnt/scratch

rsync -av /path/to/sources/ /mnt/btrfs_compat_dirs/

#btrfs subvolume create /mnt/btrfs_pool/root  # gets mounted to / with -o subvol=root
#btrfs subvolume create /mnt/btrfs_pool/home  # gets mounted to /home with -o subvol=home
#btrfs subvolume create /mnt/btrfs_pool/mnt/albums # gets mounted to /mnt/albums with -o subvol=albums
#btrfs subvolume create /mnt/btrfs_pool/mnt/mythtv # gets mounted to /mnt/mythtv with -o subvol=mythtv
#btrfs subvolume create /mnt/btrfs_pool/mnt/scratch # gets mounted to /mnt/scratch with -o subvol=scratch
```

In /etc/fstab:
```
LABEL=btrfs_pool    /              btrfs   defaults,noatime,compress=lzo,subvol=@  0 0
LABEL=btrfs_pool    /home          btrfs   defaults,noatime,compress=lzo,subvol=@home  0 0
LABEL=btrfs_pool    /mnt/albums    btrfs   defaults,noatime,subvol=@albums         0 0
LABEL=btrfs_pool    /mnt/mythtv    btrfs   defaults,noatime,subvol=@mythtv         0 0
LABEL=btrfs_pool    /media/btrfs   btrfs   defaults,noauto,compress=lzo,subvolid=0    0 0
```
