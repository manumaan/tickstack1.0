# Custom TICK stack Sandbox
This is a custom TICK stack sandbox with:  
TELEGRAF 1.13
INFLUXDB 1.8
CHRONOGRAF latest
KAPACITOR 1.5.5
GRAFANA 7.1.0 

### How to Run Sandbox

To run the `sandbox`, simply use the convenient cli:

```bash
$ ./sandbox
sandbox commands:
  up           -> spin up the sandbox environment (add -nightly to grab the latest nightly builds of InfluxDB and Chronograf)
  down         -> tear down the sandbox environment
  restart      -> restart the sandbox
  influxdb     -> attach to the influx cli
  flux         -> attach to the flux REPL

  enter (influxdb||kapacitor||chronograf||telegraf) -> enter the specified container
  logs  (influxdb||kapacitor||chronograf||telegraf) -> stream logs for the specified container

  delete-data  -> delete all data created by the TICK Stack
  docker-clean -> stop and remove all running docker containers
  rebuild-docs -> rebuild the documentation container to see updates
```

To get started just run `./sandbox up`. You browser will open two tabs:

- `localhost:8888` - Chronograf's address. You will use this as a management UI for the full stack
- `localhost:3010` - Documentation server. This contains a simple markdown server for tutorials and documentation.

### Grafana
Grafana can be accessed at localhost:3000 
Default credentials for Grafana: admin/admin 
Enter these and change the default password. 

Configuration:
Left side menu (+ icon)  -> create -> Dashboard -> Create a sample dashboard. Save. 
Left side menu (gear icon) -> configuration - Data Source 
 HTTP url:  http://influxdb:8086 
 Database: telegraf
Click Save & Test

> NOTE: Make sure to stop any existing installations of `influxdb`, `kapacitor` or `chronograf`. If you have them running the Sandbox will run into port conflicts and fail to properly start. In this case stop the existing processes and run `./sandbox restart`. Also make sure you are **not** using _Docker Toolbox_.

Once the Sandbox launches, you should see your dashboard appear in your browser:

![Dashboard](./documentation/static/images/landing-page.png)

You are ready to get started with the TICK Stack!

Click the Host icon in the left navigation bar to see your host (named `telegraf-getting-started`) and its overall status.
![Host List](./documentation/static/images/host-list.png)

You can click on `system` hyperlink to see a pre-built dashboard visualizing the basic system stats for your
host, then check out the tutorials at `http://localhost:3010/tutorials`.

If you are using the nightly builds and want to get started with Flux, make sure you check out the [Getting Started with Flux](./documentation/static/tutorials/flux-getting-started.md) tutorial.

> Note: see [influx-stress](https://github.com/influxdata/influx-stress) to create data for your Sandbox.

![Dashboard](./documentation/static/images/sandbox-dashboard.png)

### InfluxDB
Inserting data into InfluxDB CLI:
Login to the influxDB docker container from docker desktop. Enter the command 
```
influx
```
to start CLI
```
SHOW DATABASES - Lists databases
USE <DATABASE_NAME> - Select this DB for next commands. 
SHOW MEASUREMENTS - lists measurements
```
Example Insert command: 
```
INSERT redis,host=serverA,region=us_west value=0.64
```
This will insert one row into the redis measurement, which if did not exist till then will get created as well. 


### Querying influx DB:

To login to the influxdb cli, open a terminal on the influxdb container from docker desktop or using command prompt:
docker exec -it [<container_name ]  /bin/sh 
```
influx -precision rfc3339 (Precision setting allows to show timestamps in human-readable form)
```

```
show databases -> shows available databases
use [databasename] -> select the database for next commands
show measurements -> shows measurements
fields are numeric,non-indexed and tags are alphanumeric-indexed (mostly). Tags are strings.
show field keys from cpu - see all field names from cpu measurements
show tag keys from cpu - see all tag names from cpu measurements 
show tag values from cpu - see all tag values from cpu measurements
SHOW TAG VALUES CARDINALITY FROM cpu WITH KEY = "cpu" - number of distinct values of the tag 'cpu' in the measurement 'cpu'
```

At least one field column should be given in SELECT statement. 
We cannot use OR on the time values. 
Strings should be enclosed in quotes. 
In some error cases, no error is returned, and no data also. 
Cannot use maths within functions, ie not possible to do avg(gross-net)

GROUP BY TIME 
```
SELECT mean("water_level") FROM "h2o_feet" where time>='2019-09-17T20:00:00Z' group by time(10m)
SELECT mean("water_level") FROM "h2o_feet" where time>='2019-09-17T20:00:00Z' group by time(10m),location
SELECT mean("water_level") FROM "h2o_feet" where time>='2019-09-17T20:00:00Z' group by location, time(10m) fill(0)
```

CTAS in influxdb
```
SELECT * INTO "NOAA_water_database"."autogen".water_levels_bkp FROM h2o_feet GROUP BY *
```
ORDER BY TIME
ORDER by time DESC

```
SELECT mean("water_level") FROM "h2o_feet" WHERE "location" = 'santa_monica' AND time>='2019-09-17T20:00:00Z'  GROUP BY TIME(5m) ORDER BY time DESC LIMIT 5
```

Pagination
```
 SELECT "water_level","location" FROM "h2o_feet" LIMIT 3 OFFSET 3

 SELECT mean("water_level") FROM "h2o_feet" where time>='2019-09-17T20:00:00Z' tz('America/Chicago')
 SELECT mean("water_level") FROM "h2o_feet" where time>='2019-09-17T20:00:00Z' tz('Asia/Calcutta')

 SELECT "water_level" FROM "h2o_feet" WHERE time > now() - 1h

 SELECT "level description" FROM "h2o_feet" WHERE time >= '2019-09-17T21:36:00Z'
```

### InfluxDB Relay 
Influxdb relay adds a high availability layer to the InfluxDB. Here for demo purpose, there is only one backend on the Relay, not real high availability :) 
To write to the InfluxDB using the relay endpoint, below command can be used from the Relay container terminal: 

```
curl -i -XPOST 'http://localhost:9096/write?db=telegraf' \
--data-binary 'relay_test,host=server01,region=us-west value=0.96 1647873106000000000'
```

This will internally call the backend InfluxDB url using the command as below: 
```
curl -i -XPOST 'http://influxdb:8086/write?db=telegraf' \
--data-binary 'relay_test,host=server01,region=us-west value=0.96 1647873106000000000'
```

Data can be verified on the influxdb container terminal like so:
```
 select * from relay_test
```
Relay can be pinged by the below command. Look for X-Influxdb-Version: relay in the output
```
sh-4.4# curl -kv http://localhost:9096/ping
*   Trying 127.0.0.1...
* TCP_NODELAY set
* Connected to localhost (127.0.0.1) port 9096 (#0)
> GET /ping HTTP/1.1
> Host: localhost:9096
> User-Agent: curl/7.61.1
> Accept: */*
> 
< HTTP/1.1 204 No Content
< X-Influxdb-Version: relay
< Date: Mon, 21 Mar 2022 16:01:37 GMT
< 
* Connection #0 to host localhost left intact
```



