# Plex QSV transcoding fix

When utilizing Plex transcoding with Quick Sync Video the output can be broken on newer Intel systems. Which is also the case on my Intel J4105 processor. However there is a quick fix available.

!!! note
    This issue seems resolved with Plex Media Server version 1.21.1.3766.

!!! note 
    Usage of Quick Sync Video is a Plex Pass feature.

## Removing iHD driver

If you simply remove the iHD driver on these newer Intel platforms, Plex utilizes the older i965 driver. Perhaps performance isn't that good, but it is still better than using the CPU for transcoding. 

Removing this file is as simple as:

```bash
sudo rm -f /usr/lib/plexmediaserver/lib/dri/iHD_drv_video.so
sudo systemctl restart plexmediaserver.service
```

## Systemctl unit file

To automate this process I installed a simple Systemctl unit file onto my home server.

```systemctl
[Unit]
Description=Remove Plex's iHD driver
Before=plexmediaserver.service

[Service]
Type=oneshot
ExecStart=/usr/bin/rm -f /usr/lib/plexmediaserver/lib/dri/iHD_drv_video.so
TimeoutStopSec=5
User=root

[Install]
WantedBy=multi-user.target plexmediaserver.service
```

To install this, put the contents above into `/usr/lib/systemd/system/remove-ihd-driver.service`, and run the appropiate systemctl commands below.

```bash
sudo systemctl daemon-reload
sudo systemctl start remove-ihd-driver
sudo systemctl enable remove-ihd-driver
```
