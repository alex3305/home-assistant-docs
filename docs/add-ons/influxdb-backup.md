# Backup and restore

## Manual InfluxDB backup from Home Assistant add-on

We can create a manual InfluxDB backup, which can be used for backup or testing on another system. These steps use the command line and Docker to create a backup.

```bash
# Let's get into the container
docker exec -it addon_a0d7b954_influxdb bash

# We create a temporary backup directory
mkdir -p /share/backup/influxdb/

# Create the backup to our new share directory
influxd backup -database homeassistant -portable /share/backup/influxdb/

# Exit the container
exit

# (Optional) Set so all users can read the backup data
sudo chmod a+r /usr/share/hassio/share/backup/influxdb/*

# (Optional) Afterwards remove the backup
sudo rm -rf /usr/share/hassio/share/backup/influxdb/*
```

## Manual InfluxDB backup restore to Home Assistant add-on

Restoring is also quite straight forward when you have created a portable backup.

```bash
# Let's get into the container
docker exec -it addon_a0d7b954_influxdb bash

# Restore the backup from the afformentioned directory
influxd restore -portable /share/backup/influxdb/
```

## Manual InfluxDB backup restore to local Docker container

This can be useful for testing purposes. First we create set up the Docker containers for testing.

```bash
docker network create influxdb
docker run -d -p 8086:8086 -v /home/${USER}/dump/influxdb:/backup --net influxdb --name influxdb influxdb
docker run -d -p 8888:8888 --net influxdb --name chronograf chronograf --influxdb-url=http://influxdb:8086
```

The steps above run InfluxDB with Chronograf. Chronograf provides a nice and easy to use web interface, which can be accessed on `http://localhost:8888`. After we have set up InfluxDB we can restore our backup.

```bash
# Create the directory where we get to store our backup
mkdir -p /home/${USER}/dump/influxdb
cd /home/${USER}/dump/influxdb

# Let's retrieve our backup from Home Assistant
scp homeassistant.lan:/usr/share/hassio/share/backup/influxdb/\* ./

# Let's get into our newly created Docker container
docker exec -it influxdb bash

# And restore our backup!
influxd restore -portable /backup
```

And when your done, cleaning up is easy too!

```bash
# Delete the Docker containers and network
docker rm -f chronograf influxdb
docker network rm influxdb

# Clean up our backup
rm -rf /home/${USER}/dump/influxdb
```

> _**Note** these steps must be done locally_
