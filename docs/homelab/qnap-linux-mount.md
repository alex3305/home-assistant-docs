# Mounting QNAP disks in Debian Linux

I recently migrated to a QNAP NAS from an old DIY NAS for most of my home data. When starting to copy over my first batch of data I worked out that the data transfer was going to take about 12 - 18 hours on my 1GBps home network...

This was unacceptable to me, so I tried mounting my new shiny QNAP NAS disk (readily formatted) inside my old DIY NAS to speed up the transfer. After doing this, the transfer was about 6 times faster than copying all data over my network.

All in all this was a bit harder than anticipated, so I documented my steps.

> _**NOTE** This guide may not work for every RAID setup, but can provide a simple reference_

## QNAP side

On the QNAP side I had created a Single Volume with a single disk, no RAID whatsoever. On that disk I created a shared folder with a placeholder file that I could recognize for testing the mount.

I did not opt for a Storage Pool, because data redundency was not my primary goal. 

## Debian Linux side

Within Debian I had to install two packages first.

> _**NOTE** All these steps require root (`sudo`)!_

```bash
apt update
apt install -y mdadm lvm2
```

We need both [`mdadm`](https://en.wikipedia.org/wiki/Mdadm) and [`lvm2`](https://en.wikipedia.org/wiki/Logical_Volume_Manager_(Linux)) for managing the QNAP resource. After which we can scan for both RAID devices and LVM devices.

```bash
mdadm --assemble --scan
lvscan
```

The commands above are probably not mandatory, but they seem to get something going. Now we can look at which Volume Groups (VG) are found within the system.

```bash
vgdisplay
```

In my setup, this returned volume group `vg289`. I also noticed this with the `lvscan` command, but this is a bit more verbose and human readable. Now that we know the volume group (VG) name, we can activate it.

```bash
vgchange -a y vg289
```

This command will activate the volume group, if all goes well. I re-ran the `lvscan` command to check if the volume group was correctly activated and to look up the logical volume (LV) I wanted to utilize.

```bash
lvscan
  ACTIVE            '/dev/vg289/lv545' [<37.28 GiB] inherit
  ACTIVE            '/dev/vg289/lv2' [3.59 TiB] inherit
```

As you can probably tell, the `lv2` volume was the one I wanted to use. Since the LV was now active, I can mount it.

```bash
mkdir -p /mnt/qnap
mount /dev/vg289/lv2/ /mnt/qnap
```

Et voila. This mounted my QNAP locally on my Linux box. 

!!! note 
    This guide will also work with Debian derivatives, such as Ubuntu Linux

## References

* https://forum.qnap.com/viewtopic.php?t=93862
* https://superuser.com/a/666034

