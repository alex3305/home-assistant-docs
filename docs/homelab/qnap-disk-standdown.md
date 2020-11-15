# Enabling QNAP disk spindown

I like to enable disk spindown within my NAS to save on power consumption. This behaviour should be enabled by default, but in my case it wasn't working correctly. After some time scouring the internet, I found that many users were having the same issues. 

With some online resources I found a way to truly enable QNAP disk spindown.

> _**NOTE** This is advanced usage. Please bear in mind that this modifies your default NAS behaviour._

## Requirements

* QNAP NAS
* NVMe SSD installed into your NAS

## Disable QTS storage

QTS creates a RAID1 array across all your drives inside of your NAS for it's OS storage. Initially I thought this was a clever idea. But it can (and will) prevent your drives from spindown. Even if you have a NVMe SSD installed. Since SSD's rarely die from wear in normal use, I decided to disable this RAID1 array most of the time.

However we rebuild this array once a day to keep our data synced to all our drives. Just in case a drive failure occurs. More on that later.

> _**NOTE** I've added this script to my `/shares/scripts/` directory, but it can also be placed inside of your `/root/` (or `~`) directory._

### Finding out which drives to disable

Just run `cat /proc/mdstat` which would return something like this:

```
Personalities : [linear] [raid0] [raid1] [raid10] [raid6] [raid5] [raid4] [multipath]

...

md13 : active raid1 sdd[3] sdb[2] sda[1] nvme0n1p4[0]
      458880 blocks super 1.0 [32/1] [U_______________________________]
      bitmap: 1/1 pages [4KB], 65536KB chunk

md9 : active raid1 sdd[3] sdb[2] sda[1] nvme0n1p4[0]
      530048 blocks super 1.0 [32/1] [U_______________________________]
      bitmap: 1/1 pages [4KB], 65536KB chunk
```

Since I've installed an NVMe drive, I only want to retain that drive for my QTS storage. Remember the other drives, because you will need to edit the scripts below to fit your storage solution.

### Disconnecting the QTS RAID1 array

First up we add a script to disconnect our internal QTS RAID1 array.

```bash
#!/bin/bash

HDDS="sda sdb sdd"

errquit() {
    STATUS=1
    _errlog "$@"
    exit ${STATUS}
}

echo "Disconnecting hdd's from /dev/md9 array"

for disk in ${HDDS}; do
    if [ ! -e /dev/${disk} ]; then
        errquit "Could not find /dev/${disk}"
    else
        mdadm /dev/md9 --fail /dev/${disk}1
    fi
done

echo "Disconneting hdd's from /dev/md13 array"

for disk in ${HDDS}; do
    if [ ! -e /dev/${disk} ]; then
        errquit "Could not find /dev/${disk}"
    else
        mdadm /dev/md13 --fail /dev/${disk}4
    fi
done
```

> _**NOTE** You should modify the `HDDS` parameter to suit your NAS and drive configuration._

### Rebuilding the QTS RAID1 array

Second up, we add a script to rebuild our QTS array once a day for data integrity should my NVMe drive fail.

```bash
#!/bin/bash

HDDS="sda sdb sdd"

errquit() {
    STATUS=1
    _errlog "$@"
    exit 1
}

echo "Rebuilding hdd's to /dev/md9 array"

for disk in ${HDDS}; do
    if [ ! -e /dev/${disk} ]; then
        errquit "Could not find /dev/${disk}"
    else
        mdadm /dev/md9 --re-add /dev/${disk}1
    fi
done

echo "Rebuilding hdd's to /dev/md13 array"

for disk in ${HDDS}; do
    if [ ! -e /dev/${disk} ]; then
        errquit "Could not find /dev/${disk}"
    else
        mdadm /dev/md13 --re-add /dev/${disk}4
    fi
done
```

## Disable swap

Since I've added 8GB of RAM to my NAS, I decided to disable swap memory. Because swap is also shared between (hard)drives. You could also add a swap file on the NVMe storage if you still want swap enabled.

```bash
echo "Turning system swap off"
swapoff -a
```

## Autorun

To get this all going, you can edit QTS's autorun file. Editing this file is different on different NAS devices. You can check QNAP's documentation on how to edit this file on your NAS.

### Editing autorun

```bash
mount $(/sbin/hal_app --get_boot_pd port_id=0)6 /tmp/config
vi /tmp/config/autorun.sh
chmod +x /tmp/config/autorun.sh
umount /tmp/config
```

### Autorun script

```bash
if crontab -l | grep -q 'rebuild_internal_raid'; then
    # Nothing to do; /etc/config/ is on a persistent storage and was already modified
    :
else
    echo "15 10 * * * /share/scripts/rebuild_internal_raid.sh" >> /etc/config/crontab
    crontab /etc/config/crontab && /etc/init.d/crond.sh restart
fi

if crontab -l | grep -q 'disconnect_internal_raid'; then
    # Nothing to do; /etc/config/ is on a persistent storage and was already modified
    :
else
    echo "30 10 * * * /share/scripts/disconnect_internal_raid.sh" >> /etc/config/crontab
    crontab /etc/config/crontab && /etc/init.d/crond.sh restart
fi

exec /share/scripts/disconnect_internal_raid.sh

echo "Turning system swap off"
swapoff -a
```

## References

* https://www.reddit.com/r/qnap/comments/fhh61n/new_ts328_hdds_not_spinning_down_qnap_say_its/
* https://forum.qnap.com/viewtopic.php?t=130788
* https://wiki.qnap.com/wiki/Add_items_to_crontab
