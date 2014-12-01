Notes for my zfs->btrfs migration
==============

### Boot into Linux Mint 17.1 Cinnamon live CD/USB

I'm using the following mounts, yours may differ:

1. `/dev/sda` -> a large hard drive
2. `/dev/sdb` -> a temporary USB stick with the Ubuntu installer on it
3. `/dev/sdc` -> a permanent USB stick to boot from

Pre-boot:

1. Enable UEFI boot in bios
2. Start "Try Ubuntu" to full boot

In "Try Ubuntu":

5. `apt-get install btrfs-tools`

Start Ubuntu installer

1. Select to make your own partitions
2. Select /dev/sda to be btrfs, formated, mount /
3. On /dev/sdc1 (USB boot stick), make first partition at least 10 MB of type "EFI Boot Partition"
4. Make rest of sdc into a partition: /dev/sdc2, ext2, format, mount /boot
5. Walk through installer
6. Reboot, remove /dev/sdb, check that it boots correctly

# Post reboot:

1. `apt-get update`
2. `apt-get install apt-btrfs-snapshot`
2. `apt-get ugprade`
3. Add second /btrfs drive: `btrfs device add /dev/sdb /`
4. Convert to raid1: `btrfs balance -mconvert=raid1 -dconvert=raid1 /`
5. Reeboot 


## Migrate old users, /home

```
mkdir /mnt/btrfs
mount /dev/sda /mnt/btrfs
cd !$
btrfs su sn @ @_1_installed
btrfs su sn @home @_home_1_installed
```

Follow migration guide (using `export UGIDLIMIT=1000`): http://www.cyberciti.biz/faq/howto-move-migrate-user-accounts-old-to-new-server/


# Old stuff:

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

## Before the reboot:
```
umount /boot
exit # chroot
cd /mnt/btrfs/@
for dir in dev proc sys; do umount ./$dir; done  # get some error here
lsof proc  # does it show anything for why you can't unmount...anyway...carry on.
umount /mnt/sda2
```

# old stuff (WIP):


```
df -h
mkfs.btrfs -L btrfs_pool -m raid1 -d raid1 /dev/sda /dev/sdb
btrfs filesystem show
mkdir /mnt/btrfs_pool
mount /dev/sdb /mnt/btrfs_pool  # pick any dev from raid
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
mint / # mount -o compress,subvol=@ /dev/sdb /mnt/btrfs_compat_dirs/
mint / # mount -o compress,subvol=@home /dev/sdb /mnt/btrfs_compat_dirs/home
mint / # mkdir /mnt/btrfs_compat_dirs/mnt/albums
mint / # mount -o subvol=@albums /dev/sdb /mnt/btrfs_compat_dirs/mnt/albums
mint / # mkdir /mnt/btrfs_compat_dirs/mnt/mythtv
mint / # mount -o subvol=@mythtv /dev/sdb /mnt/btrfs_compat_dirs/mnt/mythtv
mint / # mkdir /mnt/btrfs_compat_dirs/mnt/scratch
mint / # mount -o compress,subvol=@scratch /dev/sdb /mnt/btrfs_compat_dirs/mnt/scratch

rsync -av /path/to/sources/ /mnt/btrfs_compat_dirs/

#btrfs subvolume create /mnt/btrfs_pool/root  # gets mounted to / with -o subvol=root
#btrfs subvolume create /mnt/btrfs_pool/home  # gets mounted to /home with -o subvol=home
#btrfs subvolume create /mnt/btrfs_pool/mnt/albums # gets mounted to /mnt/albums with -o subvol=albums
#btrfs subvolume create /mnt/btrfs_pool/mnt/mythtv # gets mounted to /mnt/mythtv with -o subvol=mythtv
#btrfs subvolume create /mnt/btrfs_pool/mnt/scratch # gets mounted to /mnt/scratch with -o subvol=scratch
```

In /etc/fstab:
```
LABEL=btrfs_pool    /              btrfs   defaults,compress,subvol=@  0 0
LABEL=btrfs_pool    /home          btrfs   defaults,compress,subvol=@home  0 0
LABEL=btrfs_pool    /mnt/albums    btrfs   defaults,subvol=@albums         0 0
LABEL=btrfs_pool    /mnt/mythtv    btrfs   defaults,subvol=@mythtv         0 0
LABEL=btrfs_pool    /media/btrfs   btrfs   defaults,noauto,compress,subvolid=0    0 0
```

# Setup printer

1. http://chadchenault.blogspot.ca/2012/05/brother-hl-2270dw-printer-driver.html
2. Set to HQ1200 DPI
3. Set to NoTumble duplex
 
# Setup cloudprint

1. git clone https://github.com/armooo/cloudprint
2. cd cloudprint
3. cp -r cloudprint /opt/
4. cd /opt/cloudprint
5. ./cloudprint.py -c  # type in Google credentials (don't use your real Google account -- security risk)
6. apt-get install python-daemon
7. Add file `/etc/cron.d/cloudprint`:

```
# /etc/cron.d/cloudprint

SHELL=/bin/sh
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

@reboot   root /opt/cloudprint/cloudprint.py -d -a /root/.cloudprintauth -p /var/lib/cloudprint.pid	
@hourly   root /opt/cloudprint/cloudprint.py -d -a /root/.cloudprintauth -p /var/lib/cloudprint.pid	
```

Run `/opt/cloudprint/cloudprint.py -d -a /root/.cloudprintauth -p /var/lib/cloudprint.pid` and then check that `cat /var/lib/cloudprint.pid` has the process ID of the running process (use `ps auxf`).

# MythTv migration

Follow these instructions (http://www.mythtv.org/wiki/Backend_migration) and remember:

1. Use the new mythtv database password in /etc/mythtv/config.xml (must update all frontends to use new password)
2. Change the database server host from `localhost` to the computer's IP address in:
  1. /etc/mythtv/config.xml
  2. /etc/mysql/debian
  3. /etc/mysql/my.cnf  (the `bind` address)
