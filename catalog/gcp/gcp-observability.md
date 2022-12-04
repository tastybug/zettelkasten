# GCP Observability: Monitoring, Logging, ..

Monitoring, Logging and instrumentation is all available under `GCP Operations Suite` (fka Stackdriver).

Overall Properties:
* Manages across platforms: AWS, GCP
* dynamic discovery of resources in GCP
* open source agents
* extendible with 3rd party software

There are 2 categories here:

* Operations-based tools, which aim at admins overseeing the platform
* Applicartion performance management tools for developers, allowing drilldown into an application stack

To have good observability (which enables TROUBLESHOOTING), you need 3 things:

* signals: 
  * metrics from apps, services, platform
  * logs from apps, services and platform
  * traces from apps
* incident management techniques:
  * alerts
  * error reporting
  * SLOs
* visualization and analytics:
  * dashboards
  * metrics explorer
  * logs explorer
  * service monitoring
  * health checks
  * debugger profiler   

## Operations-Based Tools

### Monitoring
Monitoring is about visualizing data.

Monitoring is automatically connected to a variety of signals: it's getting logs from GBQ about usage, from Cloud Run about CPU utilization, 
billable time and so on. From Cloud Compute, it gets CPU and memory utilization, disk throughput and more.

Applications can provide logs via OpenTelemetry custom metrics.

Monitoring is bound to a particular GCP project. It is possible to have Monitoring in 1 particular project spanning across multiple other projects (I guess must be in the same org).
A common setup is to have a "devops" project with Monitoring covering resources from projects covering stages: `project dev`, `project qa` and `project prod`. This allows a holistic view across all stages in a single pane of glass.

### Logging
Logging is about collecting, analysing and exporting (incl. retaining) log messages.

- It collects from automatically from App Engine, Cloud Run, GKE and VMs. Logs are organized by project.
- Analyzes log data with Logs Explorer, via Pub/Sub, Dataflow and GBQ. You can also analyze archived logs from Cloud Storage.
- Exporting to Cloud Storage, Pub/Sub and GBQ. Logs-based metrics can be exported to `Monitoring`
- Retention of data access logs is 1-3650 days (30d default), admin logs for 400 days. With Cloud Storage and GBQ, this can be even longer.

There are 3 categories of logs:
- cloud audit logs
  - "Who did what, when, where?"
  - admin activity
  - data access
  - systems events
  - access transparency (for when there is a Google employee doing manual intervention)
- agent logs
  - fluentd agent (a locally running log collection agent) for VMs and containers
  - common third-party applications
  - system software
  - apps calling the Cloud Logging API (there's libs for that)
- network logs
  - vpc flow
  - firewall rules
  - NAT gateway
  - load balancers

### Error Reporting
Counts, analyzes and aggregates crashes in running cloud services (e.g. GKE). You can filter.
It's possible to get notifications on new errors.

You see here:
- stack traces
- number of occurrences
- first and last seen
- number of affected users

### Service Monitoring
Understand and troubleshoot intra-service dependencies. Supports App Engine, Anthos Service Mesh and Istio.
You can create SLOs on errors here and see the remaining budget.

## Application Performance Management Tools

### Debugger
Debug running code. Debug sessions can be shared.
Debug using snapshots, logpoints, conditional debugging.

This is integrated with popular IDEs.

Debugger can be connected to various SCMs (Github, Bitbucket, Cloud Source, Gitlab) to understand versioning.

### Tracer

See latency of calls to an application in near-real-time. This breaks down what happens in the application.
If the application calls others, even in other projects, this will be part of the visualization and break down.

Supports App Engine, Compute Engine and GKE.

### Profiler

Helps to improve performance and reduce costs using low-impact cpu usage and heap profiling.
You'll see a heat map of your call stack and how long everything takes.

Supports VMs, App Engine, Compute Engine and GKE with Java, Go, Python and NodeJS.
