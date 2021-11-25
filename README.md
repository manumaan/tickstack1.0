# Custom TICK stack Sandbox 2.0
This is a custom TICK stack sandbox with:  

TELEGRAF 1.2
INFLUXDB 2.1
CHRONOGRAF 1.9
KAPACITOR 1.6
GRAFANA 7.1.0 

### Running

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

- `localhost:8086` - InfluxDB's address. You will use this as a management UI for the Influx DB and dashboards etc starting from 2.0
- `localhost:8888` - Chronograf's address. In 1.x Chronograf was the central tool. 
                     Below we will be doing some steps to ensure backward compatibility with chronograf. 
- `localhost:3010` - Documentation server. This contains a simple markdown server for tutorials and documentation.


> NOTE: Make sure to stop any existing installations of `influxdb`, `kapacitor` or `chronograf`. If you have them running the Sandbox will run into port conflicts and fail to properly start. In this case stop the existing processes and run `./sandbox restart`. Also make sure you are **not** using _Docker Toolbox_.

Once the Sandbox launches, you should see your dashboard appear in your browser:

### Configuration Steps 
Once sandbox is up, login to the influx db admin page: http://localhost:8086/
Enter these details:
```
Org: org
User: admin 
Password: admin123
Bucket: telegraf (Bucket name has to be telegraf for chornograf to work) 
```
Ensure the database initialization is completed. 

Copy the Admin's token from Data-> API Tokens screen and paste in telegraf.conf and kapacitor.conf. 

Restart telegraf docker from docker desktop and check status. 
Once telegraf is running, you can check the system metrics coming inside the telegraf database. 

Restart kapacitor docker container from docker desktop and check status. 

From Chronograf you can add the database into the config as given here:
https://docs.influxdata.com/chronograf/v1.9/administration/creating-connections/#manage-influxdb-connections-using-the-chronograf-ui

Influx DB config:
```
URL for influxDB: http://influxdb:8086
Org: org
Token: Value from API Tokens screen
Database: telegraf
Retention Policy: autogen 
```
Kapacitor config:
```
URL for Kapacitor: http://kapacitor:9092
Leave other fields blank/default. 
```

Influx DB 2.0 does not have databases and retention policies, it has buckets which combine both. To support tools like Chronograf, we need to create mapping between our bucket and db, rp combination. For this login to the influxdb: 

./sandbox enter influxdb

To map the bucket as per the v1.0 requirement enter below command. Token value is from API Tokens screen, and bucket-id is from Data->Buckets screen. 

```
influx v1 dbrp create \
   --org=org \
   --token=LLDY7DAD3VLT_MytcF1AX7A72qOgEFlR3GG98HNGp0Now0zaz0H1ACWcZoI6xQb5-qiT8LR8jPpTzQGJUEKGlg== \
   --db telegraf \
   --rp autogen \
   --bucket-id ff28b87f09e61d21 --default
```

### Grafana
Grafana can be accessed at localhost:3000 

Default credentials for Grafana: admin/admin 

Enter these and change the default password. 

Configuration:
Left side menu (+ icon)  -> create -> Dashboard -> Create a sample dashboard. Save. 

Left side menu (gear icon) -> configuration - Data Source (Select influxdb) 

Change query language to Flux (influxql only supported in 1.x)

```
 HTTP url:  http://influxdb:8086 
 Database: telegraf
 Org: org
 Token: value from the API Tokens
 ```
 
 
Click Save & Test


Reference: 
https://www.influxdata.com/blog/running-influxdb-2-0-and-telegraf-using-docker/

https://www.sqlpac.com/en/documents/influxdb-v2-getting-started-setup-preparing-migration-from-version-1.7.html

https://docs.influxdata.com/influxdb/v2.0/tools/chronograf/

https://github.com/influxdata/chronograf/issues/2830
