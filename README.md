btrfs
=====

# Create filesystem

```
df -h
mkfs.btrfs -L btrfs_pool -m raid1 -d raid1 /dev/sda /dev/sdb
btrfs filesystem show
mkdir /mnt/btrfs_pool
mount -o compress=lzo,noatime /dev/sda /mnt/btrfs_pool  # pick any dev from raid
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
