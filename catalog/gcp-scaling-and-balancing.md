# Scaling and Balancing

This covers elasticity-related tools.

## Managed Instance Group (ReplicaSet for VMs)

This is for VMs. It mirrors the ReplicaSet of K8S.

* deploys identical VM instances based on a template
* scalable group - typically used with an autoscaler (CPU, Queue workload, metrics, ..)
* tooling ensures all instances are running
* can be zonal or regional
* VMs are expected to support health checks with similar behavious like those in K8s

## HTTP(s) Load Balancing (Service for VMs)

This is a classic managed reverse proxy.

* global load balancing
* offers a single anycast IP address, hiding numerous backend instances
* HTTP goes to 80 or 8080, HTTPS goes to 443
* IPv4 and IPv6
* The proxy autoscales and requires no prewarming
* configuration supports URL maps to point to particular instances based on URLs
* VMs are meant to be healthy to receive traffic
* HTTPs balancer supports up to 15 certificates
* HTTPs balancer terminates the SSL session at the balancer

### Using Cloud Storage behind Load Balancers

Thanks to URL mapping, it's possible to serve bucket content via a HTTPs balancer.
Example: `/fetch-that-picture/bla.png` can be configured as `/fetch-that-picture/` -> bucket `pic-bucket` in `europe-north-1`.
