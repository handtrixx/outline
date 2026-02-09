# Filesystems

## BTRFS

Assuming we have have formatted the system with BTRFS as documented before

```javascript
sudo rm -rf /var/lib/docker

# Create subvolumes
sudo btrfs subvolume create /var/lib/docker
sudo btrfs subvolume create /data

sudo chown -R handtrixxx: /data

sudo btrfs subvolume list /
```


configure snapshots with "sudo crontab -e"

```bash
# Daily btrfs snapshots at 3:15
15 1 * * * btrfs subvolume snapshot -r /var/lib/docker /var/lib/docker-snap-$(date +\%F)
15 1 * * * btrfs subvolume snapshot -r /data /data-snap-$(date +\%F)

# Cleanup older than 14 days at 3:45
45 1 * * * find / -maxdepth 1 -name "docker-snap-*" -type d -mtime +14 -exec btrfs subvolume delete {} \;
45 1 * * * find / -maxdepth 1 -name "data-snap-*" -type d -mtime +14 -exec btrfs subvolume delete {} \;
```

## ZFS

ZFS doesn't come with most OS's but has very good compatibility e.g. to Ubuntu.

To Install openzfs:

```bash
sudo apt-get install zfsutils-linux
```


\
Run "lsblk" to show available disks and their basic config:

```bash
handtrixxx@handtrixxx / $ lsblk
NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
sda      8:0    0 953.9G  0 disk
├─sda1   8:1    0    32G  0 part [SWAP]
├─sda2   8:2    0     1G  0 part /boot
└─sda3   8:3    0 920.9G  0 part /
sdb      8:16   0   7.3T  0 disk
sdc      8:32   0   7.3T  0 disk 
```

create zfs mirrored pool

```bash
sudo zpool create zfsdata mirror /dev/sdb /dev/sdc

sudo zpool status zfsdata
```

create datasets

```bash
sudo zfs create zfsdata/apps
sudo zfs create zfsdata/backup
sudo zfs create zfsdata/downloads
sudo zfs create zfsdata/media
sudo zfs create zfsdata/sources

sudo chown -R handtrixxx: /zfsdata/apps
sudo chown -R handtrixxx: /zfsdata/backup
sudo chown -R handtrixxx: /zfsdata/downloads
sudo chown -R handtrixxx: /zfsdata/media
sudo chown -R handtrixxx: /zfsdata/sources
```


create job for snapshots by running "sudo crontab -e"

```bash
0 3 * * * zfs snapshot -r zfsdata@daily-$(date +\%F)
30 3 * * * zfs list -t snapshot -o name,creation -Hp | awk -v cutoff=$(date -d '14 days ago' +\%s) '$2 < cutoff {print $1}' | xargs -r -n1 zfs destroy
```

verify

```bash
sudo zfs list -t snapshot
```

set zfs parameters recursive

```bash
zfs get compression zfsdata
sudo zfs set atime=off zfsdata
```

##