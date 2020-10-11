# InfluxDB

I use the [InfluxDB](https://www.influxdata.com/) time-series database for retaining all my sensor data. InfluxDB is a mature database with plenty of support from the Home Assistant community.

## Add-on configuration

Installing this add-on is pretty straight forward, as it's available as an official Home Assistant add-on. I use the default settings this add-on provides.

After this add-on is installed and running, I created a database and an user that has correct rights on the database.

## Home Assistant configuration

Configuration within Home Assistant is also fairly straight forward. Just use the [influxdb integration](https://www.home-assistant.io/integrations/influxdb/). Since we are using the add-on, we can also leverage the DNS from Home Assistant Supervisor.

```yaml
influxdb:
  host: a0d7b954-influxdb
  port: 8086
  database: homeassistant
  username: homeassistant
  password: homeassistant
  max_retries: 3
  default_measurement: state
  exclude:
    domains:
      - zwave
      - automation
    entities:
      - sensor.date
      - sensor.date_time
      - sensor.time
```

As you can see, I exclude the Z-Wave and automation domains, since they can get quite verbose with a extensive setup. Also I exclude the date and time sensors, since they would log every second.

## Backup and restore

### Manual InfluxDB backup from Home Assistant add-on

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

### Manual InfluxDB backup restore to Home Assistant add-on

Restoring is also quite straight forward when you have created a portable backup.

```bash
# Let's get into the container
docker exec -it addon_a0d7b954_influxdb bash

# Restore the backup from the afformentioned directory
influxd restore -portable /share/backup/influxdb/
```

### Manual InfluxDB backup restore to local Docker container

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

## Continious queries and retention policies

Storing all Home Assistant data is quite nice, but our data pool can get quite large over time. This can be solved with using both continious queries (CQ) and retention policies (RP).

**Continious queries** are database queries that are ran when new data is added to the database. Thus running almost continuously. We can leverage this functionality to downsample our large term data.

We can also use **Retention policies* on our data to let old data time out. When this happens the old data is automatically deleted from our database. 

Combining both of these features, we can continuously downsample and delete older fine grained data, while retainig the downsampled data.

### Info queries

```influxql
-- Running this query will show the current continuous queries.
SHOW CONTINUOUS QUERIES

-- Running this query will show the current retention policies on our database.
SHOW RETENTION POLICIES on "homeassistant"
```

### Downsample data

```influxql
-- Create a retention policy for our infinite storage
CREATE RETENTION POLICY "infinite" ON "homeassistant" DURATION INF REPLICATION 1

-- Create a continuous query for downsampling our data
--   Let's group by 1h intervals, this can be adjusted to your likings
CREATE CONTINUOUS QUERY "cq_downsample_1h" ON "homeassistant"
BEGIN
  SELECT mean(*) INTO "homeassistant"."infinite".:MEASUREMENT FROM "homeassistant"."autogen"./.*/ 
  GROUP BY time(1h), *
  FILL(previous)
END

-- Show our just created CQ and RP
SHOW CONTINUOUS QUERIES
SHOW RETENTION POLICIES ON homeassistant 

-- Backfill our data for up to two years (can be longer if you need to)
--   Let's do this with intervals of 10 weeks, otherwise InfluxDB can crash!
SELECT mean(*) INTO "homeassistant"."infinite".:MEASUREMENT FROM "homeassistant"."autogen"./.*/ WHERE time > now() - 101w and time < now() - 90w GROUP BY time(1h), * FILL(previous)
SELECT mean(*) INTO "homeassistant"."infinite".:MEASUREMENT FROM "homeassistant"."autogen"./.*/ WHERE time > now() - 91w and time < now() - 80w GROUP BY time(1h), * FILL(previous)
SELECT mean(*) INTO "homeassistant"."infinite".:MEASUREMENT FROM "homeassistant"."autogen"./.*/ WHERE time > now() - 81w and time < now() - 70w GROUP BY time(1h), * FILL(previous)
SELECT mean(*) INTO "homeassistant"."infinite".:MEASUREMENT FROM "homeassistant"."autogen"./.*/ WHERE time > now() - 71w and time < now() - 60w GROUP BY time(1h), * FILL(previous)
SELECT mean(*) INTO "homeassistant"."infinite".:MEASUREMENT FROM "homeassistant"."autogen"./.*/ WHERE time > now() - 61w and time < now() - 50w GROUP BY time(1h), * FILL(previous)
SELECT mean(*) INTO "homeassistant"."infinite".:MEASUREMENT FROM "homeassistant"."autogen"./.*/ WHERE time > now() - 51w and time < now() - 40w GROUP BY time(1h), * FILL(previous)
SELECT mean(*) INTO "homeassistant"."infinite".:MEASUREMENT FROM "homeassistant"."autogen"./.*/ WHERE time > now() - 41w and time < now() - 30w GROUP BY time(1h), * FILL(previous)
SELECT mean(*) INTO "homeassistant"."infinite".:MEASUREMENT FROM "homeassistant"."autogen"./.*/ WHERE time > now() - 31w and time < now() - 20w GROUP BY time(1h), * FILL(previous)
SELECT mean(*) INTO "homeassistant"."infinite".:MEASUREMENT FROM "homeassistant"."autogen"./.*/ WHERE time > now() - 21w and time < now() - 10w GROUP BY time(1h), * FILL(previous)
SELECT mean(*) INTO "homeassistant"."infinite".:MEASUREMENT FROM "homeassistant"."autogen"./.*/ WHERE time > now() - 11w and time < now() GROUP BY time(1h), * FILL(previous)

-- Alter our Home Assistant autogen retention policy to retain a half a year of data
ALTER RETENTION POLICY "autogen" on "homeassistant" DURATION 26w SHARD DURATION 1d DEFAULT
```

Setting these CQ and RP reduced my dataset of two years by about 2GB (~3.3GB -> ~1GB).

> _**Note** I recommend doing these steps from the CLI and not from Chronograf._

