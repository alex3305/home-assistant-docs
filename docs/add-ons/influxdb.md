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

See also [InfluxDB: backup and restore](influxdb-backup.md).

## Downsampling data

See also [InfluxDB: downsampling](influxdb-downsampling.md).
