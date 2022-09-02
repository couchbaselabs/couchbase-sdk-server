# Couchbase SDK Server
Allows rebuilding the SDK server.  
Should probably be in Salt/Ansible/Puppet/Chef instead but I don't know any of those, so here we are.

# Running
Clone this then:
```
docker-compose up -d --remove-orphans
```

On production (performance.sdk.couchbase.com), it's setup with run automatically with systemd using the scripts under `systemd` directory:

```
sudo systemctl start couchbase_sdk_server
```

# Observability Stack
Used for FIT integration testing of SDK observability tracing and metrics output.

Consists of:

* Jaeger.  The tracing database used for integration testing.  Can access the UI on http://performance.sdk.couchbase.com:16686.  The retention is 1m traces, which should be fine for integration testing. 
* Prometheus.  The metrics database.  Can access the UI on http://performance.sdk.couchbase.com:9090.  Retention is 15 days.
* Honeycomb.  The tracing database used for adhoc and performance testing.  Can join with this link https://ui.honeycomb.io/join_team/couchbase-sdk.

There are three `opentelemetry-collector` instances ready for ingesting OTLP GRPC data:

* Port 4317 for adhoc testing.  Forwards to Prometheus and Honeycomb.  SDK team members should feel free to use this for any adhoc needs.  Though do bear in mind that our free plan for Honeycomb is very rate limited..
* Port 10001 for performance testing (http://performance.sdk.couchbase.com:8080/).  Forwards to Prometheus and Honeycomb.
* Port 10003 for integration testing.  Forwards to Jaeger, Prometheus and Honeycomb (and, for now, Grafana Tempo).
Important: Do not send anything else to this port - e.g. don't use it for performance or adhoc testing.
We don't want to interfere with the tests.

And deprecated/legacy/commented-out:

* Grafana Tempo.  The tracing database.  Can access the UI on http://performance.sdk.couchbase.com:.  Retention is 14 days.
  * The search capabilities just don't work reliably enough for integration testing.  Replaced with Jaeger.

## Debugging dropped packages

http://13.57.94.142:10006/metrics

# Performance
Production is setup to automatically regularly pull and use any pushed changes to the repositories, as can be seen with:

```
ssh -i ~/keys/sdk-performance.pem ec2-user@performance.sdk.couchbase.com
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