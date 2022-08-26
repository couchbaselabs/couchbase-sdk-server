# Couchbase SDK Server
Allows rebuilding the SDK server.  
Should probably be in Salt/Ansible/Puppet/Chef instead but I don't know any of those, so here we are.

# Running
Clone this then:
```
docker-compose up -d --remove-orphans
```

# Observability Stack
Used for FIT integration testing of SDK observability tracing and metrics output.

Consists of:

* Jaeger.  The tracing database.  Can access the UI on http://performance.sdk.couchbase.com:16686.  The retention is 1m traces, which should be fine for integration testing. 
* Prometheus.  The metrics database.  Can access the UI on http://performance.sdk.couchbase.com:9090.  Retention is 15 days.
* OpenTelemetry collector.  This is the ingest point for integration testing, running on port 4317. 
The performers are told to send their data here, and it forwards it on to Prometheus, Jaeger and (for now) Grafana Tempo.
IMPORTANT: Do not send anything else to this port - e.g. don't use it for performance or adhoc testing.
We don't want to interfere with the tests.
* Grafana, with UI running on http://performance.sdk.couchbase.com:3000.  Just to allow easier viewing and debugging of the telemetry.

And deprecated/legacy (may be removed in future):

* Grafana Tempo.  The tracing database.  Can access the UI on http://performance.sdk.couchbase.com:.  Retention is 14 days.
  * The search capabilities just don't work reliably enough for integration testing.  Replaced with Jaeger.
