# Couchbase SDK Server
Allows rebuilding the SDK server.  
Should probably be in Salt/Ansible/Puppet/Chef instead but I don't know any of those, so here we are.

# Running
Clone this, change to the directory, create a `.env` file:
```
POSTGRES_PASSWORD=<must be provided>
```

(You'll need to get the password from someone, the same one other accessors of the database are depending on.) 

Then:
```
docker-compose up -d --remove-orphans
```

On production (performance-sdk.couchbase.com), it's setup with run automatically with systemd using the scripts under `systemd` directory:

```
sudo systemctl start couchbase_sdk_server
```

# Observability Stack
Used for FIT integration testing of SDK observability tracing and metrics output.

Consists of:

* Jaeger.  The tracing database used for integration testing.  Can access the UI on https://performance-sdk.couchbase.com:16686.  The retention is 1m traces, which should be fine for integration testing. 
* Prometheus.  The metrics database.  Can access the UI on https://performance-sdk.couchbase.com/prometheus.  Retention is 15 days.
* Honeycomb.  The tracing database used for adhoc and performance testing.  Can join with this link https://ui.honeycomb.io/join_team/couchbase-sdk.

There are three `opentelemetry-collector` instances ready for ingesting OTLP GRPC data:

* Port 4317 for adhoc testing.  Forwards to Prometheus and Honeycomb.  SDK team members should feel free to use this for any adhoc needs.  Though do bear in mind that our free plan for Honeycomb is very rate limited.  Exposed at https://performance-sdk.couchbase.com:4317
* Port 10001 for performance testing.  Forwards to Prometheus and Honeycomb.  Exposed at https://performance-sdk.couchbase.com:10001
* Port 10003 for integration testing.  Forwards to Jaeger, Prometheus and Honeycomb (and, for now, Grafana Tempo).  Exposed at https://performance-sdk.couchbase.com:10003
Important: Do not send anything else to this port - e.g. don't use it for performance or adhoc testing.
We don't want to interfere with the tests.

## Debugging missing telemetry
Check if opentelemetry-collector saw it: https://performance-sdk.couchbase.com:10006/metrics (that's the integration test collector - if looking for performance or adhoc data, replace that port with the correct one from prometheus.yaml) 

# Logging Stack
Grafana on https://performance-sdk.couchbase.com/grafana, with Loki as backend.

# Performance
The database is started with the docker-compose.  If it needs to be done manually for some reason then: 
```
sudo docker run -p 5432:5432 -v /home/ec2-user/perf-data:/var/lib/postgresql/data -e POSTGRES_PASSWORD=<PROD_PASSWORD> --name timedb timescale/timescaledb:latest-pg14
```

Follow instructions under transactions-fit-performer/perf-driver for one-off database setup.

Production is setup to automatically regularly pull and use any pushed changes to the repositories, as can be seen with:

```
ssh -i ~/keys/sdk-performance.pem ec2-user@performance-sdk.couchbase.com
crontab -l
crontab -e
less /var/spool/mail/ec2-user
```

The backend and frontend are also setup to run automatically with systemd via the scripts in `systemd`.

```
sudo systemctl daemon-reload
sudo systemctl start perf_vue
sudo systemctl start perf_nest
```
