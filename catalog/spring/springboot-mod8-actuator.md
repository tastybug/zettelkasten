# Actuator: Making Health Visible

Actuator is a SB module that helps with observability. At a basic level, it will provide 3 endpoints, but not all are exposed by default:

* `/actuator/info`: you can put anything here and by default, this is empty. You could return build time, version, name and so on.
* `/actuator/health`: by default, this just returns `{ "status": "UP" } as a composite status across database connectivity etc.
* `/actuator/metrics`: a list of generic and custom metric names provided by the application
  * `/actuator/metrics/<metric-name>`: returns then the content of a metric 
