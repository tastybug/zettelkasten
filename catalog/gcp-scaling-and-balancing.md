# Scaling and Balancing

This covers elasticity-related tools.

## Managed Instance Group (ReplicaSet for VMs)

This is for VMs. It mirrors the ReplicaSet of K8S.

* deploys identical VM instances based on a template
* scalable group - typically used with an autoscaler (CPU, Queue workload, metrics, ..)
* tooling ensures all instances are running
* can be zonal or regional
* VMs are expected to support health checks with similar behavious like those in K8s

## HTTP(s) GLOBAL, external Load Balancing (Service for VMs)

This is a classic managed reverse proxy.

* global load balancing; proxies to the closest backend
* offers a single anycast IP address, hiding numerous backend instances
* HTTP goes to 80 or 8080, HTTPS goes to 443
* IPv4 and IPv6
* The proxy autoscales and requires no prewarming
* configuration supports URL maps to point to particular instances based on URLs
* VMs are meant to be healthy to receive traffic
* HTTPs LB supports up to 15 certificates
* HTTPs LB can take care of certificate management, SSL policies and security patching for known vulnerabilities in the network stack 
* HTTPs LB terminates the SSL session at the balancer
* TCP LB offer the same routing like HTTP(s) LBs and support security patching

### Using Cloud Storage behind Load Balancers

Thanks to URL mapping, it's possible to serve bucket content via a HTTPs balancer.
Example: `/fetch-that-picture/bla.png` can be configured as `/fetch-that-picture/` -> bucket `pic-bucket` in `europe-north-1`.

### Using CDN behind Load Balancers

When setting up the LB, it's possible to hook up a CDN. The CDN is global, with edge locations in many places. If a client connects to the LB, the closest CDN will be involved so that a subsequent request from the same area can be served from CDN instead of from behind the LB.

The caching strategy has to be set:

* depends on origin headers (caching headers from the backend): `USE_ORIGIN_HEADERS`
* cache everything static (I cannot tell how it figure out what's static): `CACHE_ALL_STATIC`
* put everything into CDN: `FORCE_CACHE_ALL`

## Network Load Balancing: the REGIONAL, external balancer

A regional, non-proxied load balancer (meaning connections don't terminate at the LB).
Forwarding rules are based on IP protocol data (address, port). It supports UDP, TCP and SSL traffic on non-standard ports.

## Internal Load Balancing: the REGIONAL, internal balancer

A regional, private load balancer for VM instances. Supports TCP and UDP traffic.
This is also a non-proxying LB, meaning connections don't terminate at the LB.

![LB Summary](./gcp-lb-summary.png)

![Choosing the right LB](choosing-gcp-loadbalancer.png)
