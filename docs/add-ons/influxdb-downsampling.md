# InfluxDB downsampling data

When you store all your sensor data from Home Assistant in InfluxDB, it will continue to grow. And of course storing all Home Assistant data is quite nice, but our data pool can exceed our disk space over time. This can be solved with using both continious queries (CQ) and retention policies (RP).

**Continious queries** are database queries that are ran when new data is added to the database. Thus running almost continuously. We can leverage this functionality to downsample our large term data.

We can also use **Retention policies* on our data to let old data time out. When this happens the old data is automatically deleted from our database. 

Combining both of these features, we can continuously downsample and delete older fine grained data, while retainig the downsampled data.

## Basic queries

```influxql
-- Running this query will show the current continuous queries.
SHOW CONTINUOUS QUERIES

-- Running this query will show the current retention policies on our database.
SHOW RETENTION POLICIES on "homeassistant"
```

## Downsampling data

I started downsampling data, because my original, inifite autogen RP was filling up with about 1.5GB-2GB/year. This doesn't seem much, but I also tend to keep a couple of weeks of snapshot backups. After I have applied the CQ and RP below, I reduced my InfluxDB storage pool to about 1.3GB.

### Infinite retention policy

Let's create an infinite retention policy within InfluxDB on our database.

```influxql
CREATE RETENTION POLICY "infinite" ON "homeassistant" DURATION INF REPLICATION 1
```

### Downsample continuous query

We can downsample our data quite easily by creating a continuous query. I chose to downsample to 1 hour intervals, which is more than sufficiant for me.

```influxql
CREATE CONTINUOUS QUERY "cq_downsample_1h" ON "homeassistant"
BEGIN
  SELECT mean(*) INTO "homeassistant"."infinite".:MEASUREMENT FROM "homeassistant"."autogen"./.*/ 
  GROUP BY time(1h), *
  FILL(previous)
END
```

### Backfilling our data

After creating our RP and CQ, we can than backfill our data into our newly created infinite storage. This is needed, because InfluxDB only applies our new CQ to new data.

I have grouped my queries into periods of (about) 10 weeks. Otherwise it is possible to crash InfluxDB, halting our backfill. Below are the queries to backfill a year worth of data. If you have older data, you should add additional queries. 

```influxql
SELECT mean(*) INTO "homeassistant"."infinite".:MEASUREMENT FROM "homeassistant"."autogen"./.*/ WHERE time > now() - 52w and time < now() - 40w GROUP BY time(1h), * FILL(previous)
SELECT mean(*) INTO "homeassistant"."infinite".:MEASUREMENT FROM "homeassistant"."autogen"./.*/ WHERE time > now() - 41w and time < now() - 30w GROUP BY time(1h), * FILL(previous)
SELECT mean(*) INTO "homeassistant"."infinite".:MEASUREMENT FROM "homeassistant"."autogen"./.*/ WHERE time > now() - 31w and time < now() - 20w GROUP BY time(1h), * FILL(previous)
SELECT mean(*) INTO "homeassistant"."infinite".:MEASUREMENT FROM "homeassistant"."autogen"./.*/ WHERE time > now() - 21w and time < now() - 10w GROUP BY time(1h), * FILL(previous)
SELECT mean(*) INTO "homeassistant"."infinite".:MEASUREMENT FROM "homeassistant"."autogen"./.*/ WHERE time > now() - 11w and time < now() GROUP BY time(1h), * FILL(previous)
```

### Alter autogen RP

The final step in this proces is to alter our original, autogen RP to only store 26 weeks, or half a year. You can get away with a shorter RP, and even with a longer RP. For me, 26 weeks seems to be the sweet spot.

```influxql
ALTER RETENTION POLICY "autogen" on "homeassistant" DURATION 26w SHARD DURATION 1d DEFAULT
```
