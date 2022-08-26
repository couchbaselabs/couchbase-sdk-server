# Couchbase SDK Server
Allows rebuilding the SDK server.  
Should probably be in Salt/Ansible/Puppet/Chef instead but I don't know any of those, so here we are.

# Running
Clone this then:
```
docker-compose up --remove-orphans
```

# Observability Stack
Used for FIT integration testing of SDK observability tracing and metrics output.

Consists of:

* Grafana, with UI running on port 3000.  Just to allow easier viewing and debugging of the telemetry.
* Jaeger.  The tracing database.  Can access the UI on port 16686.  The retention is 1m traces. 
* Prometheus.  The metrics database.  Can access the UI on port 9090.  Retention is 15 days.
* OpenTelemetry collector.  This is the ingest point for integration testing, running on port 4317.  The performers are told to send their data here, and it forwards it on to Prometheus, Jaeger and (for now) Grafana Tempo.

And deprecated/legacy (may be removed in future):

* Grafana Tempo.  The tracing database.  Can access the UI on port 3200.  Retention is 14 days.
  * The search capabilities just don't work reliably enough for integration testing.  Replaced with Jaeger.
