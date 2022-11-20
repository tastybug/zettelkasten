# GCP Observability: Monitoring, Logging, ..

Monitoring, Logging and instrumentation is all available under `GCP Operations Suite` (fka Stackdriver).

Parts:
* Monitoring
* Logging
* Error Reporting
* Tracing
* Debugging

Overall Properties:
* Manages across platforms: AWS, GCP
* dynamic discovery of resources in GCP
* open source agents
* extendible with 3rd party software

## Monitoring aka Cloud Monitoring fka Stackdriver Monitoring

This encompasses platform, system and application metrics and the visualization of those. Alerting pushes notification to you if things get out of hand.

Sources: Metrics, Events, Metadata, Uptime Checks (Appengine Instance, VM, HTTP/HTTPS URL, AWS Instance or Loadbalancer)
Sinks: Alerts, Dashboards, Charts

> VM instances require a monitoring agent to report data. This can be done as part of the startup script by running 2 lines of script.

### Metric Scope

When you open the Monitoring Product Page, it will tell you yhe Metrics Scope. This defines which GCP projects Monitoring is able to monitor.
In principle it seems to be possible to have a dedicated monitoring project that sees all other projects and monitors what is going on.

### Uptime Checks

This is a pretty convenient category in the Monitoring product that allows you to see if resources are healthy. There a ton of options on how to check it.
Uptime checks are run from multiple continents: Asia Pacific, Europe and both Americas.

### Logging

* log retention is 30d, but it can be exported to pub/sub, cloud storage or GBQ
* it's possible to have alerts on logging
* for VMs there is a "logging agent", that has to be installed

### Error Reporting

This SEEMS (didn't check) to collect errors from a variety of data sources, allowing you to create dashboards and notifications.

Data sources mentioned are:
* App Engine
* Apps Script
* Compute Engine
* Cloud Functions
* Cloud Run
* GKE
* EC2

Supported languages are: Go, Java, .NET, Node.js, PHP, Python, Ruby

### Tracing

Distributed tracing system. Allows real time view of:

* per URL latency samping
* latency reporting

Data sources:
* App Engine
* HTTP(S) load balancers
* applications instrumented with the cloud trace SDK

### Debugging

Allows debugging without stopping or slowing down the application. Allows:
* inject logging without stopping
* capture snapshot of stack

Supported languages: Java, Pythin, Go, Node.js, Ruby, PHP and .NET Core
