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

FIND OUT WHAT THAT IS
