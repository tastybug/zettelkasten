# Istio

## Why

Istio gives you traffic management, security, observability.

## How

Istio is a set of concepts and applications (Envoy Proxy). It is working as part of K8s.

## Concepts and their K8s CRDs

### Virtual Services

`kubectl get vs`

Virtual Service is one or more k8s services in a namespace. Virtual Services let you configure ingress and egress traffic in combination with gateways.
VS come with basic routing options. More complex rules are done with destination rules.

Hint: since you can group multiple k8s services into a single VS, you can present multiple services as a monolithic system to the outside world. Or you can start with a proper monolith and then over time break it apart but obfuscating this by using a VS.

Example:
```
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: reviews
spec:
  # hosts: can be an IP address, a DNS name, K8s service name
  hosts:
  - reviews
  http:
  - match:
    - headers:
        end-user:
          exact: jason
    route:
    - destination:
		# host must be a k8s service name; when given as short, it must be in the same namespace
        host: reviews
        subset: v2
  - route:
    - destination:
        host: reviews
        subset: v3
```


### Destination Rules

You can think of virtual servicds as how you route traffic to a given destination and then you use destination rules to configure what happens with the traffic for that destination.
You can create subsets of service instances, e.g. group them by version. Confusingly, you can then alter your VS to route traffic to these subsets.

In DR you control load balancing, TLS security mode (unclear yet what that is) and circuit breaker settings.

Here is an example with 3 subsets grouped by label, a general load balancing setting and a group specific load balancing mode.

```
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: my-destination-rule
spec:
  host: my-svc
  trafficPolicy:
    loadBalancer:
      simple: RANDOM
  subsets:
  - name: v1
    labels:
      version: v1
  - name: v2
    labels:
      version: v2
    trafficPolicy:
      loadBalancer:
        simple: ROUND_ROBIN
  - name: v3
    labels:
      version: v3
```

#### Load Balancing

By default, Istio uses a least requests load balancing policy. Alternatively, you can use:
* random
* weighted (according to percentage rules)
* round robin

### Gateways

### Service Entries

### Sidecars
